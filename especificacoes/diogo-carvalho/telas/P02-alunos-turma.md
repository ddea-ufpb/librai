## P02 · Alunos da turma

> **Pós-MVP.**

- **Superfície · ator:** Painel do professor · professor
- **Objetivo:** Ver quem da turma está praticando e como, para decidir quem acompanhar de perto.
- **Chega de / vai para:** **P01** (abrir turma), **P03** e **P04** (Voltar) → **P03** (abrir aluno) · **P04** (Ver dificuldades da turma) · **P01** (Voltar)
- **Mostra:**
  - cabeçalho: `class_name`, `invite_code`, período (`since`/`until`, padrão: últimos 30 dias)
  - por aluno (`items[]`): `display_name`, `sessions_count`, `signs_mastered_count`, `signs_not_mastered_count`, `last_practice_at`
- **Coleta:**
  - `since` · data · opcional · `until` · data · opcional (filtro de período)
- **Ações:**
  - **Abrir aluno** (primária, em cada linha) → P03 com `student_id`
  - **Ver dificuldades da turma** → P04 com `class_id`
  - **Filtrar período** → `list_class_students?since=&until=` → permanece
  - **Voltar** → P01
- **Estados:**
  - *carregando:* esqueleto da tabela (REQ-54)
  - *vazio:* turma sem alunos → "Nenhum aluno entrou ainda. Compartilhe o código `invite_code`." · alunos sem prática no período → linha com "Sem prática no período", sem números zerados enganosos (REQ-60)
  - *erro:* `not_found` (turma de outro professor ou inexistente) → "Turma não encontrada" + P01 (REQ-37) · 5xx → "Tentar de novo" (REQ-57)
  - *sucesso:* tabela ordenada por `last_practice_at` decrescente (REQ-61)
  - *offline:* dados já carregados visíveis; filtro desabilitado (REQ-51)
  - *sem permissão:* `role` errado → aviso (REQ-07); sessão expirada → G01 (REQ-56)
- **Regras:** só alunos com `class_enrollment` na turma; números derivados de `sign_outcome`/`attempt`, sem imagem ou vídeo (REQ-39); sem ranking: a ordenação padrão é por data, não por desempenho (diretrizes §5).
- **Requisitos · contrato:** REQ-37, REQ-39 · `GET /classes/{class_id}/students` (`list_class_students`)
- **Acessibilidade:** tabela semântica com `<th scope>`; ordenação anunciada (`aria-sort`); funciona em tablet sem rolagem horizontal da página (colunas secundárias recolhíveis).
