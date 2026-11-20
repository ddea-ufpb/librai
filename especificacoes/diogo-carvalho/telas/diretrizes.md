# LibrAI · Diretrizes globais da interface

> Camada 1 da especificação de telas. Vale para **todas** as fichas em `telas/`; nenhuma ficha repete o que está aqui.
> Termos em `monoespaçado` são nomes do modelo de dados e dos contratos em [`../arquitetura/contratos/`](../arquitetura/contratos/).

## 1. Público

| Ator | Quem é | Em que situação usa |
|---|---|---|
| **Aluno** (`role = student`) | Pessoa ouvinte ou surda aprendendo Libras, do iniciante ao intermediário, em geral estudante universitário ou de curso livre | Prática individual curta (5 a 15 min), em casa, na universidade ou no transporte, **sem professor por perto**; quer saber se executou o sinal corretamente |
| **Professor / instrutor** (`role = teacher`) | Docente ou intérprete que acompanha uma ou mais turmas | Entre aulas, no computador ou tablet, para ver quem praticou e quais sinais a turma mais erra |
| **Administrador / curador** (`role = admin`) | Pessoa da equipe com fluência em Libras que mantém o catálogo de sinais | No computador, cadastrando sinais, vídeos de referência e licenças, e tratando reportes de conteúdo |

Parte afetada que **não opera** o sistema, mas orienta a interface: a **comunidade surda**. A interface nunca apresenta uma variante regional como "a forma correta" (sempre mostra `regional_variant`) e sempre credita quem gravou a referência (`source_credit`).

## 2. Plataforma e orientação

| Superfície | Plataforma | Orientação | Observação |
|---|---|---|---|
| **Acesso** (G) | Web responsiva | Retrato no celular, paisagem no desktop | Comum aos três atores |
| **App do aluno** (A) | Web responsiva, **celular primeiro**, também desktop | Celular em **retrato**; desktop em paisagem | Precisa de câmera frontal ou webcam; funciona em Chrome, Edge, Firefox e Safari atuais |
| **Painel do professor** (P) | Web, desktop e tablet | Paisagem; largura mínima útil de 768 px | Sem uso de câmera |
| **Console de curadoria** (C) | Web, desktop | Paisagem; largura mínima útil de 1024 px | Upload de vídeo de referência |

As três superfícies são rotas do mesmo aplicativo web (ver [ADR-0007](../arquitetura/adr/0007-servir-tres-superficies-num-unico-app-web.md)); o papel do usuário decide qual delas abre depois de entrar.

## 3. Tom de voz e de interface

- **Voz de tutor paciente, não de juiz.** Errar faz parte da prática: "Ainda não foi desta vez — a mão ficou fora do quadro" em vez de "Errado".
- **Direto e em segunda pessoa** ("Faça o sinal da letra B"). Frases curtas, uma instrução por vez.
- **Sem gamificação punitiva:** nada de vidas, contagem regressiva de ofensiva ou perda de pontos.
- **Explica o porquê das decisões do agente** em linguagem simples, sem jargão ("confiança", "limiar" aparecem só no detalhe expansível).
- **Interface calma:** fundo claro, poucos elementos, o vídeo de referência e a câmera são os protagonistas na prática.
- Nos painéis do professor e do curador o tom é **neutro e informativo**, com números sempre acompanhados do período a que se referem.

## 4. Princípios verificáveis

Cada princípio pode ser checado olhando uma tela gerada; todos apontam um requisito em [`requisitos.md`](../arquitetura/requisitos.md).

| # | Princípio | Como verificar | Requisito |
|---|---|---|---|
| P1 | **Uma intenção por tela.** Cada tela tem exatamente **uma** ação primária visualmente destacada. | Contar botões primários: deve haver um. | — (diretiva 2) |
| P2 | **Nunca mentir sobre o resultado.** Acerto, erro ou decisão do agente só aparecem **depois** da resposta da API; não existe "correto" otimista. | Com a API desligada, nenhuma tela mostra resultado de tentativa. | REQ-55 |
| P3 | **A câmera nunca é surpresa.** Antes de pedir a câmera ao navegador, a interface explica o que é processado e que **nenhuma imagem sai do aparelho nem é guardada**; enquanto a câmera estiver ligada, há indicador visível. | Primeiro acesso: o pedido do navegador só aparece depois da tela A02. | REQ-08, REQ-09, REQ-11 |
| P4 | **Resultado não depende só de cor.** Todo resultado tem texto + ícone + cor; nenhuma informação é transmitida apenas por som (o sistema não usa áudio). | Ver a tela em escala de cinza: o resultado continua legível. | REQ-52 |
| P5 | **O agente sempre explica a decisão.** Toda decisão `REPEAT`/`ADVANCE` vem com o motivo (`decision_reason`) em linguagem simples. | Toda tela A04 mostra uma frase de motivo. | REQ-20 |

## 5. Referência visual

**O que inspira**
- Fluxos de aprendizagem de uma tarefa por vez (ex.: lição de aplicativo de idiomas), com o conteúdo ocupando a maior parte da tela.
- Padrão de serviços públicos digitais (ex.: Design System do gov.br): contraste alto, tipografia grande, linguagem simples.
- Painéis analíticos sóbrios para professor e curador: tabelas legíveis, filtros explícitos, números com período.

**O que evitar**
- Emojis ou desenhos estáticos de mãos **no lugar** do vídeo do sinal — não representam Libras (movimento, orientação e expressão facial ficam de fora).
- Mascote infantilizado, confete, sons de acerto/erro, vermelho alarmante para tentativa errada.
- Autoplay de vídeo em loop sem controle de pausa; animações sem respeitar `prefers-reduced-motion`.
- Painéis com ranking de alunos (expõe quem erra mais) — o professor vê agregados e o aluno individual, nunca um "placar".

## 6. Idioma e acessibilidade

- **Idioma:** português do Brasil (pt-BR) em toda a interface, datas no formato `dd/mm/aaaa`, números com vírgula decimal. O nome do sinal (`gloss`) aparece em MAIÚSCULAS, como é convenção na escrita de glosas de Libras.
- **Piso de acessibilidade: WCAG 2.2 nível AA**, com atenção especial a:
  - **1.4.3 / 1.4.11** — contraste mínimo 4,5:1 em texto e 3:1 em componentes e no contorno do quadro da câmera.
  - **1.2.x** — todo vídeo de referência tem descrição textual (`description_text`); não há áudio.
  - **2.2.2** — vídeo de referência com pausar, repetir e velocidade 0,5× (REQ-52).
  - **2.4.7 / 2.4.11** — foco sempre visível e não encoberto pelo quadro da câmera.
  - **2.5.7** — nenhuma ação depende de arrastar.
  - **2.5.8** — alvos de toque com no mínimo 24 × 24 px (o projeto adota **44 × 44 px** no app do aluno).
  - **3.3.7 / 3.3.8** — sem pedir de novo dado já informado; login aceita colar senha e gerenciador de senhas.
  - **4.1.3** — mudanças de estado (resultado da tentativa, erro, offline) anunciadas por região `aria-live`.
- **Público surdo:** nenhuma informação apenas sonora; mensagens curtas e em linguagem simples, porque o português pode ser a segunda língua de quem usa Libras.
- **Movimento:** respeitar `prefers-reduced-motion`; o vídeo de referência nunca inicia em loop automático sem controle.

## 7. Estados padrão (vale para todas as fichas)

Toda ficha especifica os seis estados. Quando uma ficha não detalha um estado, vale o padrão abaixo.

| Estado | Comportamento padrão | Requisito |
|---|---|---|
| **Carregando** | Esqueleto da área de conteúdo depois de 300 ms; nunca exibe dado antigo como se fosse atual | REQ-54 |
| **Vazio** | Frase que explica por que está vazio + a ação que resolve (quando houver) | REQ-60 |
| **Erro** | Mensagem em linguagem simples, botão "Tentar de novo", dados digitados preservados | REQ-57 |
| **Sucesso** | Confirmação curta anunciada por `aria-live`; nada de modal bloqueante | REQ-61 |
| **Offline** | Faixa fixa "Sem conexão"; ações que gravam ficam desabilitadas; nada é enfileirado | REQ-51 |
| **Sem permissão** | Papel errado → "Esta área é de outro tipo de conta" + link para a área certa; sessão expirada → volta para G01 guardando o destino | REQ-07, REQ-56 |
