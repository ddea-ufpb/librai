# LibrAI · Arquitetura do sistema

> Trabalho 1 — Telas e arquitetura · Programação com Agentes · aluno **Diogo Soares Alves Barreto de Carvalho** · 27/10/2026
> Entrada para `/paul:init` e `/paul:plan`. Telas correspondentes em [`../telas/`](../telas/).

## 1. Padrão escolhido: arc42 enxuto

O **arc42** (Starke e Hruschka) serve de **índice**, e cada seção usa o padrão mais adequado ao tipo de informação, como o enunciado recomenda (§3.2):

| Peça | Padrão | Seção arc42 | Arquivo | Mínimo pedido | Entregue |
|---|---|---|---|---|---|
| Metas e restrições | texto | §1–2 | [`metas-e-restricoes.md`](metas-e-restricoes.md) | obrigatório | ✔ |
| Contexto | C4 nível 1 (Mermaid `C4Context`) | §3 | [`c4-contexto.md`](c4-contexto.md) | obrigatório | ✔ |
| Contêineres | C4 nível 2 (Mermaid `C4Container`) + sequência | §5 | [`c4-conteineres.md`](c4-conteineres.md) | obrigatório | ✔ |
| Decisões | ADR no formato MADR | §9 | [`adr/`](adr/) | ≥ 3 | **8** |
| Requisitos | EARS | — | [`requisitos.md`](requisitos.md) | ids `REQ-xx` | **61** (REQ-01 a REQ-61) |
| Contratos | OpenAPI 3.1 + contratos internos do agente | — | [`contratos/openapi.yaml`](contratos/openapi.yaml), [`contratos/ferramentas-agente.md`](contratos/ferramentas-agente.md) | operações usadas pelas telas | **34 operações** (todas usadas por ao menos uma tela) |
| Modelo de dados | ER (Mermaid) + DDL SQLite | — | [`contratos/modelo-dados.md`](contratos/modelo-dados.md) | modelo do sistema | 11 tabelas |
| Qualidade | cenários de 6 partes (Bass, Clements e Kazman) | §10 | [`qualidade.md`](qualidade.md) | 5, um por item do envelope | **5** (QS-01 a QS-05) |
| Riscos e lacunas | texto | §11 | [`riscos-e-lacunas.md`](riscos-e-lacunas.md) | — | extra |
| Glossário | texto | §12 | [`glossario.md`](glossario.md) | obrigatório | ✔ |
| Rastreabilidade | matriz gerada | — | [`rastreabilidade.md`](rastreabilidade.md) | — | extra |

Seções do arc42 **omitidas de propósito**: §4 (estratégia de solução — está resumida nos ADRs), §6 (visão de execução — o único fluxo crítico está como diagrama de sequência em `c4-conteineres.md`), §7 (implantação — tabela no fim de `c4-conteineres.md`) e §8 (conceitos transversais — privacidade e concorrência estão nos ADR-0002 a ADR-0005). Num sistema deste tamanho, elas repetiriam o que já está em outro lugar.

## 2. Justificativa: como o padrão entrega as seis informações do PAUL

O enunciado (§3.1) exige que o padrão entregue seis informações que o plano do PAUL consome. A tabela mostra **onde** cada uma está e **como** ela vira artefato do PAUL:

| Informação (§3.1) | Onde está nesta especificação | Por que este padrão entrega bem | No PAUL, vira |
|---|---|---|---|
| **Limites e vizinhos** | C4 nível 1 ([`c4-contexto.md`](c4-contexto.md): seção "Limites"), C4 nível 2, fora de escopo em [`metas-e-restricoes.md`](metas-e-restricoes.md) §2.4 | O C4 obriga a desenhar a fronteira do sistema e cada sistema externo com o protocolo da relação; a lista "dentro/fora" fica explícita, não implícita no diagrama | Escopo no `PROJECT.md` e `boundaries` dos planos (ex.: "não integrar LLM como decisor", "não persistir landmarks") |
| **Decisões e porquês** | 8 ADRs em [`adr/`](adr/) | O MADR registra **opções descartadas** e **consequências negativas**; o agente vê por que a alternativa óbvia (ex.: o LLM decidir, Redis) foi rejeitada e não reabre a discussão | Tabela *Key Decisions* do `PROJECT.md`; decisões que o agente não reabre a cada sessão |
| **Contratos** | [`contratos/openapi.yaml`](contratos/openapi.yaml) (com `x-screens` e `x-requirements` por operação), [`ferramentas-agente.md`](contratos/ferramentas-agente.md) (assinaturas Python tipadas), DDL em [`modelo-dados.md`](contratos/modelo-dados.md) | O OpenAPI é verificável por máquina (validado com `openapi-spec-validator`) e permite gerar schemas Pydantic e o cliente TypeScript; o DDL é executável | Tasks com `files` (ex.: `librai/tutor/decide.py`) e `verify` precisos (ex.: "Schemathesis 0 violações", "`sqlite3 < ddl.sql`") |
| **Requisitos testáveis** | 61 requisitos EARS em [`requisitos.md`](requisitos.md), cada um com telas e `operationId` | Os moldes EARS forçam gatilho + resposta observável (código de erro, status HTTP, campo gravado), o que se traduz direto em Given/When/Then | Critérios de aceite Given/When/Then dos planos (ex.: REQ-25 → *Given* tentativa gravada com `client_attempt_id` X, *When* chega outra com X, *Then* 200 com o mesmo `attempt_id` e nenhuma linha nova) |
| **Qualidade com medida** | 5 cenários em [`qualidade.md`](qualidade.md), um por item do envelope | O formato de 6 partes obriga a nomear o **estímulo** e a **medida** numérica com a ferramenta que a produz (pytest, Locust, Schemathesis) | ACs do envelope: persistência (QS-01), API (QS-02), concorrência (QS-03), desempenho medido (QS-04), testes (QS-05) |
| **Restrições** | [`metas-e-restricoes.md`](metas-e-restricoes.md) §2: stack, hospedagem (Docker, sem GPU), LGPD com a consequência técnica de cada artigo | Separa restrições técnicas, organizacionais e legais, e a LGPD aparece como regra de schema ("não existe coluna para imagem"), não como intenção | O que o *audit* do PAUL cobra: stack fixa, sem filas, nenhuma imagem persistida, consentimento antes da câmera |

**Por que arc42 enxuto e não outro padrão:**
- *arc42 completo* teria seções vazias ou repetidas para um sistema de 3 atores e 5 contêineres.
- *4+1 / ISO 42010* organiza por visões e pontos de vista, mas não traz um lugar natural para requisitos testáveis nem para contratos de API, que são justamente o que o PAUL mais consome.
- *Kiro / Spec Kit* são ótimos para requisitos e tarefas, mas fracos em decisões com alternativas e em limites do sistema.
- O arc42 enxuto combina o melhor de cada um: C4 para limites, MADR para decisões, EARS para requisitos, OpenAPI para contratos e Bass/Clements/Kazman para qualidade, cada peça no padrão que a torna verificável.

## 3. Princípio "a tela é um contrato com a arquitetura"

- Toda ficha de tela cita **requisitos** (`REQ-xx`) e **operações** (`operationId`); toda operação do OpenAPI declara em `x-screens` as telas que a usam e em `x-requirements` os requisitos que implementa.
- Os nomes de campo nas telas (`gloss`, `confidence`, `decision_reason`, …) são os do modelo de dados e do OpenAPI — não "nome" ou "valor".
- A coerência é **verificada por código**: [`../../../scripts/verificar_rastreabilidade.py`](../../../scripts/verificar_rastreabilidade.py) falha se houver requisito citado e inexistente, operação sem tela, tela sem operação, ids pulados ou `x-screens` divergente das fichas. Ele gera [`rastreabilidade.md`](rastreabilidade.md).

```bash
pip install pyyaml
python scripts/verificar_rastreabilidade.py            # só verifica
python scripts/verificar_rastreabilidade.py --write    # verifica e gera a matriz
```

## 4. Relação com o FI-01

Esta especificação parte do FI-01 (Formulário de Ideação). As seções do formulário que estavam em branco — envelope não-funcional (§7), concorrência (§7.1), desempenho (§7.2), dados e privacidade (§8) e modelo de dados (§9) — foram decididas aqui seguindo a lógica do projeto. **Tudo o que foi acrescentado está listado em [`riscos-e-lacunas.md`](riscos-e-lacunas.md) §3 (itens N-01 a N-10)**, e as lacunas que continuam abertas (limiar de confiança, conjunto de sinais, estrutura do grafo, metas de desempenho) estão em §1, com valor provisório e momento de decisão. Assim, na auditoria do `PROJECT.md`, fica claro o que veio do formulário, o que veio desta especificação e o que o agente não pode inventar.

## 5. Referências

- Starke, G.; Hruschka, P. **arc42**. https://arc42.org
- Brown, S. **The C4 model for visualising software architecture**. https://c4model.com
- Nygard, M. **Documenting Architecture Decisions**, 2011. https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions
- **MADR — Markdown Architectural Decision Records**. https://adr.github.io/madr/
- Mavin, A. et al. **Easy Approach to Requirements Syntax (EARS)**. IEEE RE, 2009.
- **OpenAPI Specification 3.1**. https://spec.openapis.org/oas/v3.1.0
- Bass, L.; Clements, P.; Kazman, R. **Software Architecture in Practice**, 4ª ed. Addison-Wesley/SEI, 2021.
- Kruchten, P. **The 4+1 View Model of Architecture**. IEEE Software, 12(6), 1995.
- **ISO/IEC/IEEE 42010:2022** — Architecture description.
- GitHub **Spec Kit**. https://github.com/github/spec-kit · **Kiro — Specs**. https://kiro.dev/docs/specs/concepts
- W3C. **Web Content Accessibility Guidelines (WCAG) 2.2**. https://www.w3.org/TR/WCAG22/
- Brasil. **Lei nº 13.709/2018 (LGPD)** e **Lei nº 10.436/2002 (Libras)**.
- Google. **MediaPipe Hand Landmarker**. https://ai.google.dev/edge/mediapipe/solutions/vision/hand_landmarker
