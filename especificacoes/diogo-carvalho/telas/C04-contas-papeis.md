## C04 · Contas e papéis

> **Pós-MVP.** No MVP, o primeiro administrador é criado por comando de linha (`python -m librai.create_admin`).

- **Superfície · ator:** Console de curadoria · administrador
- **Objetivo:** Criar contas de professor e de administrador e ativar ou desativar contas.
- **Chega de / vai para:** **C01** (Contas) → **C01** (Voltar)
- **Mostra:** por conta (`items[]`): `display_name`, `email`, `role`, `status`, `created_at`; filtro por `role`.
- **Coleta:** (no diálogo "Nova conta")
  - `display_name` · texto, 2 a 60 caracteres · obrigatório
  - `email` · e-mail · obrigatório
  - `role` · escolha `teacher` ou `admin` · obrigatório
  - `password` · senha provisória, 8 a 128 caracteres · obrigatório
- **Ações:**
  - **Nova conta** (primária) → diálogo → `admin_create_user` → conta na lista
  - **Desativar / Reativar** (em cada linha) → `admin_update_user` com `status` → linha atualizada
  - **Mudar papel** (em cada linha) → `admin_update_user` com `role` → linha atualizada
  - **Voltar** → C01
- **Estados:**
  - *carregando:* esqueleto da tabela (REQ-54)
  - *vazio:* filtro sem resultado → "Nenhuma conta com esse papel" (REQ-60)
  - *erro:* `email_taken` (REQ-59) · `invalid_payload` em senha (REQ-06) · `last_admin` → "Não é possível remover o último administrador ativo" (REQ-47) · 5xx → "Tentar de novo" (REQ-57)
  - *sucesso:* conta criada ou alterada → linha destacada e anunciada (REQ-61)
  - *offline:* lista visível; ações desabilitadas (REQ-51)
  - *sem permissão:* `role` diferente de `admin` → aviso (REQ-07); sessão expirada → G01 (REQ-56)
- **Regras:** contas de professor e de administrador só nascem aqui (REQ-04, REQ-59); nunca fica sem administrador ativo (REQ-47); a lista não mostra nem permite ver senha.
- **Requisitos · contrato:** REQ-06, REQ-47, REQ-59 · `GET /admin/users` (`admin_list_users`), `POST /admin/users` (`admin_create_user`), `PATCH /admin/users/{user_id}` (`admin_update_user`)
- **Acessibilidade:** ações destrutivas (desativar, remover papel `admin`) com confirmação textual e botão rotulado com o nome da conta ("Desativar Maria Souza").
