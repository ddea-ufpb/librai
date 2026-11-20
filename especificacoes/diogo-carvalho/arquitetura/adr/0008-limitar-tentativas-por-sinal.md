# ADR-0008 · Limitar a 3 tentativas por sinal para garantir o término da sessão

- **Status:** aceita
- **Data:** 27/10/2026
- **Contexto:** Pelos critérios AC-1 e AC-2 do FI-01, o agente avança após acerto e repete após erro. Se valesse só isso, um aluno que não consegue executar um sinal — ou um sinal que o modelo reconhece mal — ficaria preso para sempre: a sessão violaria a **vivacidade** ("toda sessão termina") e a prática viraria frustração, o contrário da aposta central. O FI-01 também prevê que o aluno escolha seguir para o próximo sinal (feature 4).
- **Opções consideradas:**
  1. **Repetir indefinidamente** (AC-2 literal), confiando que o aluno vai pular ou encerrar.
  2. **Limite de tentativas por sinal**, após o qual o agente avança e marca o sinal como não dominado.
  3. **Limite de tempo por sinal.**
- **Decisão:** Opção 2, com `MAX_ATTEMPTS_PER_SIGN = 3`, copiado para a sessão no início (REQ-13). Na tentativa de número 3 que seria `REPEAT`, o agente decide `ADVANCE` com `decision_reason = max_attempts_reached` e grava `sign_outcome = not_mastered` (REQ-19). Independentemente disso, o aluno pode pular a qualquer momento em A04 (`skip_sign`, REQ-23) — escolha do aluno, não decisão do agente.
- **Consequências:**
  - \+ Vivacidade garantida: toda sessão termina em no máximo `SESSION_SIZE × MAX_ATTEMPTS_PER_SIGN` = 15 tentativas válidas.
  - \+ `not_mastered` alimenta as dificuldades do professor (REQ-40) com dado mais útil que "pulou".
  - − Exceção explícita à AC-2 do FI-01: precisa ser comunicada no `PROJECT.md` para o agente não "corrigir" de volta.
  - − O número 3 é escolha de produto (equilíbrio entre insistência e frustração), não medido; revisar com dados de uso.
- **Requisitos afetados:** REQ-13, REQ-18, REQ-19, REQ-23.
