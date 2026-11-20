## C03 · Reportes de conteúdo

> **Pós-MVP.** Canal de correção para a comunidade de usuários (parte afetada do FI-01 §3.2).

- **Superfície · ator:** Console de curadoria · administrador / curador
- **Objetivo:** Tratar os reportes de alunos sobre sinais incorretos, regionais ou inadequados.
- **Chega de / vai para:** **C01** (Reportes abertos), **C02** (Voltar) → **C02** (Corrigir sinal) · **C01** (Voltar)
- **Mostra:**
  - filtro por `status` (padrão: `open`)
  - por reporte (`items[]`): `sign_gloss`, `reason`, `comment`, `created_at`, `status`, `resolution_note`, `resolved_at`
- **Coleta:** (no diálogo "Resolver")
  - `status` · escolha `resolved` ("corrigido") ou `dismissed` ("sem alteração") · obrigatório
  - `resolution_note` · texto, 5 a 500 caracteres · obrigatório
- **Ações:**
  - **Resolver** (primária, em cada reporte `open`) → diálogo → `admin_resolve_report` → o reporte sai da lista de abertos
  - **Corrigir sinal** → C02 com `sign_id`
  - **Filtrar** → `admin_list_reports?status=` → permanece
  - **Voltar** → C01
- **Estados:**
  - *carregando:* esqueleto (REQ-54)
  - *vazio:* nenhum reporte aberto → "Nenhum reporte aberto" (REQ-60)
  - *erro:* `invalid_payload` (sem `resolution_note`) → mensagem no campo (REQ-46) · `not_found` (reporte já removido) → recarrega a lista · 5xx → "Tentar de novo", nota preservada (REQ-57)
  - *sucesso:* reporte resolvido → removido da lista com confirmação anunciada (REQ-61)
  - *offline:* lista visível; "Resolver" desabilitado (REQ-51)
  - *sem permissão:* `role` diferente de `admin` → aviso (REQ-07); sessão expirada → G01 (REQ-56)
- **Regras:** resolução sempre com nota (REQ-46); o nome de quem reportou não aparece — só o motivo e o comentário (minimização de dados); reportes do tipo `regional_variant` são resolvidos preferencialmente ajustando `regional_variant` no C02, não removendo o sinal.
- **Requisitos · contrato:** REQ-35, REQ-46 · `GET /admin/reports` (`admin_list_reports`), `PATCH /admin/reports/{report_id}` (`admin_resolve_report`)
- **Acessibilidade:** `reason` em texto; o diálogo prende o foco e devolve o foco à linha seguinte ao fechar.
