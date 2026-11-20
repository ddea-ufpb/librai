## P04 · Dificuldades da turma

> **Pós-MVP.**

- **Superfície · ator:** Painel do professor · professor
- **Objetivo:** Descobrir quais sinais a turma mais erra, para revisar em aula.
- **Chega de / vai para:** **P02** (Ver dificuldades da turma) → **P02** (Voltar)
- **Mostra:**
  - `class_name`, período (`since`/`until`, padrão: 30 dias)
  - por sinal (`items[]`): `gloss`, `students_count` (alunos distintos que praticaram), `outcomes_count`, `not_mastered_rate` em %, ou "Dados insuficientes" quando `insufficient_data = true`
- **Coleta:**
  - `since` · data · opcional · `until` · data · opcional
- **Ações:**
  - **Voltar à turma** (primária) → P02
  - **Filtrar período** → `list_class_difficulties?since=&until=` → permanece
- **Estados:**
  - *carregando:* esqueleto (REQ-54)
  - *vazio:* nenhuma prática da turma no período → "Ainda não há práticas desta turma no período" (REQ-60)
  - *erro:* `not_found` (turma de outro professor) → "Turma não encontrada" + P01 (REQ-37) · 5xx → "Tentar de novo" (REQ-57)
  - *sucesso:* sinais ordenados por `not_mastered_rate` decrescente; os de `insufficient_data` ficam no fim (REQ-40)
  - *offline:* dados já carregados visíveis; filtro desabilitado (REQ-51)
  - *sem permissão:* `role` errado → aviso (REQ-07); sessão expirada → G01 (REQ-56)
- **Regras:** taxa só aparece com 3 ou mais alunos distintos no sinal, para não identificar um aluno pela taxa (REQ-40); a lista não mostra nomes de alunos.
- **Requisitos · contrato:** REQ-37, REQ-40 · `GET /classes/{class_id}/difficulties` (`list_class_difficulties`)
- **Acessibilidade:** barras de taxa sempre acompanhadas do número em %; "Dados insuficientes" em texto, não como barra vazia.
