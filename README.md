# LibrAI

Prática guiada de Libras: um agente conduz a sessão e usa reconhecimento de sinais como ferramenta
para decidir o que vem depois.

> **Status:** repositório recém-criado. Nenhuma linha de aplicação escrita. Este README existe para
> receber os artefatos que alimentam o `/paul:init` de cada módulo.
>
> **Disciplina:** DINF00071 — Desenvolvimento Dirigido por Especificação com Agentes · CI/UFPB · 2026.2

---

## O problema

Quem começa a aprender Libras trava no mesmo ponto: consegue estudar o vocabulário sozinho, mas não
tem como saber se o sinal que está fazendo está certo. Vídeo e aplicativo mostram o sinal correto e
não olham de volta. Sem devolutiva, o estudante repete o erro até ele virar hábito — e a correção,
quando aparece, chega tarde e cara.

Quem tem acesso a um intérprete ou a um professor resolve isso. A maioria não tem.

## O que o sistema faz

Apresenta uma tarefa de sinalização, captura a tentativa pela câmera, usa um modelo de
reconhecimento para avaliar o que foi feito, e **decide o próximo passo** a partir do resultado:
repetir com dica, repetir sem dica, ou avançar. O agente conduz a sessão; o modelo é ferramenta
dentro dela, não o produto.

## Quem usa

| Papel | O que precisa |
|---|---|
| Estudante de Libras | Praticar sozinho e saber, na hora, se acertou — sem depender de alguém disponível |
| Professor ou intérprete | Ver onde a turma trava e acompanhar progresso sem corrigir vídeo a vídeo |

## Fora de escopo (preliminar — o grupo fecha e justifica)

- Tradução automática de Libras para português: problema de pesquisa, não cabe no prazo.
- Treinar um modelo novo de reconhecimento: o sistema **usa** um modelo, não o produz.
- Sinais de contexto e expressão facial gramatical: o MVP fica no vocabulário isolado.
- Certificação ou avaliação formal de proficiência.

## Módulos

Um repositório, quatro módulos independentes. Cada módulo tem persistência, API, componente
concorrente, desempenho medido e testes — os cinco, sozinho.

| # | Módulo | Responsável | O que faz |
|---|---|---|---|
| 1 | `captura` | *a definir* | Recebe a tentativa do estudante, valida, enfileira e persiste o registro da tentativa |
| 2 | `tutor` | *a definir* | Agente que lê o resultado e decide repetir, dar dica ou avançar; mantém o estado da sessão |
| 3 | `catalogo` | *a definir* | Vocabulário, trilhas e histórico de tentativas; API de consulta |
| 4 | `progresso` | *a definir* | Progresso do estudante, painel do professor e relatórios |

**Contrato principal:** o resultado da tentativa — identificador do sinal esperado, o que o modelo
reconheceu e a confiança. O tutor decide a partir disso e nunca a partir do vídeo.

**Limiar de confiança é decisão de produto, não detalhe técnico:** abaixo de quanto o sistema diz
"não consegui ver" em vez de dizer "errado"? Dizer "errado" para quem acertou é o pior erro
possível aqui.

## Dados e privacidade

O sistema captura **imagem de pessoa**, o que é dado pessoal sensível — potencialmente biométrico.
Decisões preliminares:

- O vídeo da tentativa não é retido por padrão: o que persiste é o resultado, não a imagem.
- Se a retenção for necessária para o professor revisar, ela é opcional, consentida e com prazo.
- Imagens nunca em bucket público; acesso por papel e com URL de validade curta.
- O painel do professor mostra agregado da turma; tentativa individual só com vínculo explícito.

**Dados:** gerados pelos usuários. O modelo de reconhecimento entra como dependência externa — e é
o maior risco técnico do projeto.

## Estrutura do repositório

```
librai/
├── README.md              # este arquivo — brief do projeto
├── CONTRATOS.md           # FI-02: módulos, contratos e duplos
├── modulos/
│   ├── captura/    .paul/
│   ├── tutor/      .paul/
│   ├── catalogo/   .paul/
│   └── progresso/  .paul/
└── .gitignore             # .claude/ fora do versionamento
```

## Artefatos de entrada do `/paul:init`

O init de cada módulo só roda depois que isto estiver no repositório:

- [ ] **FI-01 de cada integrante**, com as seções 7 e 8 escritas no nível do módulo
- [ ] **CONTRATOS.md** do grupo: os quatro módulos, os contratos entre pares que se tocam e o duplo de cada contrato
- [ ] **Teste de independência** respondido por cada integrante
- [ ] **Telas do sistema** (Trabalho 1): no mínimo dez, como mock, descrição detalhada ou desenho
- [ ] **Artefatos de arquitetura** (Trabalho 1): padrão de especificação escolhido, justificativa e os artefatos produzidos

## Como o trabalho é conduzido

Cada módulo roda o ciclo **Plan → Audit → Apply → Unify**: o plano declara critérios de aceitação
verificáveis, a auditoria revisa antes de qualquer código, a execução segue o plano aprovado e o
fechamento compara planejado com entregue.

A **Fase 1 (Fundação)** é a mesma para todos os projetos da turma: scaffold com CI que falha quando
deve, banco com as restrições de dados no schema, e deploy que só publica com o CI verde. A
diferença entre os projetos começa na Fase 2.

## Licença

MIT.
