# LibrAI · Matriz de rastreabilidade

> **Gerado** por `scripts/verificar_rastreabilidade.py --write` a partir das fichas de tela, de
> [`requisitos.md`](requisitos.md) e de [`contratos/openapi.yaml`](contratos/openapi.yaml) (`x-screens`,
> `x-requirements`). Não editar à mão: rode o script. O script também falha se houver id citado e não
> definido, operação sem tela, tela sem operação ou divergência entre `x-screens` e as fichas.

## 1. Tela → contrato → requisitos

| Tela | Nome | Operações (`operationId`) | Requisitos citados na ficha |
|---|---|---|---|
| [G01](../telas/G01-entrar.md) | Entrar | `get_me`, `login` | REQ-01, REQ-02, REQ-03, REQ-04, REQ-07, REQ-51, REQ-54, REQ-56, REQ-57, REQ-60, REQ-61 |
| [G02](../telas/G02-criar-conta.md) | Criar conta | `create_account` | REQ-04, REQ-05, REQ-06, REQ-07, REQ-08, REQ-51, REQ-54, REQ-57, REQ-60, REQ-61 |
| [A01](../telas/A01-inicio.md) | Início | `get_home`, `logout`, `start_session` | REQ-07, REQ-08, REQ-13, REQ-14, REQ-15, REQ-29, REQ-32, REQ-51, REQ-54, REQ-56, REQ-57 |
| [A02](../telas/A02-preparar-camera.md) | Preparar câmera | `grant_consent`, `revoke_consent` | REQ-07, REQ-08, REQ-09, REQ-11, REQ-12, REQ-28, REQ-51, REQ-54, REQ-57, REQ-60, REQ-61 |
| [A03](../telas/A03-praticar-sinal.md) | Praticar sinal | `abandon_session`, `get_active_session`, `submit_attempt` | REQ-07, REQ-08, REQ-09, REQ-10, REQ-16, REQ-24, REQ-25, REQ-26, REQ-27, REQ-28, REQ-29, REQ-34, REQ-50, REQ-51, REQ-52, REQ-54, REQ-55, REQ-57, REQ-58 |
| [A04](../telas/A04-resultado-tentativa.md) | Resultado da tentativa | `get_active_session`, `skip_sign`, `submit_attempt` | REQ-07, REQ-17, REQ-18, REQ-19, REQ-20, REQ-21, REQ-22, REQ-23, REQ-24, REQ-26, REQ-29, REQ-51, REQ-52, REQ-53, REQ-54, REQ-55, REQ-56, REQ-57, REQ-61 |
| [A05](../telas/A05-resumo-sessao.md) | Resumo da sessão | `get_session_summary`, `start_session` | REQ-07, REQ-13, REQ-14, REQ-22, REQ-31, REQ-51, REQ-53, REQ-54, REQ-56, REQ-57, REQ-58, REQ-61 |
| [A06](../telas/A06-meu-progresso.md) | Meu progresso | `get_my_progress` | REQ-07, REQ-32, REQ-51, REQ-54, REQ-56, REQ-57, REQ-60, REQ-61 |
| [A07](../telas/A07-estudar-sinal.md) | Estudar sinal | `get_sign`, `report_sign` | REQ-07, REQ-33, REQ-34, REQ-35, REQ-41, REQ-51, REQ-52, REQ-54, REQ-56, REQ-57, REQ-61 |
| [A08](../telas/A08-entrar-turma.md) | Entrar em turma | `join_class` | REQ-07, REQ-36, REQ-39, REQ-51, REQ-54, REQ-56, REQ-57, REQ-60 |
| [P01](../telas/P01-minhas-turmas.md) | Minhas turmas | `create_class`, `list_classes`, `logout` | REQ-07, REQ-37, REQ-38, REQ-51, REQ-54, REQ-56, REQ-57, REQ-60 |
| [P02](../telas/P02-alunos-turma.md) | Alunos da turma | `list_class_students` | REQ-07, REQ-37, REQ-39, REQ-51, REQ-54, REQ-56, REQ-57, REQ-60, REQ-61 |
| [P03](../telas/P03-desempenho-aluno.md) | Desempenho do aluno | `get_student_performance` | REQ-07, REQ-11, REQ-37, REQ-39, REQ-51, REQ-54, REQ-56, REQ-57, REQ-60, REQ-61 |
| [P04](../telas/P04-dificuldades-turma.md) | Dificuldades da turma | `list_class_difficulties` | REQ-07, REQ-37, REQ-40, REQ-51, REQ-54, REQ-56, REQ-57, REQ-60 |
| [C01](../telas/C01-catalogo-sinais.md) | Catálogo de sinais | `admin_list_signs`, `logout` | REQ-07, REQ-14, REQ-42, REQ-45, REQ-51, REQ-54, REQ-56, REQ-57, REQ-60, REQ-61 |
| [C02](../telas/C02-editar-sinal.md) | Editar sinal | `admin_change_sign_status`, `admin_create_sign`, `admin_get_sign`, `admin_list_recognizer_labels`, `admin_update_sign`, `admin_upload_reference_media` | REQ-07, REQ-41, REQ-42, REQ-43, REQ-44, REQ-45, REQ-48, REQ-51, REQ-54, REQ-55, REQ-56, REQ-57, REQ-60 |
| [C03](../telas/C03-reportes-conteudo.md) | Reportes de conteúdo | `admin_list_reports`, `admin_resolve_report` | REQ-07, REQ-35, REQ-46, REQ-51, REQ-54, REQ-56, REQ-57, REQ-60, REQ-61 |
| [C04](../telas/C04-contas-papeis.md) | Contas e papéis | `admin_create_user`, `admin_list_users`, `admin_update_user` | REQ-04, REQ-06, REQ-07, REQ-47, REQ-51, REQ-54, REQ-56, REQ-57, REQ-59, REQ-60, REQ-61 |

## 2. Requisito → telas → contrato

| Requisito | Molde | Telas | Operações |
|---|---|---|---|
| REQ-01 | U | G01 | `get_me`, `login`, `logout` |
| REQ-02 | I | G01 | `login` |
| REQ-03 | I | G01 | `login` |
| REQ-04 | E | G01, G02, C04 | `create_account` |
| REQ-05 | I | G02 | `create_account` |
| REQ-06 | U | G02, C04 | `admin_create_user`, `create_account` |
| REQ-07 | I | G01, G02, A01, A02, A03, A04, A05, A06, A07, A08, P01, P02, P03, P04, C01, C02, C03, C04 | `admin_get_sign`, `admin_list_signs`, `admin_list_users`, `list_classes` |
| REQ-08 | S | G02, A01, A02, A03 | `get_home`, `grant_consent`, `submit_attempt` |
| REQ-09 | U | A02, A03 | `submit_attempt` |
| REQ-10 | I | A03 | `submit_attempt` |
| REQ-11 | U | A02, P03 | `get_student_performance`, `submit_attempt` |
| REQ-12 | E | A02 | `revoke_consent` |
| REQ-13 | E | A01, A05 | `start_session` |
| REQ-14 | I | A01, A05, C01 | `admin_list_signs`, `get_home`, `start_session` |
| REQ-15 | S | A01 | `start_session` |
| REQ-16 | E | A03 | `submit_attempt` |
| REQ-17 | E | A04 | `submit_attempt` |
| REQ-18 | E | A04 | `submit_attempt` |
| REQ-19 | I | A04 | `submit_attempt` |
| REQ-20 | U | A04 | `submit_attempt` |
| REQ-21 | I | A04 | `submit_attempt` |
| REQ-22 | E | A04, A05 | `skip_sign`, `submit_attempt` |
| REQ-23 | E | A04 | `skip_sign` |
| REQ-24 | I | A03, A04 | `skip_sign`, `submit_attempt` |
| REQ-25 | I | A03 | `submit_attempt` |
| REQ-26 | I | A03, A04 | `skip_sign`, `submit_attempt` |
| REQ-27 | I | A03 | `submit_attempt` |
| REQ-28 | I | A02, A03 | `submit_attempt` |
| REQ-29 | E | A01, A03, A04 | `get_active_session`, `get_home` |
| REQ-30 | U | — | `skip_sign`, `submit_attempt` |
| REQ-31 | E | A05 | `get_session_summary` |
| REQ-32 | O | A01, A06 | `get_home`, `get_my_progress` |
| REQ-33 | U | A07 | `get_sign` |
| REQ-34 | U | A03, A07 | `get_active_session`, `get_sign` |
| REQ-35 | E | A07, C03 | `admin_list_reports`, `report_sign` |
| REQ-36 | E | A08 | `join_class` |
| REQ-37 | I | P01, P02, P03, P04 | `get_student_performance`, `list_class_difficulties`, `list_class_students`, `list_classes` |
| REQ-38 | E | P01 | `create_class` |
| REQ-39 | U | A08, P02, P03 | `get_student_performance`, `list_class_students` |
| REQ-40 | E | P04 | `list_class_difficulties` |
| REQ-41 | I | A07, C02 | `admin_change_sign_status`, `admin_upload_reference_media` |
| REQ-42 | E | C01, C02 | `admin_change_sign_status`, `admin_list_signs` |
| REQ-43 | I | C02 | `admin_change_sign_status`, `admin_update_sign` |
| REQ-44 | I | C02 | `admin_upload_reference_media` |
| REQ-45 | E | C01, C02 | `admin_change_sign_status`, `admin_list_signs`, `start_session` |
| REQ-46 | I | C03 | `admin_resolve_report` |
| REQ-47 | I | C04 | `admin_update_user` |
| REQ-48 | I | C02 | `admin_create_sign`, `admin_list_recognizer_labels`, `admin_update_sign` |
| REQ-49 | U | — | `submit_attempt` |
| REQ-50 | U | A03 | `submit_attempt` |
| REQ-51 | S | G01, G02, A01, A02, A03, A04, A05, A06, A07, A08, P01, P02, P03, P04, C01, C02, C03, C04 | — (cliente/operação) |
| REQ-52 | U | A03, A04, A07 | `get_active_session`, `get_sign` |
| REQ-53 | E | A04, A05 | `submit_attempt` |
| REQ-54 | U | G01, G02, A01, A02, A03, A04, A05, A06, A07, A08, P01, P02, P03, P04, C01, C02, C03, C04 | — (cliente/operação) |
| REQ-55 | U | A03, A04, C02 | `admin_create_sign`, `admin_update_sign`, `submit_attempt` |
| REQ-56 | E | G01, A01, A04, A05, A06, A07, A08, P01, P02, P03, P04, C01, C02, C03, C04 | `get_me` |
| REQ-57 | I | G01, G02, A01, A02, A03, A04, A05, A06, A07, A08, P01, P02, P03, P04, C01, C02, C03, C04 | — (cliente/operação) |
| REQ-58 | E | A03, A05 | `abandon_session` |
| REQ-59 | E | C04 | `admin_create_user` |
| REQ-60 | U | G01, G02, A02, A06, A08, P01, P02, P03, P04, C01, C02, C03, C04 | — (cliente/operação) |
| REQ-61 | U | G01, G02, A02, A04, A05, A06, A07, P02, P03, C01, C03, C04 | — (cliente/operação) |

## 3. Operação → telas

| `operationId` | Rota | Telas (`x-screens`) |
|---|---|---|
| `login` | `POST /auth/login` | G01 |
| `logout` | `POST /auth/logout` | A01, C01, P01 |
| `get_me` | `GET /me` | G01 |
| `create_account` | `POST /accounts` | G02 |
| `get_home` | `GET /me/home` | A01 |
| `grant_consent` | `POST /me/consents` | A02 |
| `revoke_consent` | `DELETE /me/consents/camera_processing` | A02 |
| `start_session` | `POST /practice-sessions` | A01, A05 |
| `get_active_session` | `GET /practice-sessions/active` | A03, A04 |
| `submit_attempt` | `POST /practice-sessions/{session_id}/attempts` | A03, A04 |
| `skip_sign` | `POST /practice-sessions/{session_id}/skip` | A04 |
| `abandon_session` | `POST /practice-sessions/{session_id}/abandon` | A03 |
| `get_session_summary` | `GET /practice-sessions/{session_id}/summary` | A05 |
| `get_my_progress` | `GET /me/progress` | A06 |
| `get_sign` | `GET /signs/{sign_id}` | A07 |
| `report_sign` | `POST /signs/{sign_id}/reports` | A07 |
| `join_class` | `POST /class-enrollments` | A08 |
| `list_classes` | `GET /classes` | P01 |
| `create_class` | `POST /classes` | P01 |
| `list_class_students` | `GET /classes/{class_id}/students` | P02 |
| `get_student_performance` | `GET /classes/{class_id}/students/{student_id}/performance` | P03 |
| `list_class_difficulties` | `GET /classes/{class_id}/difficulties` | P04 |
| `admin_list_signs` | `GET /admin/signs` | C01 |
| `admin_create_sign` | `POST /admin/signs` | C02 |
| `admin_get_sign` | `GET /admin/signs/{sign_id}` | C02 |
| `admin_update_sign` | `PUT /admin/signs/{sign_id}` | C02 |
| `admin_upload_reference_media` | `POST /admin/signs/{sign_id}/reference-media` | C02 |
| `admin_change_sign_status` | `POST /admin/signs/{sign_id}/status` | C02 |
| `admin_list_recognizer_labels` | `GET /admin/recognizer/labels` | C02 |
| `admin_list_reports` | `GET /admin/reports` | C03 |
| `admin_resolve_report` | `PATCH /admin/reports/{report_id}` | C03 |
| `admin_list_users` | `GET /admin/users` | C04 |
| `admin_create_user` | `POST /admin/users` | C04 |
| `admin_update_user` | `PATCH /admin/users/{user_id}` | C04 |
