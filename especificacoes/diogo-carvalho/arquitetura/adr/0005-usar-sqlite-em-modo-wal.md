# ADR-0005 · Usar SQLite em modo WAL como único banco, com transações `BEGIN IMMEDIATE`

- **Status:** aceita
- **Data:** 27/10/2026
- **Contexto:** O FI-01 escolheu SQLite porque o MVP só armazena sinais, sessões e tentativas, sem banco externo para operar (§6). A especificação completa acrescenta contas, consentimentos, turmas e reportes, ainda num volume pequeno (uma turma de dezenas de alunos, poucas gravações por segundo). As garantias de REQ-30 (atomicidade) e ADR-0003 (versão otimista) exigem transações confiáveis com leituras concorrentes.
- **Opções consideradas:**
  1. **SQLite no modo padrão (rollback journal)** — leitores bloqueiam durante a escrita.
  2. **SQLite em modo WAL** com `busy_timeout=5000` e escrita em `BEGIN IMMEDIATE`.
  3. **PostgreSQL** em contêiner.
- **Decisão:** Opção 2. `PRAGMA journal_mode=WAL; PRAGMA foreign_keys=ON; PRAGMA busy_timeout=5000`. Toda operação que grava abre `BEGIN IMMEDIATE` (pega o lock de escrita no início, evitando `SQLITE_BUSY` no meio da transação). Leitura do painel do professor não bloqueia a gravação de tentativas. O acesso passa por uma camada de repositório, para que trocar para PostgreSQL afete um só módulo.
- **Consequências:**
  - \+ Zero operação de infraestrutura; o banco é um arquivo no volume `librai-data`; backup = copiar o arquivo com `sqlite3 .backup`.
  - \+ Índices únicos parciais (`WHERE status = 'active'`) estão disponíveis — usados em REQ-15, REQ-35 e no consentimento vigente.
  - \+ Leituras concorrentes com uma escrita, suficientes para a carga de QS-04.
  - − Um escritor por vez: vira gargalo se houver centenas de tentativas por segundo — muito acima da carga prevista. O sinal para migrar a PostgreSQL é o p95 de espera por lock passar de 100 ms em QS-04.
  - − Um único host (sem réplica); aceitável para disciplina e piloto.
- **Requisitos afetados:** REQ-15, REQ-25, REQ-30, REQ-43.
