# ADR-0003 · Garantir idempotência da tentativa por `client_attempt_id` e serializar a sessão com versão otimista

- **Status:** aceita
- **Data:** 27/10/2026
- **Contexto:** O componente concorrente do LibrAI sai do problema: o aluno pratica no celular, em rede instável. (a) O cliente reenvia a mesma tentativa quando a resposta se perde; (b) o aluno abre duas abas ou dá duplo toque em "Gravar"; (c) "Pular" (A04) e uma tentativa podem chegar juntos. O **recurso compartilhado** é a linha de `practice_session` (`current_position`, `status`) e o contador de tentativas do sinal atual. Propriedades:
  - **Segurança (nunca pode acontecer):** uma mesma tentativa gerar duas decisões; a sessão avançar duas vezes por uma tentativa; existir tentativa gravada sem a mudança de estado correspondente, ou o contrário; um aluno ter duas sessões ativas.
  - **Vivacidade (tem que acabar acontecendo):** toda tentativa aceita recebe uma resposta (resultado, 409 ou 503) em até 5 s; toda sessão termina (ADR-0008).
- **Opções consideradas:**
  1. **Lock em memória por sessão** (`asyncio.Lock` num dicionário) — não sobrevive a reinício e não vale para mais de um processo Uvicorn.
  2. **Fila por sessão** (Redis/Celery) — descartada pelo FI-01 §6 ("nenhuma fila no MVP").
  3. **Chave de idempotência gerada no cliente + controle otimista por `version` + restrições únicas no banco.**
- **Decisão:** Opção 3.
  - O cliente gera `client_attempt_id` (UUID) por clique e o reutiliza em reenvios; `UNIQUE (session_id, client_attempt_id)` no banco; reenvio devolve 200 com a tentativa existente (REQ-25).
  - O cliente envia `expected_sign_id`; se não for o sinal atual, a API responde `stale_sign` (REQ-26).
  - A gravação faz `UPDATE practice_session ... SET version = version + 1 WHERE id = ? AND version = ?` na mesma transação do `INSERT attempt` (REQ-30); 0 linhas afetadas → rollback e reavaliação.
  - Índice parcial único `ux_session_one_active` garante uma sessão ativa por aluno (REQ-15).
  - O mesmo padrão de `version` vale para a edição de sinais por dois curadores (REQ-43).
- **Consequências:**
  - \+ As invariantes de segurança ficam no banco; um bug no código não consegue violá-las.
  - \+ Sem infraestrutura extra (sem Redis), coerente com o FI-01.
  - \+ Testável com N requisições simultâneas (QS-03).
  - − Em corrida real, uma das requisições paga uma inferência "desperdiçada" antes de perder o `version`; aceitável, porque corrida é exceção (duas abas).
  - − O cliente precisa tratar 409 `stale_sign` recarregando o estado (A03/A04 especificam isso).
- **Requisitos afetados:** REQ-15, REQ-25, REQ-26, REQ-30, REQ-43.
