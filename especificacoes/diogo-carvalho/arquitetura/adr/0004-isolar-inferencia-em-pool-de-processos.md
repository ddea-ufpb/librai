# ADR-0004 · Executar a inferência num pool de processos de tamanho fixo com fila limitada

- **Status:** aceita
- **Data:** 27/10/2026
- **Contexto:** Várias sessões de alunos enviam tentativas ao mesmo tempo (ex.: uma turma praticando durante a aula). A inferência é trabalho de CPU (não há GPU, FI-01 §10.3). Rodá-la no laço de eventos do FastAPI travaria todas as requisições; rodar em threads esbarra no GIL do Python; aceitar trabalho sem limite faz a latência crescer sem teto e todos os alunos esperam. O **recurso compartilhado** são os núcleos de CPU e o modelo carregado em memória.
- **Opções consideradas:**
  1. Inferência síncrona no próprio handler.
  2. `ThreadPoolExecutor`.
  3. **`ProcessPoolExecutor` com N workers fixos, cada um com o modelo carregado uma vez, e um semáforo limitando trabalhos pendentes a 2×N; acima disso, 503 imediato com `Retry-After`.**
  4. Serviço de inferência separado com fila (Celery/Redis) — descartado pelo FI-01 §6.
- **Decisão:** Opção 3. `RECOGNIZER_WORKERS` (padrão: núcleos − 1, mínimo 1). A API adquire o semáforo com `try_acquire` (sem esperar); se não conseguir, responde `recognizer_unavailable` (REQ-27). Cada chamada tem `asyncio.wait_for(..., 5 s)`. O modelo é carregado no inicializador do worker, não por requisição.
- **Consequências:**
  - \+ Latência limitada: com fila de 2×N, ninguém espera mais que cerca de 2 inferências na frente (base da meta de QS-04).
  - \+ Degradação explícita (503 com tempo para tentar de novo) em vez de lentidão generalizada — "nunca mentir sobre o envio".
  - \+ Isola falhas: um worker que trava não derruba a API.
  - − Memória × N (uma cópia do modelo por worker); o N precisa ser calibrado na máquina de implantação.
  - − Serializar `landmarks` entre processos custa alguns milissegundos (cerca de 100 KB por tentativa).
- **Requisitos afetados:** REQ-16, REQ-27, REQ-50.
