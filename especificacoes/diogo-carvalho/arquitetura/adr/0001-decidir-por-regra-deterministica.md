# ADR-0001 · Decidir REPETIR/AVANÇAR por regra determinística e usar o LLM só para redigir o feedback

- **Status:** aceita
- **Data:** 27/10/2026
- **Contexto:** O núcleo do LibrAI é o ciclo do agente tutor: apresentar o sinal → chamar a ferramenta de reconhecimento → observar o resultado → decidir a próxima ação (FI-01 §4.3). O FI-01 restringe as decisões a **repetir** ou **avançar** (§10.2), pede **100% dos cenários de teste** do fluxo (§10.1) e declara a meta de "evitar que decisões simples do MVP dependam excessivamente do modelo de linguagem" (§6.2). A equipe tem domínio baixo-médio em LangGraph. Um LLM decidindo diretamente seria não determinístico, difícil de testar e poderia avançar um aluno que errou.
- **Opções consideradas:**
  1. **LLM decide** (tool calling livre: o modelo escolhe repetir ou avançar a partir do resultado).
  2. **LLM decide com saída estruturada** restrita ao enum `REPEAT|ADVANCE`, com a regra no prompt.
  3. **Regra determinística decide; LLM só redige o texto do feedback**, com fallback para texto-modelo.
- **Decisão:** Opção 3. A função pura `decide_next` ([`ferramentas-agente.md`](../contratos/ferramentas-agente.md) §3) calcula `decision` e `decision_reason` a partir de `recognized_label`, `confidence`, `confidence_threshold`, `attempt_number` e `max_attempts_per_sign`. O nó `generate_feedback` chama o LLM só para o texto (≤ 280 caracteres), com timeout de 3 s e fallback por `decision_reason`. O LLM nunca recebe dado pessoal e não pode alterar a decisão (REQ-20, REQ-21).
- **Consequências:**
  - \+ Decisão 100% testável com uma tabela de quatro linhas (QS-05); AC-1 e AC-2 do FI-01 viram testes unitários sem LLM.
  - \+ O sistema funciona sem LLM (`LIBRAI_LLM_PROVIDER=none`), o que atende a primeira fatia (terminal, ferramenta simulada) e a troca para modelo local na Semana 8.
  - \+ A latência do LLM não bloqueia a decisão; no pior caso, o feedback cai no texto-modelo.
  - − O "agente" decide menos: a parte agentic se resume a orquestrar ferramenta, observação e decisão por regra. É uma escolha consciente para o MVP; personalizar a decisão por histórico está fora de escopo (FI-01 §5).
  - − O texto do LLM pode contradizer a decisão (ex.: elogiar quando a decisão é `REPEAT`); o cliente sempre mostra o motivo derivado de `decision_reason` ao lado do texto (P5 das diretrizes), e o prompt recebe a decisão já tomada.
- **Requisitos afetados:** REQ-17, REQ-18, REQ-19, REQ-20, REQ-21.
