# LibrAI · Contratos

| Arquivo | Conteúdo | Padrão |
|---|---|---|
| [`openapi.yaml`](openapi.yaml) | As 34 operações REST usadas pelas telas, com `x-screens` e `x-requirements` em cada uma | OpenAPI 3.1 (validado com `openapi-spec-validator`) |
| [`modelo-dados.md`](modelo-dados.md) | Diagrama ER, DDL SQLite, decisões de modelagem e retenção | Mermaid + SQL |
| [`ferramentas-agente.md`](ferramentas-agente.md) | Ferramentas internas do agente tutor (`recognize_sign`, `decide_next`, `generate_feedback`, `record_attempt`) | Assinaturas Python tipadas + tabela de decisão |

## Por que não há AsyncAPI

O FI-01 §6 decidiu **"Concorrência / filas: nenhuma no MVP"**: uma tentativa é enviada, processada e respondida na mesma requisição. Não existe canal assíncrono (fila, WebSocket, evento entre serviços) para descrever. A concorrência do sistema está **dentro** da requisição — pool de processos de inferência, idempotência e versão otimista ([ADR-0003](../adr/0003-garantir-idempotencia-e-versao-otimista.md), [ADR-0004](../adr/0004-isolar-inferencia-em-pool-de-processos.md)) — e é coberta pelo OpenAPI (respostas 200 idempotente, 409 e 503). Se o grupo introduzir eventos (ex.: notificar o professor), o contrato entra aqui como `asyncapi.yaml`.

## Operação × tela

| `operationId` | Método e caminho | Telas |
|---|---|---|
| `login` | POST /auth/login | G01 |
| `logout` | POST /auth/logout | A01, P01, C01 |
| `get_me` | GET /me | G01 |
| `create_account` | POST /accounts | G02 |
| `get_home` | GET /me/home | A01 |
| `grant_consent` | POST /me/consents | A02 |
| `revoke_consent` | DELETE /me/consents/camera_processing | A02 |
| `start_session` | POST /practice-sessions | A01, A05 |
| `get_active_session` | GET /practice-sessions/active | A03, A04 |
| `submit_attempt` | POST /practice-sessions/{session_id}/attempts | A03, A04 |
| `skip_sign` | POST /practice-sessions/{session_id}/skip | A04 |
| `abandon_session` | POST /practice-sessions/{session_id}/abandon | A03 |
| `get_session_summary` | GET /practice-sessions/{session_id}/summary | A05 |
| `get_my_progress` | GET /me/progress | A06 |
| `get_sign` | GET /signs/{sign_id} | A07 |
| `report_sign` | POST /signs/{sign_id}/reports | A07 |
| `join_class` | POST /class-enrollments | A08 |
| `list_classes` | GET /classes | P01 |
| `create_class` | POST /classes | P01 |
| `list_class_students` | GET /classes/{class_id}/students | P02 |
| `get_student_performance` | GET /classes/{class_id}/students/{student_id}/performance | P03 |
| `list_class_difficulties` | GET /classes/{class_id}/difficulties | P04 |
| `admin_list_signs` | GET /admin/signs | C01 |
| `admin_create_sign` | POST /admin/signs | C02 |
| `admin_get_sign` | GET /admin/signs/{sign_id} | C02 |
| `admin_update_sign` | PUT /admin/signs/{sign_id} | C02 |
| `admin_upload_reference_media` | POST /admin/signs/{sign_id}/reference-media | C02 |
| `admin_change_sign_status` | POST /admin/signs/{sign_id}/status | C02 |
| `admin_list_recognizer_labels` | GET /admin/recognizer/labels | C02 |
| `admin_list_reports` | GET /admin/reports | C03 |
| `admin_resolve_report` | PATCH /admin/reports/{report_id} | C03 |
| `admin_list_users` | GET /admin/users | C04 |
| `admin_create_user` | POST /admin/users | C04 |
| `admin_update_user` | PATCH /admin/users/{user_id} | C04 |
