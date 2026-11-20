# ADR-0006 · Orquestrar o agente tutor com LangGraph dentro do processo da API

- **Status:** aceita
- **Data:** 27/10/2026
- **Contexto:** O FI-01 escolheu LangGraph para "representar de forma explícita o estado da sessão, o uso de ferramentas e as decisões do agente" (§6.1) e descartou FastAPI com chamadas diretas a um LLM. É preciso decidir **onde** o grafo roda e **qual estado** ele guarda, sabendo que a sessão já é persistida no SQLite (ADR-0005) e que o FI-01 limita o sistema a um único agente (§10.2).
- **Opções consideradas:**
  1. **Serviço de agente separado** (outro processo/contêiner) chamado pela API.
  2. **Grafo LangGraph com checkpointer próprio** (`SqliteSaver`) guardando o estado da conversa.
  3. **Grafo LangGraph sem checkpointer, invocado por requisição dentro da API**; o estado durável é a `practice_session` no banco, e o `TutorState` do grafo é montado a partir dela a cada tentativa.
- **Decisão:** Opção 3. Um `StateGraph` compilado na inicialização da API, com os nós `validar → reconhecer → decidir → redigir → gravar` ([`ferramentas-agente.md`](../contratos/ferramentas-agente.md) §1). O `TutorState` é efêmero (dura uma requisição); a fonte da verdade é `practice_session` + `attempt`. Arestas condicionais tratam idempotência, `no_hand_detected` e `recognizer_unavailable`.
- **Consequências:**
  - \+ Um único lugar para a verdade do estado (o banco), sem dois estados para sincronizar; a idempotência do ADR-0003 continua valendo.
  - \+ O grafo pode ser testado nó a nó com `FakeRecognizer` e LLM desligado (QS-05), atendendo a primeira fatia do FI-01 §11 (terminal, sem câmera).
  - \+ Sem processo extra para implantar.
  - − Não usa a memória de longo prazo do LangGraph; se o grupo quiser personalização por histórico (fora de escopo hoje), este ADR precisa ser revisto.
  - − A equipe ainda está aprendendo LangGraph (FI-01 §6.2); o grafo fica pequeno (cinco nós) de propósito. A estrutura exata dos nós é lacuna declarada no FI-01 §12, e esta decisão fixa só a fronteira e o estado.
- **Requisitos afetados:** REQ-16, REQ-20, REQ-25, REQ-29.
