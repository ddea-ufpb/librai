# ADR-0007 · Servir as três superfícies num único aplicativo web com rotas por papel

- **Status:** aceita
- **Data:** 27/10/2026
- **Contexto:** O sistema tem três atores com superfícies diferentes — app do aluno (celular primeiro, com câmera), painel do professor (desktop/tablet) e console de curadoria (desktop) — além do acesso comum. O FI-01 escolheu Next.js/React (§6), e a equipe é pequena (60 h).
- **Opções consideradas:**
  1. **Três aplicações separadas** (aluno, professor, curadoria).
  2. **App nativo para o aluno** + web para professor e curadoria.
  3. **Um único app Next.js com grupos de rotas por papel** (`/(aluno)`, `/(professor)`, `/(curadoria)`), layout responsivo e guarda de rota por `role`.
- **Decisão:** Opção 3. Depois do login (G01), `get_me` devolve `role` e o app abre A01, P01 ou C01. A autorização de verdade é sempre da API (REQ-07); a guarda de rota no cliente é só navegação. O código de MediaPipe é carregado apenas nas rotas que usam câmera (A02, A03), para não pesar nas demais.
- **Consequências:**
  - \+ Um build, um design system, um cliente HTTP gerado a partir do `openapi.yaml`.
  - \+ O web funciona no celular sem loja de aplicativos; a câmera pelo navegador basta para landmarks (ADR-0002).
  - − Um bug no layout comum afeta as três superfícies.
  - − Sem recursos nativos (notificação push, câmera em segundo plano); não são necessários para o escopo.
- **Requisitos afetados:** REQ-07, REQ-56.
