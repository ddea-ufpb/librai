# LibrAI · §1 Introdução e metas · §2 Restrições

> Seções 1 e 2 do arc42. Fonte: FI-01 (Formulário de Ideação, 23/09/2026). O que **não** estava no FI-01 e foi decidido nesta especificação está marcado com **[novo]**, para a auditoria do `PROJECT.md`.

## §1.1 O que é

Sistema de ensino de Libras em que um agente tutor conduz sessões de prática: apresenta um sinal, analisa a execução do aluno capturada pela câmera (com um modelo de reconhecimento usado como ferramenta) e decide entre repetir e avançar, acompanhando o progresso do aluno.

## §1.2 Problema

Quem aprende Libras sozinho não sabe se está executando os sinais corretamente: vídeos e materiais estáticos mostram o sinal, mas não observam o aluno. Sem correção, os erros se repetem e o aprendizado fica dependente de alguém que domine Libras.

**Aposta central (FI-01 §4.4):** se um agente conseguir conduzir uma prática curta de Libras, usando o resultado do reconhecimento para decidir quando repetir ou avançar, a prática individual fica menos dependente de acompanhamento humano constante.

## §1.3 Metas de qualidade (ordem de prioridade)

| # | Meta | Por que nesta ordem | Cenário |
|---|---|---|---|
| 1 | **Privacidade da imagem do aluno** — nenhuma imagem sai do aparelho nem é guardada | Imagem de rosto e corpo é o dado mais sensível do sistema; o FI-01 fixou "0 ocorrências" de persistência indevida | [QS-01](qualidade.md#qs-01--persistência) |
| 2 | **Decisão correta e previsível do agente** — repetir e avançar seguem uma regra testável | É a aposta central; o FI-01 pede 100% dos cenários de teste do fluxo | [QS-05](qualidade.md#qs-05--testes) |
| 3 | **Consistência sob concorrência** — reenvio ou duas abas nunca duplicam tentativa nem pulam sinal | Rede móvel instável é o contexto de uso do aluno | [QS-03](qualidade.md#qs-03--concorrência) |
| 4 | **Resposta rápida o bastante para manter o ritmo da prática** | Sem retorno rápido, o aluno perde o fio da tentativa | [QS-04](qualidade.md#qs-04--desempenho-medido) |
| 5 | **Contrato de API estável** — as telas e o backend evoluem em paralelo | O grupo vai dividir o sistema em módulos | [QS-02](qualidade.md#qs-02--api-com-contrato) |

## §1.4 Partes interessadas

| Parte | Expectativa | Onde aparece |
|---|---|---|
| Aluno de Libras | Praticar sozinho e saber se acertou | A01–A08 |
| Professor / instrutor | Ver quem pratica e o que a turma mais erra | P01–P04 |
| Administrador / curador | Catálogo correto, licenciado e revisado | C01–C04 |
| Comunidade surda (não usuária) | Não ver sinais ensinados errado nem variante regional tratada como a única forma | `regional_variant` obrigatório (REQ-34), reportes (REQ-35), revisão antes de publicar (REQ-42) |
| Pessoas filmadas nas referências (não usuárias) | Uso da imagem só com autorização | `consent_confirmed` e `license` obrigatórios para publicar (REQ-41) |
| Professores e produtores de material (não usuários) | Crédito e licença respeitados | `source_credit`, `author_credit`, `license` (REQ-34, REQ-41) |
| Professor da disciplina / PAUL | Especificação rastreável, que vire `PROJECT.md` e planos | este diretório |

## §2 Restrições

### §2.1 Técnicas (stack do FI-01 §6)

| Restrição | Valor | Origem |
|---|---|---|
| Linguagem do backend e do agente | Python 3.12 | FI-01 §6 |
| API | FastAPI | FI-01 §6 |
| Orquestração do agente | LangGraph — **um único** agente tutor | FI-01 §6, §10.2 |
| Persistência | SQLite (modo WAL **[novo]**, ADR-0005) | FI-01 §6 |
| Interface | Next.js / React, uma aplicação web | FI-01 §6 |
| Extração de pontos das mãos | MediaPipe Hands (Tasks Vision) no navegador **[novo]**, ADR-0002 | decorre de "não armazenar imagens" (FI-01 §10.2) |
| Modelo de reconhecimento | Reaproveitar o classificador do LibrAI original: Random Forest (scikit-learn) sobre 21 landmarks de uma mão, letras **estáticas** do alfabeto manual | FI-01 §10.3 ("aproveitar componentes já existentes do LibrAI"); repositório github.com/diogo19025/LibrAI |
| Filas e brokers | **Nenhum** (sem Redis, Celery ou broker) | FI-01 §6 |
| Testes | pytest (backend); Playwright opcional para as telas | FI-01 §6 |
| Medição | logging estruturado + medição de latência em Python | FI-01 §6 |
| Empacotamento | Docker / Docker Compose | FI-01 §6 |
| GPU | **Não** disponível; inferência em CPU | FI-01 §10.3 |

### §2.2 Organizacionais

- Disciplina de **60 h**, equipe pequena; MVP fechável no semestre (FI-01 §10.3).
- Domínio **baixo-médio** em LangGraph: a complexidade do agente é limitada de propósito a duas decisões (FI-01 §6.2, ADR-0001).
- Conta Claude Pro; modelo local a partir da Semana 8 → o provedor de LLM é configurável e opcional (`LIBRAI_LLM_PROVIDER`).
- Reaproveitar componentes já existentes do LibrAI (modelo de reconhecimento).

### §2.3 Legais e éticas (LGPD)

| Restrição | Consequência técnica |
|---|---|
| Imagem de rosto e corpo pode identificar a pessoa e revelar dados de saúde (ex.: deficiência auditiva) — dado sensível, LGPD art. 5º, II e art. 11 | Quadros da câmera nunca saem do navegador; só `landmarks` numéricos são enviados e descartados após a inferência (REQ-09, REQ-11, ADR-0002) |
| Consentimento específico, informado e revogável (art. 8º) | `consent` versionado com `revoked_at`; tela A02 antes do pedido de câmera; revogação encerra a sessão (REQ-08, REQ-12) |
| Minimização (art. 6º, III) | Professor vê só agregados; taxas por turma só com 3 ou mais alunos (REQ-39, REQ-40); o LLM não recebe dado pessoal (`ferramentas-agente.md` §4) |
| Uso de imagem de terceiros nas referências | Publicação exige `license` + `consent_confirmed` (REQ-41) |

### §2.4 Fora de escopo (FI-01 §5)

Reconhecimento de Libras contínua ou tradução de frases · avaliação separada de parâmetros linguísticos (configuração de mão, ponto de articulação, movimento, expressão facial) · múltiplos agentes · trilhas personalizadas por histórico de longo prazo · dashboard institucional. As telas de professor (P01–P04) são especificadas para o grupo, mas ficam **fora do MVP**, como o FI-01 declara.
