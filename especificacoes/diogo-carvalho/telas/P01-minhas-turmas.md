## P01 · Minhas turmas

> **Pós-MVP** (dashboard do professor, FI-01 §5).

- **Superfície · ator:** Painel do professor · professor
- **Objetivo:** Ver as turmas do professor e criar uma nova com código de convite.
- **Chega de / vai para:** **G01** (`role = teacher`) → **P02** (abrir turma) · **G01** (Sair)
- **Mostra:** por turma (`items[]`): `name`, `invite_code` (com botão copiar), `students_count`, `last_practice_at` (prática mais recente de qualquer aluno da turma), `created_at`.
- **Coleta:** (no diálogo "Nova turma")
  - `name` · texto, 3 a 80 caracteres · obrigatório
- **Ações:**
  - **Nova turma** (primária) → diálogo → **Criar** → `create_class` → a turma aparece na lista com o `invite_code` em destaque
  - **Abrir turma** (em cada linha) → P02 com `class_id`
  - **Copiar código** → copia `invite_code` para a área de transferência → "Código copiado"
  - **Sair** → `logout` → G01
- **Estados:**
  - *carregando:* esqueleto da lista (REQ-54)
  - *vazio:* nenhuma turma → "Crie sua primeira turma e compartilhe o código com os alunos" + "Nova turma" (REQ-60)
  - *erro:* `invalid_payload` em `name` → mensagem no campo · 5xx → "Tentar de novo", nome preservado (REQ-57)
  - *sucesso:* turma criada → linha nova destacada e anunciada (REQ-38)
  - *offline:* lista já carregada visível; "Nova turma" desabilitado (REQ-51)
  - *sem permissão:* `role` diferente de `teacher` → aviso + link para a área certa (REQ-07); sessão expirada → G01 (REQ-56)
- **Regras:** o professor só vê as turmas em que `teacher_id` é ele (REQ-37); o `invite_code` é gerado pelo sistema e nunca digitado pelo professor (REQ-38).
- **Requisitos · contrato:** REQ-07, REQ-37, REQ-38 · `GET /classes` (`list_classes`), `POST /classes` (`create_class`), `POST /auth/logout` (`logout`)
- **Acessibilidade:** o `invite_code` aparece em fonte monoespaçada e é soletrado pelo leitor de tela (`aria-label` com espaços entre os caracteres); a confirmação de cópia é anunciada por `aria-live`.
