# LibrAI · Cenários de qualidade (arc42 §10)

> Um cenário por item do envelope não-funcional da disciplina, no formato de 6 partes de Bass, Clements e Kazman (*Software Architecture in Practice*, 4ª ed.). Cada medida é verificável por um teste ou script nomeado; esses cenários viram os **ACs do envelope** no PAUL.
>
> † = meta **provisória**. O FI-01 §7.2 diz que a meta numérica vem depois do baseline. Os valores abaixo existem para que o teste seja escrito agora; a origem de cada um está declarada e será confirmada ou corrigida após a primeira medição ([`riscos-e-lacunas.md`](riscos-e-lacunas.md) L-04).

## Árvore de utilidade (resumo)

| Envelope | Cenário | Meta de qualidade (§1.3) | Requisitos | Decisões |
|---|---|---|---|---|
| Persistência | QS-01 | 1 · Privacidade, 3 · Consistência | REQ-11, REQ-30 | ADR-0002, ADR-0005 |
| API com contrato | QS-02 | 5 · Contrato estável | REQ-10, todos os `x-requirements` | — |
| Concorrência | QS-03 | 3 · Consistência | REQ-15, REQ-25, REQ-26 | ADR-0003 |
| Desempenho medido | QS-04 | 4 · Resposta rápida | REQ-27, REQ-49, REQ-50 | ADR-0004 |
| Testes | QS-05 | 2 · Decisão correta | REQ-17–REQ-21 | ADR-0001, ADR-0008 |

---

## QS-01 · Persistência

| Parte | Conteúdo |
|---|---|
| **Fonte** | Falha de infraestrutura: o processo da API é morto (`kill -9`) ou o contêiner é reiniciado |
| **Estímulo** | Interrupção no meio de `submit_attempt`, entre o `INSERT attempt` e o `COMMIT`, repetida 100 vezes em pontos aleatórios |
| **Ambiente** | Operação normal com 10 sessões ativas; SQLite em WAL no volume `librai-data` |
| **Artefato** | `record_attempt` + banco SQLite (`attempt`, `sign_outcome`, `practice_session`) |
| **Resposta** | Depois do reinício, cada sessão está **inteira** no estado anterior ou no posterior à tentativa: nunca há `attempt` sem a mudança de `current_position`/`version` correspondente, nem o contrário. Nenhum arquivo ou coluna contém imagem ou vídeo de aluno |
| **Medida** | **0 sessões inconsistentes** em 100 interrupções, verificadas pela consulta de integridade `scripts/check_integrity.sql` (para cada sessão: `current_position` = nº de `sign_outcome` da sessão) · **0 ocorrências** de imagem/vídeo de aluno na varredura do volume e do banco (busca por assinaturas JPEG/PNG/WebM/MP4 fora de `media/`, que só contém referências do curador) — métrica do FI-01 §10.1 |

## QS-02 · API com contrato

| Parte | Conteúdo |
|---|---|
| **Fonte** | Suíte de testes de contrato gerada a partir de [`contratos/openapi.yaml`](contratos/openapi.yaml) (Schemathesis) |
| **Estímulo** | 1.000 requisições geradas por propriedade cobrindo as 34 operações, incluindo corpos inválidos, campos extras (ex.: imagem em base64 em `submit_attempt`) e ids inexistentes |
| **Ambiente** | Pipeline de CI, banco semeado com `seed_dev.py` (3 contas, 5 sinais publicados), `FakeRecognizer`, LLM desligado |
| **Artefato** | API `/api/v1` |
| **Resposta** | Toda resposta usa um status declarado para a operação e um corpo válido no schema; corpo fora do schema → 422 `invalid_payload` sem gravar nada (REQ-10); `error.code` sempre entre os valores do enum |
| **Medida** | **0 violações de schema ou de status** · **34 de 34 operações** exercitadas · **0 respostas 5xx** · o arquivo `openapi.yaml` passa no `openapi-spec-validator` a cada commit |

## QS-03 · Concorrência

| Parte | Conteúdo |
|---|---|
| **Fonte** | Cliente do aluno: reenvio automático por rede instável + duas abas abertas na mesma sessão |
| **Estímulo** | **20 requisições simultâneas** de `submit_attempt` para a mesma sessão e o mesmo `expected_sign_id`: 10 com o **mesmo** `client_attempt_id` e 10 com ids **diferentes** |
| **Ambiente** | Operação normal; `FakeRecognizer` devolvendo o sinal esperado com `confidence` 0,95 (acerto) |
| **Artefato** | `submit_attempt` → `record_attempt` (idempotência + `version`, ADR-0003) |
| **Resposta** | O mesmo `client_attempt_id` gera **uma** tentativa gravada; as 9 repetições recebem 200 com o mesmo `attempt_id` (REQ-25). A sessão avança **exatamente uma** posição; as requisições com outros ids que chegaram depois recebem 409 `stale_sign` (REQ-26). Todas as 20 recebem resposta (vivacidade) |
| **Medida** | **0 tentativas duplicadas** e **0 avanços duplos** em **500 execuções** do teste `test_concurrent_submit_same_session` (pytest + `asyncio.gather` + `httpx.AsyncClient`) · 100% das requisições respondidas em ≤ 5 s · variante: 20 chamadas simultâneas a `start_session` do mesmo aluno → **1** sessão ativa (REQ-15) |

## QS-04 · Desempenho medido

| Parte | Conteúdo |
|---|---|
| **Fonte** | Script de carga Locust (`load/locustfile.py`) simulando alunos |
| **Estímulo** | **10 sessões simultâneas**, cada uma enviando uma tentativa a cada 8 s (3 s de captura + cerca de 5 s lendo o resultado) por 10 minutos — cerca de 750 tentativas — com `landmarks` reais gravados de 5 sinais (fixtures, sem imagem) |
| **Ambiente** | Docker Compose numa máquina de 4 núcleos, sem GPU (FI-01 §10.3); rodada A com LLM desligado (template), rodada B com o LLM configurado |
| **Artefato** | `submit_attempt` (`total_ms`) e `recognize_sign` (`inference_ms`), registrados em cada `attempt` e no log JSON (REQ-49) |
| **Resposta** | Todas as tentativas respondidas; o pool recusa com 503 em vez de enfileirar sem limite (REQ-27) |
| **Medida** | † **p95 de `total_ms` ≤ 2.000 ms** e **p99 ≤ 4.000 ms** (REQ-50) · **503 ≤ 1%** das tentativas · **0 outros 5xx** · relatório com p50, p95 e p99 de `inference_ms` e `total_ms` por rodada. *Origem:* 10 sessões = uma turma praticando ao mesmo tempo em laboratório; 2 s = limite em que o retorno ainda parece ligado ao gesto (faixa de 1 a 10 s de Nielsen para manter a atenção), com folga para rede móvel. Baseline ainda não medido |

## QS-05 · Testes

| Parte | Conteúdo |
|---|---|
| **Fonte** | Desenvolvedor ou agente de código (Claude Code via PAUL) ao fazer push |
| **Estímulo** | Execução de `pytest` no CI a cada push e antes de fechar cada plano |
| **Ambiente** | **Sem câmera, sem modelo real e sem LLM**: `FakeRecognizer(label, confidence)` e `LIBRAI_LLM_PROVIDER=none` (FI-01 §11.3) |
| **Artefato** | `decide_next`, grafo do agente tutor, endpoints da API |
| **Resposta** | Toda a suíte passa; cada linha da tabela de decisão ([`ferramentas-agente.md`](contratos/ferramentas-agente.md) §3) e cada AC do FI-01 tem um teste nomeado: `test_decide_next_advances_when_label_matches_and_confidence_above_threshold` (AC-1, REQ-17), `test_decide_next_repeats_on_wrong_label` e `test_decide_next_repeats_on_low_confidence` (AC-2, REQ-18), `test_decide_next_advances_after_max_attempts` (REQ-19), `test_feedback_falls_back_to_template_on_llm_timeout` (REQ-21) |
| **Medida** | **100%** das 4 linhas da tabela de decisão e dos 2 ACs cobertos · cobertura de ramos **≥ 90%** em `librai/tutor/` e **≥ 80%** no backend (`pytest --cov --cov-branch`) · suíte completa em **< 60 s** · **0 testes** que dependam de câmera, rede ou LLM |
