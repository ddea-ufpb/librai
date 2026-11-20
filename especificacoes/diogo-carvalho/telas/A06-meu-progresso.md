## A06 · Meu progresso

> **Pós-MVP** (feature 5 do FI-01). Visível só com a funcionalidade `progress_history` habilitada (REQ-32).

- **Superfície · ator:** App do aluno · aluno
- **Objetivo:** Ver, sinal por sinal, como a prática evoluiu entre sessões.
- **Chega de / vai para:** **A01** (Meu progresso) → **A07** (abrir um sinal) · **A01** (Voltar)
- **Mostra:**
  - período considerado (padrão: todas as sessões)
  - por sinal praticado (`items[]`): `gloss`, `sessions_count`, `mastered_count`, `last_practiced_at`
  - total de sessões `finished` e `abandoned`
- **Coleta:**
  - `since` · data · opcional (filtro "desde")
- **Ações:**
  - **Abrir sinal** (primária, em cada linha) → A07 com `sign_id`
  - **Filtrar período** → `get_my_progress?since=` → permanece
  - **Voltar** → A01
- **Estados:**
  - *carregando:* esqueleto da lista (REQ-54)
  - *vazio:* nenhuma sessão concluída → "Seu progresso aparece aqui depois da primeira prática" + botão para A01 (REQ-60)
  - *erro:* `feature_disabled` → a tela não é oferecida; acesso direto mostra "Esta área ainda não está disponível" (REQ-32) · 5xx → "Tentar de novo" (REQ-57)
  - *sucesso:* lista carregada, ordenada por `last_practiced_at` decrescente (REQ-61)
  - *offline:* dados já carregados continuam visíveis; filtro desabilitado (REQ-51)
  - *sem permissão:* `role` errado → aviso (REQ-07); sessão expirada → G01 (REQ-56)
- **Regras:** o aluno vê apenas os próprios dados (REQ-07); a tela não sugere trilha personalizada — personalização por histórico está fora de escopo (FI-01 §5).
- **Requisitos · contrato:** REQ-32 · `GET /me/progress` (`get_my_progress`)
- **Acessibilidade:** números com rótulo explícito ("dominado em 2 de 3 sessões"), nunca só um gráfico; se houver gráfico, ele tem tabela equivalente.
