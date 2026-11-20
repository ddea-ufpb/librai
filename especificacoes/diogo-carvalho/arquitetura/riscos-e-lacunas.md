# LibrAI · Riscos, lacunas e diferenças em relação ao FI-01 (arc42 §11)

> O FI-01 diz: "o que você não escrever aqui, o agente vai inventar". Este arquivo separa o que **veio do FI-01**, o que **foi decidido nesta especificação** e o que **continua em aberto**, para que a auditoria do `PROJECT.md` saiba onde olhar.

## 1. Lacunas declaradas (decisões ainda não tomadas)

| ID | O que ainda não está decidido | Valor provisório na especificação | Por que ainda não | Quando decidir |
|---|---|---|---|---|
| L-01 | Estrutura exata do grafo LangGraph (FI-01 §12) | 5 nós: validar → reconhecer → decidir → redigir → gravar (ADR-0006) | Equipe aprendendo o framework | Durante a primeira fatia |
| L-02 | Limiar de confiança (FI-01 §12) | `CONFIDENCE_THRESHOLD = 0,80` | Modelo real ainda não medido no conjunto do MVP | Quando o modelo real for integrado; medir a curva de acerto × limiar em fixtures rotuladas |
| L-03 | Conjunto exato de 5 a 10 sinais (FI-01 §12) | Letras **estáticas** do alfabeto manual que o Random Forest atual já reconhece, cadastradas pelo curador em C02; `SESSION_SIZE = 5` | Depende de quais sinais o modelo atual reconhece com confiança | Antes da integração do modelo de visão |
| L-04 | Metas de desempenho (FI-01 §7.2) | p95 ≤ 2.000 ms, p99 ≤ 4.000 ms com 10 sessões (QS-04) | Baseline não medido | Após a primeira rodada de QS-04 |
| L-05 | ~~Formato de entrada do modelo existente~~ **resolvida** | O LibrAI original classifica 21 landmarks de MediaPipe com Random Forest (`models/librai_rf.joblib`) → compatível com ADR-0002 | — | Resolvida em 27/10/2026 ao conferir o repositório LibrAI |
| L-08 | Formato da referência das letras | `reference_media` aceita vídeo (mp4/webm) | O LibrAI original usa imagens estáticas (`assets/references/A.png`); para letras estáticas uma imagem basta, mas o vídeo deixa o catálogo pronto para sinais dinâmicos | Na consolidação do grupo: aceitar também `image/png` ou converter as imagens em vídeos curtos |
| L-06 | Provedor de LLM e modelo | `LIBRAI_LLM_PROVIDER` configurável; padrão `none` | Conta Claude Pro agora, modelo local na Semana 8 (FI-01 §10.3) | Semana 8 |
| L-07 | Política de exclusão de conta pelo aluno | Apagar em cascata sessões e tentativas, manter `consent` por 5 anos | Não há tela de exclusão especificada | Na consolidação do grupo |

## 2. Riscos

| ID | Risco | Probabilidade · impacto | Mitigação | Gatilho para revisar |
|---|---|---|---|---|
| R-01 | Um modelo futuro (sinais dinâmicos) exigir imagem, e não pontos | baixa · alto | O modelo atual já usa pontos (L-05); ADR-0002 prevê o plano B (quadros em memória, sem persistência); `recognize_sign` isola o formato | Troca de modelo |
| R-07 | O modelo atual foi treinado só com a **mão esquerda**, palma para a câmera, e só com letras **estáticas** | alta · médio | `recognize_sign` espelha a mão direita (`x → 1 − x`); A03 instrui "palma para a câmera"; o vocabulário do MVP (L-03) se limita a letras estáticas que o modelo já reconhece; sinais com movimento (ex.: J, Z, H, X) ficam fora | Taxa de `low_confidence` > 30% para destros em testes |
| R-02 | MediaPipe lento em celulares de entrada | média · médio | A02 detecta falha e lentidão; quadros reduzidos a 15 fps (`fps` aceita de 10 a 60) | Mais de 10% de `no_hand_detected` em testes de campo |
| R-03 | O modelo reconhece mal variantes regionais e o agente "reprova" quem sinaliza certo | alta · alto | `regional_variant` visível; limite de 3 tentativas (ADR-0008); reporte `regional_variant` (REQ-35); feedback nunca diz "errado", diz "não reconhecido" | Reportes `regional_variant` > 3 por sinal |
| R-04 | O LLM gera texto inadequado ou contraditório com a decisão | baixa · médio | O LLM não decide (ADR-0001); texto limitado a 280 caracteres; o motivo da decisão aparece ao lado; fallback para template | Qualquer reporte de feedback inadequado |
| R-05 | Gargalo de escrita no SQLite | baixa · médio | WAL + `BEGIN IMMEDIATE` (ADR-0005) | p95 de espera por lock > 100 ms em QS-04 |
| R-06 | Escopo: as telas de professor e curadoria consomem as 60 h | média · alto | Marcadas como pós-MVP no inventário; o MVP mínimo de curadoria é C01/C02 | Planejamento do grupo |

## 3. O que esta especificação acrescentou ao FI-01

Itens **[novo]** — não estavam no formulário e devem aparecer no `PROJECT.md` como decisões desta especificação, não como algo que o FI-01 disse:

| # | Acréscimo | Onde | Motivo |
|---|---|---|---|
| N-01 | Extrair landmarks no navegador; nenhuma imagem sai do aparelho | ADR-0002 | Torna "0 imagens persistidas" (FI-01 §10.1) uma garantia da arquitetura |
| N-02 | Limite de 3 tentativas por sinal → `ADVANCE` com `not_mastered` | ADR-0008, REQ-19 | Vivacidade; é **exceção à AC-2** do FI-01 |
| N-03 | "Pular sinal" como escolha do aluno | A04, REQ-23 | Feature 4 do FI-01 ("escolha entre repetir ou seguir") |
| N-04 | Idempotência por `client_attempt_id` + versão otimista | ADR-0003 | FI-01 §7.1 (concorrência) estava em branco |
| N-05 | Pool de processos com fila limitada e 503 | ADR-0004 | FI-01 §7.1 e §7.2 estavam em branco |
| N-06 | SQLite em modo WAL | ADR-0005 | Detalha a escolha do FI-01 |
| N-07 | Consentimento da câmera versionado e revogável | A02, REQ-08, REQ-12 | FI-01 §8 (privacidade) estava em branco |
| N-08 | Curadoria com status, licença e consentimento de quem aparece na referência | C01–C03, REQ-41–REQ-46 | Partes afetadas do FI-01 §3.2 |
| N-09 | Turmas com código de convite e agregados com mínimo de 3 alunos | A08, P01–P04, REQ-36–REQ-40 | Ator professor do FI-01 §3.1; minimização |
| N-10 | Modelo de dados completo | `contratos/modelo-dados.md` | FI-01 §9 estava em branco |

## 4. Dívida técnica aceita

- Sem réplica nem backup automático do SQLite: backup manual com `sqlite3 .backup` antes de cada demonstração.
- Sem recuperação de senha (não há provedor de e-mail no escopo nem operação de redefinição); fica para a consolidação do grupo.
- Sem internacionalização: só pt-BR.
