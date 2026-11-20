## A08 · Entrar em turma

> **Pós-MVP.** Necessária para o painel do professor (P01–P04) ter alunos.

- **Superfície · ator:** App do aluno · aluno
- **Objetivo:** Vincular o aluno à turma do professor usando o código de convite.
- **Chega de / vai para:** **A01** (Entrar em turma) → **A01** (entrou na turma ou Voltar)
- **Mostra:** explicação: "Ao entrar na turma, o professor passa a ver quantas práticas você fez e quais sinais você dominou. Ele **não** vê vídeo nem imagem sua." (REQ-39); turmas em que já está.
- **Coleta:**
  - `invite_code` · texto, 8 caracteres, sem diferenciar maiúsculas e minúsculas · obrigatório
- **Ações:**
  - **Entrar na turma** (primária) → `join_class` → A01 com "Você entrou na turma `class_name`"
  - **Voltar** → A01
- **Estados:**
  - *carregando:* botão desabilitado com indicador (REQ-54)
  - *vazio:* campo em branco; botão habilitado com 8 caracteres (REQ-60)
  - *erro:* `class_not_found` → "Código não encontrado. Confira com seu professor." (REQ-36) · 5xx → "Tentar de novo", código preservado (REQ-57)
  - *sucesso:* 201 → volta para A01 com confirmação · 200 (já matriculado) → "Você já está nesta turma" (REQ-36)
  - *offline:* botão desabilitado (REQ-51)
  - *sem permissão:* `role` errado → aviso (REQ-07); sessão expirada → G01 (REQ-56)
- **Regras:** o código não diferencia maiúsculas e minúsculas e ignora espaços; entrar duas vezes não duplica a matrícula (REQ-36).
- **Requisitos · contrato:** REQ-36, REQ-39 · `POST /class-enrollments` (`join_class`)
- **Acessibilidade:** campo com `autocapitalize="characters"` e `inputmode="text"`; o erro diz como corrigir, não só que falhou (WCAG 3.3.3).
