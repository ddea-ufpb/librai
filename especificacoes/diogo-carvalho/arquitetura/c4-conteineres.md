# LibrAI · C4 nível 2 — Contêineres (arc42 §5)

> Unidades executáveis do LibrAI, a tecnologia de cada uma e como se comunicam. "Contêiner", no C4, é uma unidade que executa ou guarda dados — não necessariamente um contêiner Docker (o mapeamento para Docker está na última seção).

## Diagrama

```mermaid
C4Container
    title LibrAI — Contêineres

    Person(aluno, "Aluno")
    Person(professor, "Professor")
    Person(admin, "Administrador / curador")

    System_Boundary(librai, "LibrAI") {
        Container(web, "Aplicação web", "Next.js 15 / React / TypeScript", "Telas G, A, P e C com rotas por papel. No navegador, extrai os landmarks das mãos com MediaPipe Hands; a imagem não sai do aparelho.")
        Container(api, "API LibrAI", "Python 3.12 / FastAPI / Uvicorn", "Autenticação, consentimento, sessões, curadoria, turmas. Hospeda o agente tutor (LangGraph) e a regra decide_next.")
        Container(rec, "Pool de reconhecimento", "Python ProcessPoolExecutor, N workers fixos", "Carrega o modelo de sinais do LibrAI e classifica landmarks → (label, confidence). Fila limitada a 2×N.")
        ContainerDb(db, "Banco de dados", "SQLite 3 em modo WAL (arquivo em volume)", "Contas, consentimentos, sinais, sessões, tentativas, resultados, turmas, reportes.")
        ContainerDb(media, "Mídia de referência", "Sistema de arquivos (volume Docker)", "Vídeos de referência mp4/webm enviados pelo curador.")
    }

    System_Ext(llm, "Provedor de LLM", "Local ou API; opcional")

    Rel(aluno, web, "Usa", "HTTPS")
    Rel(professor, web, "Usa", "HTTPS")
    Rel(admin, web, "Usa", "HTTPS")
    Rel(web, api, "Chama /api/v1 (cookie de sessão)", "HTTPS/JSON")
    Rel(api, rec, "recognize_sign(landmarks)", "IPC de processo, timeout 5 s")
    Rel(api, db, "Lê e grava (transação + version)", "sqlite3")
    Rel(api, media, "Grava uploads e serve vídeos", "arquivo")
    Rel(api, llm, "generate_feedback", "HTTP/JSON, timeout 3 s")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

## Contêineres

| Contêiner | Tecnologia | Responsabilidade | Expõe / consome | Decisões | Requisitos-chave |
|---|---|---|---|---|---|
| **Aplicação web** | Next.js 15, React, TypeScript; `@mediapipe/tasks-vision` servido localmente | 18 telas (G01–C04), roteamento por `role`, captura e extração de `landmarks`, estados offline e carregando | Consome `openapi.yaml` | ADR-0002, ADR-0007 | REQ-09, REQ-51, REQ-52, REQ-54, REQ-55, REQ-57 |
| **API LibrAI** | FastAPI, Pydantic v2 (schemas gerados a partir do OpenAPI), LangGraph, Argon2id | Regras de negócio; agente tutor (grafo validar → reconhecer → decidir → redigir → gravar); autorização por papel e posse | Expõe `/api/v1` ([`openapi.yaml`](contratos/openapi.yaml)) | ADR-0001, ADR-0003, ADR-0006, ADR-0008 | REQ-01–REQ-48, REQ-59 |
| **Pool de reconhecimento** | `concurrent.futures.ProcessPoolExecutor` com `RECOGNIZER_WORKERS` (padrão: nº de núcleos − 1) | Inferência em CPU isolada do laço de eventos da API; recusa trabalho além da fila | `recognize_sign` ([`ferramentas-agente.md`](contratos/ferramentas-agente.md)) | ADR-0004 | REQ-16, REQ-27, REQ-28, REQ-50 |
| **Banco de dados** | SQLite 3, `journal_mode=WAL`, `busy_timeout=5000` | Persistência transacional e invariantes (índices únicos parciais) | [`modelo-dados.md`](contratos/modelo-dados.md) | ADR-0005 | REQ-15, REQ-25, REQ-30, REQ-43 |
| **Mídia de referência** | Diretório em volume; nome do arquivo = `reference_media.id` | Guardar e servir vídeos de referência (nunca vídeo de aluno) | `GET /media/{id}` estático via API | — | REQ-41, REQ-44 |

## Fluxo principal: uma tentativa (A03 → A04)

```mermaid
sequenceDiagram
    autonumber
    participant N as Navegador (Aplicação web)
    participant A as API (agente tutor)
    participant R as Pool de reconhecimento
    participant D as SQLite
    participant L as Provedor de LLM

    N->>N: captura 3 s · MediaPipe → landmarks (imagem descartada)
    N->>A: POST /practice-sessions/{id}/attempts {client_attempt_id, expected_sign_id, landmarks}
    A->>D: SELECT attempt WHERE session_id, client_attempt_id
    alt já existe (reenvio)
        A-->>N: 200 AttemptResult existente (REQ-25)
    else nova
        A->>D: SELECT sessão (status, sinal atual, version)
        A->>R: recognize_sign(landmarks) [timeout 5 s]
        R-->>A: label, confidence, inference_ms
        A->>A: descarta landmarks · decide_next() → REPEAT/ADVANCE
        A->>L: generate_feedback() [timeout 3 s]
        L-->>A: feedback_text (ou template, REQ-21)
        A->>D: BEGIN IMMEDIATE · INSERT attempt · INSERT sign_outcome? · UPDATE sessão WHERE version=? · COMMIT
        A-->>N: 201 AttemptResult + SessionState
    end
    N->>N: A04 mostra resultado só agora (REQ-55)
```

## Mapeamento para implantação (Docker Compose)

| Serviço Docker | Contêineres C4 | Volume |
|---|---|---|
| `web` | Aplicação web | — |
| `api` | API LibrAI + Pool de reconhecimento (mesmo contêiner Docker, processos separados) | `librai-data:/data` (`librai.db` + `media/`) |

Um único host, sem GPU (FI-01 §10.3). O primeiro administrador é criado no contêiner `api` pelo comando `python -m librai.create_admin` (antes de existir C04). O LLM local, quando houver, roda fora do Compose e é configurado por `LIBRAI_LLM_URL`.
