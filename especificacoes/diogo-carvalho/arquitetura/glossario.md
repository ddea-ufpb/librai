# LibrAI · Glossário (arc42 §12)

> Vocabulário do domínio e da arquitetura. Quando há nome no modelo de dados ou no contrato, ele aparece em `monoespaçado` — é esse o nome que telas, código e testes usam.

## Domínio (Libras e ensino)

| Termo | Definição | No modelo |
|---|---|---|
| **Libras** | Língua Brasileira de Sinais, reconhecida pela Lei 10.436/2002; língua visual-espacial com gramática própria, não é "português sinalizado" | — |
| **Sinal** | Unidade lexical de Libras que o sistema ensina; no MVP, sinais isolados (não frases) | `sign` |
| **Glosa** | Palavra em português, em MAIÚSCULAS, usada para nomear um sinal (ex.: LETRA-B; no MVP, letras do alfabeto manual) | `sign.gloss` |
| **Variante regional** | Forma de um sinal usada numa região; o sistema nunca a trata como a única correta | `sign.regional_variant` |
| **Parâmetros do sinal** | Configuração de mão, ponto de articulação, movimento, orientação e expressão facial; **não** são avaliados separadamente (fora de escopo) | — |
| **Vídeo de referência** | Gravação de uma pessoa fluente executando o sinal, com licença e autorização de imagem | `reference_media` |
| **Crédito da fonte** | Quem executa ou produziu a referência; sempre visível ao aluno | `sign.source_credit`, `reference_media.author_credit` |
| **Licença** | Termo que autoriza o uso da referência (ex.: CC BY 4.0, autorização escrita) | `reference_media.license` |
| **Ordem curricular** | Posição do sinal na sequência de ensino, usada para montar a sessão | `sign.curriculum_order` |
| **Turma** | Grupo de alunos acompanhado por um professor | `class_group` |
| **Código de convite** | 8 caracteres que o aluno digita para entrar na turma | `class_group.invite_code` |

## Prática e agente

| Termo | Definição | No modelo |
|---|---|---|
| **Sessão de prática** | Sequência curta (5 sinais) conduzida pelo agente tutor; `active`, `finished` ou `abandoned` | `practice_session` |
| **Sinal atual** | Sinal na posição `current_position` de `sign_ids` | `practice_session.current_position` |
| **Tentativa** | Uma execução do sinal pelo aluno, capturada em até 3 s e avaliada; guarda só o resultado | `attempt` |
| **Landmarks** | Coordenadas (x, y, z) de 21 pontos de cada mão por quadro, extraídas no navegador pelo MediaPipe Hands; nunca persistidas | `AttemptRequest.landmarks` |
| **MediaPipe Hands** | Biblioteca do Google que detecta as mãos e seus pontos numa imagem, executada no navegador | — |
| **Agente tutor** | Grafo LangGraph único que valida, chama o reconhecedor, decide e redige o feedback | — (ADR-0006) |
| **Ferramenta** | Função que o agente chama: `recognize_sign`, `decide_next`, `generate_feedback` | `ferramentas-agente.md` |
| **Reconhecimento** | Classificação dos landmarks em um rótulo com confiança | `recognize_sign` |
| **Rótulo do reconhecedor** | Identificador da classe no modelo de visão (ex.: `B`, classe do Random Forest do LibrAI); liga o sinal do catálogo à saída do modelo | `sign.recognizer_label`, `attempt.recognized_label` |
| **Confiança** | Probabilidade (0 a 1) que o modelo atribui ao rótulo reconhecido | `attempt.confidence` |
| **Limiar de confiança** | Confiança mínima para considerar a tentativa correta; provisório 0,80 | `practice_session.confidence_threshold` |
| **Decisão** | Escolha do agente após a tentativa: `REPEAT` (mesmo sinal) ou `ADVANCE` (próximo sinal) | `attempt.decision` |
| **Motivo da decisão** | `correct`, `wrong_sign`, `low_confidence` ou `max_attempts_reached` | `attempt.decision_reason` |
| **Feedback** | Texto curto ao aluno sobre a tentativa, do LLM (`llm`) ou do texto-modelo (`template`) | `attempt.feedback_text`, `feedback_source` |
| **Pular** | Escolha do aluno (não do agente) de seguir sem dominar o sinal | `sign_outcome.outcome = skipped` |
| **Resultado do sinal** | Como o sinal terminou na sessão: `mastered`, `not_mastered` ou `skipped` (e `not_reached` no resumo) | `sign_outcome` |
| **Taxa de não domínio** | (`not_mastered` + `skipped`) ÷ total de resultados de um sinal numa turma | `not_mastered_rate` |

## Curadoria, contas e privacidade

| Termo | Definição | No modelo |
|---|---|---|
| **Curadoria** | Ciclo de vida do sinal: `draft → in_review → published → archived` | `sign.status` |
| **Publicado** | Sinal visível a alunos e usado em novas sessões | `status = published` |
| **Reporte** | Aviso de um aluno sobre um sinal incorreto, regional ou inadequado | `content_report` |
| **Papel** | `student`, `teacher` ou `admin`; define as telas e as operações permitidas | `user_account.role` |
| **Consentimento da câmera** | Autorização informada, versionada e revogável para processar a câmera | `consent` (`kind = camera_processing`) |
| **Dado pessoal sensível** | Na LGPD (art. 5º, II), dado que pode revelar saúde ou biometria — caso da imagem do aluno | — |

## Arquitetura e processo

| Termo | Definição |
|---|---|
| **arc42** | Modelo de documentação de arquitetura em 12 seções (Starke e Hruschka), usado aqui como índice |
| **C4** | Modelo de diagramas em níveis: contexto (1), contêineres (2), componentes (3), código (4) (Simon Brown) |
| **ADR / MADR** | Registro de decisão arquitetural com contexto, opções, decisão e consequências (Nygard; formato MADR) |
| **EARS** | Sintaxe de requisitos em cinco moldes: ubíquo, evento, estado, indesejado, opcional (Mavin et al.) |
| **Cenário de qualidade** | Fonte, estímulo, ambiente, artefato, resposta e medida (Bass, Clements e Kazman) |
| **Envelope** | Os cinco itens não-funcionais exigidos pela disciplina: persistência, API com contrato, concorrência, desempenho medido, testes |
| **PAUL** | Fluxo da disciplina em que `/paul:init` gera o `PROJECT.md` e `/paul:plan` gera planos de tarefas a partir desta especificação |
| **Idempotência** | Repetir a mesma requisição (mesmo `client_attempt_id`) produz o mesmo efeito de uma vez só |
| **Controle otimista** | Gravar só se `version` não mudou desde a leitura; senão, conflito (409) |
| **Pool de reconhecimento** | Conjunto fixo de processos Python que executam a inferência, com fila limitada |
| **WAL** | *Write-Ahead Logging* do SQLite: permite leituras concorrentes com uma escrita |
| **p95 / p99** | Valor abaixo do qual estão 95% / 99% das medições |
| **Superfície** | Conjunto de telas para um contexto de uso: Acesso (G), App do aluno (A), Painel do professor (P), Console de curadoria (C) |
