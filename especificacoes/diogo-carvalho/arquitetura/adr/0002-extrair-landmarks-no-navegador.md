# ADR-0002 · Extrair os pontos das mãos no navegador e nunca transmitir nem guardar imagens da câmera

- **Status:** aceita
- **Data:** 27/10/2026
- **Contexto:** O FI-01 decidiu "não armazenar imagens ou vídeos das tentativas" (§10.2) e definiu a métrica "persistência indevida de imagem/vídeo da câmera = 0 ocorrências" (§10.1). A imagem da câmera mostra rosto, corpo e ambiente doméstico do aluno e pode revelar dado de saúde (deficiência auditiva) — dado pessoal sensível na LGPD (art. 5º, II). Mesmo sem gravar, **enviar** quadros ao servidor já expõe a imagem em trânsito, em logs, em dumps de memória e a quem opera o servidor. O modelo de reconhecimento só precisa da geometria das mãos, não dos pixels.
- **Opções consideradas:**
  1. **Enviar quadros (JPEG) ao servidor** e descartá-los após a inferência.
  2. **Enviar quadros e guardar por N dias** para melhorar o modelo.
  3. **Extrair `landmarks` (21 pontos × 3 coordenadas por mão) no navegador com MediaPipe Hands e enviar só os números**; o servidor classifica a sequência de pontos.
- **Decisão:** Opção 3. O cliente roda MediaPipe Hands (`@mediapipe/tasks-vision`, modelo servido pelo próprio LibrAI, sem CDN de terceiros) e envia em `submit_attempt` apenas `landmarks` (até 90 quadros). O schema `AttemptRequest` tem `additionalProperties: false` e limite de 512 KB (REQ-10); a tabela `attempt` não tem coluna para pontos nem imagem (REQ-11). Os `landmarks` vivem só na memória da API durante a inferência.
- **Consequências:**
  - \+ A meta "0 imagens persistidas" vira propriedade da arquitetura, não promessa de código: nenhuma imagem existe no servidor para vazar. A mitigação **preserva a função** (o reconhecimento continua) em vez de amputá-la.
  - \+ O corpo da requisição cai de cerca de 3 MB (90 quadros JPEG) para cerca de 100 KB de JSON, o que reduz a latência em rede móvel (QS-04).
  - \+ O professor e o curador nunca têm acesso a imagem de aluno (REQ-39).
  - \+ **Compatível com o modelo que já existe:** o LibrAI original (github.com/diogo19025/LibrAI) já usa MediaPipe Hands e classifica os 21 pontos com um Random Forest. A mudança é só **onde** o MediaPipe roda (no navegador, não no `app.py` com OpenCV); o classificador é reaproveitado sem retreino.
  - − Se um modelo futuro (ex.: sinais dinâmicos) exigir pixels, este ADR precisa ser revisto ([`riscos-e-lacunas.md`](../riscos-e-lacunas.md) R-01); a alternativa seria a opção 1, com descarte em memória.
  - − Sem quadros guardados, não há dados reais para retreinar o modelo; um futuro conjunto de treino exige outro fluxo, com consentimento próprio.
  - − O processamento no aparelho pesa em celulares antigos; a tela A02 detecta falha ao carregar o modelo e informa o aluno.
  - − Pontos das mãos ainda são dado pessoal (comportamental); por isso são descartados após a inferência e nunca entram em log.
- **Requisitos afetados:** REQ-09, REQ-10, REQ-11, REQ-28, REQ-39.
