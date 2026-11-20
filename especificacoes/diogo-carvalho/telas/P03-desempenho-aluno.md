## P03 · Desempenho do aluno

> **Pós-MVP.**

- **Superfície · ator:** Painel do professor · professor
- **Objetivo:** Entender em quais sinais um aluno específico tem dificuldade.
- **Chega de / vai para:** **P02** (abrir aluno) → **P02** (Voltar)
- **Mostra:**
  - `display_name`, período (`since`/`until`)
  - sessões (`sessions[]`): `started_at`, `status`, `signs_mastered_count`, `signs_total`
  - por sinal (`signs[]`): `gloss`, `attempts_total`, `mastered_count`, `not_mastered_count`, `skipped_count`, `avg_attempts_to_master`
- **Coleta:**
  - `since` · data · opcional · `until` · data · opcional
- **Ações:**
  - **Voltar à turma** (primária) → P02
  - **Filtrar período** → `get_student_performance?since=&until=` → permanece
- **Estados:**
  - *carregando:* esqueleto (REQ-54)
  - *vazio:* aluno sem prática no período → "Este aluno ainda não praticou neste período" (REQ-60)
  - *erro:* `not_found` (aluno fora das turmas do professor) → "Aluno não encontrado" + P02 (REQ-37) · 5xx → "Tentar de novo" (REQ-57)
  - *sucesso:* lista de sinais ordenada por `not_mastered_count` decrescente (REQ-61)
  - *offline:* dados já carregados visíveis; filtro desabilitado (REQ-51)
  - *sem permissão:* `role` errado → aviso (REQ-07); sessão expirada → G01 (REQ-56)
- **Regras:** o professor só acessa alunos matriculados em turma sua (REQ-37); nenhuma imagem, vídeo ou `landmark` existe para exibir (REQ-11, REQ-39); `feedback_text` individual das tentativas não é exibido — o painel é agregado.
- **Requisitos · contrato:** REQ-11, REQ-37, REQ-39 · `GET /classes/{class_id}/students/{student_id}/performance` (`get_student_performance`)
- **Acessibilidade:** números com rótulo ("dominou em 3 de 5 tentativas"); se houver gráfico, há tabela equivalente ao lado.
