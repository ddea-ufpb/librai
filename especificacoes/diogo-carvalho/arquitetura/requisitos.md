# LibrAI · Requisitos (EARS)

> Requisitos funcionais e de restrição em **EARS** (Mavin et al., 2009), com ids estáveis `REQ-xx`. Cada requisito aponta as telas ([`../telas/inventario.md`](../telas/inventario.md)) e os contratos ([`contratos/openapi.yaml`](contratos/openapi.yaml), por `operationId`) em que aparece. Os cenários de qualidade com medida estão em [`qualidade.md`](qualidade.md).

## Convenções

- **O SISTEMA** = backend LibrAI (API + agente tutor + reconhecedor + banco). **O CLIENTE** = aplicativo web no navegador.
- Códigos de erro são o campo `error.code` do schema `Error` do OpenAPI; o status HTTP vem entre parênteses.
- Moldes: **U** ubíquo · **E** evento · **S** estado · **I** indesejado · **O** opcional.

### Parâmetros de configuração

Os números abaixo aparecem nos requisitos. Os marcados com † são **provisórios**: o FI-01 §12 declara que ainda não foram medidos; eles têm valor inicial para que os testes sejam escritos agora e serão revistos depois do baseline (ver [`riscos-e-lacunas.md`](riscos-e-lacunas.md)).

| Parâmetro | Valor | Origem |
|---|---|---|
| `SESSION_SIZE` | 5 sinais | limite inferior da faixa de 5 a 10 sinais do MVP (FI-01 §4.3): sessão curta, de 5 a 15 min |
| `MAX_ATTEMPTS_PER_SIGN` | 3 | [ADR-0008](adr/0008-limitar-tentativas-por-sinal.md): garante o término da sessão e evita frustração |
| `CONFIDENCE_THRESHOLD` † | 0,80 | valor inicial; lacuna do FI-01 §12, a definir com o modelo real |
| `CAPTURE_WINDOW` | 3 s · até 90 quadros (30 fps) | duração típica de um sinal isolado com folga; limita o tamanho do corpo |
| `MIN_HAND_FRAMES` | 10 quadros com mão detectada | ⅓ s de mão visível; abaixo disso não há gesto a classificar |
| `RECOGNIZER_TIMEOUT` | 5 s | 2,5× a meta de p95 da tentativa (REQ-50) |
| `RECOGNIZER_QUEUE_LIMIT` | 2 × número de workers | [ADR-0004](adr/0004-isolar-inferencia-em-pool-de-processos.md) |
| `LLM_TIMEOUT` | 3 s | o feedback não pode estourar a meta de p95 da tentativa (REQ-50) |
| `SESSION_TTL` | 12 h | um dia de uso sem pedir login de novo |
| `LOGIN_LOCKOUT` | 5 falhas em 15 min → bloqueio de 15 min | proporcional ao risco (conta educacional, sem dado financeiro) |
| `MAX_UPLOAD` | 50 MB · `video/mp4` ou `video/webm` | vídeo de referência curto (< 30 s) em 1080p |
| `MIN_GROUP_SIZE` | 3 alunos | anonimato mínimo dos agregados por turma (REQ-40) |

---

## 1. Contas e acesso

| ID | Molde | Requisito | Telas | Contrato |
|---|---|---|---|---|
| REQ-01 | U | O SISTEMA DEVE exigir sessão autenticada (cookie `librai_session`) em todas as operações, exceto `login` e `create_account`. | todas | todas |
| REQ-02 | I | SE o e-mail ou a senha informados em `login` não conferirem, ENTÃO O SISTEMA DEVE responder `invalid_credentials` (401) sem indicar qual dos dois campos está errado. | G01 | `login` |
| REQ-03 | I | SE ocorrerem 5 falhas de `login` para o mesmo `email` em 15 minutos, ENTÃO O SISTEMA DEVE recusar novas tentativas desse `email` por 15 minutos com `too_many_attempts` (429) e cabeçalho `Retry-After`. | G01 | `login` |
| REQ-04 | E | QUANDO uma conta for criada por `create_account`, O SISTEMA DEVE criá-la com `role = student` e `status = active`, ignorando qualquer papel enviado pelo cliente. | G02 | `create_account` |
| REQ-05 | I | SE o `email` informado em `create_account` já existir, ENTÃO O SISTEMA DEVE responder `email_taken` (409) e não criar conta. | G02 | `create_account` |
| REQ-06 | U | O SISTEMA DEVE aceitar senhas de 8 a 128 caracteres, armazená-las apenas como hash Argon2id em `password_hash` e responder `invalid_payload` (422) a senhas fora desse tamanho. | G02, C04 | `create_account`, `admin_create_user` |
| REQ-07 | I | SE o usuário autenticado chamar uma operação reservada a outro `role`, ENTÃO O SISTEMA DEVE responder `forbidden` (403) sem devolver dados do recurso. | todas | todas |
| REQ-56 | E | QUANDO a sessão do usuário completar `SESSION_TTL` (12 h), O SISTEMA DEVE responder `session_expired` (401) e O CLIENTE DEVE levar o usuário a G01 preservando a rota de destino. | todas | todas |

## 2. Consentimento e privacidade da câmera

| ID | Molde | Requisito | Telas | Contrato |
|---|---|---|---|---|
| REQ-08 | S | ENQUANTO o aluno não tiver `consent` do tipo `camera_processing` com `revoked_at` nulo, O SISTEMA DEVE recusar `submit_attempt` com `consent_required` (403). | A01, A02, A03 | `grant_consent`, `submit_attempt`, `get_home` |
| REQ-09 | U | O CLIENTE DEVE extrair os pontos das mãos (`landmarks`) no próprio navegador e enviar à API apenas coordenadas numéricas; quadros de imagem ou vídeo da câmera NÃO DEVEM sair do aparelho. | A02, A03 | `submit_attempt` |
| REQ-10 | I | SE o corpo de `submit_attempt` contiver campos fora do schema `AttemptRequest` (por exemplo, imagem em base64) ou passar de 512 KB, ENTÃO O SISTEMA DEVE responder `invalid_payload` (422) sem gravar nada. | A03 | `submit_attempt` |
| REQ-11 | U | O SISTEMA DEVE descartar os `landmarks` assim que a inferência terminar e gravar da tentativa somente os campos da tabela `attempt` (sem coordenadas, imagem ou vídeo). | A02, A03 | `submit_attempt` |
| REQ-12 | E | QUANDO o aluno revogar o consentimento `camera_processing`, O SISTEMA DEVE preencher `revoked_at` e mudar para `abandoned` a `practice_session` ativa desse aluno, se houver. | A02 | `revoke_consent` |

## 3. Sessão de prática e agente tutor

| ID | Molde | Requisito | Telas | Contrato |
|---|---|---|---|---|
| REQ-13 | E | QUANDO o aluno sem sessão ativa chamar `start_session`, O SISTEMA DEVE criar uma `practice_session` com `status = active`, `current_position = 0`, `sign_ids` com até `SESSION_SIZE` sinais `published` em ordem de `curriculum_order`, e copiar `confidence_threshold` e `max_attempts_per_sign` da configuração vigente; resposta 201. | A01, A05 | `start_session` |
| REQ-14 | I | SE não houver nenhum `sign` com `status = published`, ENTÃO O SISTEMA DEVE responder `no_signs_available` (409) a `start_session`. | A01 | `start_session`, `get_home` |
| REQ-15 | S | ENQUANTO o aluno tiver uma `practice_session` com `status = active`, O SISTEMA DEVE responder a `start_session` com essa sessão (200) em vez de criar outra; no banco existe no máximo uma sessão ativa por aluno. | A01 | `start_session` |
| REQ-16 | E | QUANDO `submit_attempt` for aceito, O SISTEMA DEVE fazer o agente tutor chamar a ferramenta `recognize_sign` com os `landmarks` e obter `recognized_label` e `confidence`. | A03 | `submit_attempt` · ferramenta `recognize_sign` |
| REQ-17 | E | QUANDO `recognized_label` for igual ao `recognizer_label` do sinal atual e `confidence` ≥ `confidence_threshold` da sessão, O SISTEMA DEVE decidir `ADVANCE` com `decision_reason = correct`, gravar `sign_outcome = mastered` e passar ao próximo sinal. (AC-1 do FI-01) | A04 | `submit_attempt` · ferramenta `decide_next` |
| REQ-18 | E | QUANDO `recognized_label` for diferente do esperado ou `confidence` < `confidence_threshold`, O SISTEMA DEVE decidir `REPEAT` com `decision_reason` `wrong_sign` ou `low_confidence`, mantendo o mesmo sinal atual. (AC-2 do FI-01) | A04 | `submit_attempt` · ferramenta `decide_next` |
| REQ-19 | I | SE a tentativa que seria `REPEAT` for a de número `max_attempts_per_sign` para aquele sinal na sessão, ENTÃO O SISTEMA DEVE decidir `ADVANCE` com `decision_reason = max_attempts_reached` e gravar `sign_outcome = not_mastered`. (exceção à REQ-18; [ADR-0008](adr/0008-limitar-tentativas-por-sinal.md)) | A04 | `submit_attempt` · ferramenta `decide_next` |
| REQ-20 | U | O SISTEMA DEVE calcular `decision` apenas entre `REPEAT` e `ADVANCE`, por regra determinística sobre o resultado do reconhecimento; o modelo de linguagem só redige `feedback_text` (até 280 caracteres) e não altera a decisão. | A04 | `submit_attempt` · ferramentas `decide_next`, `generate_feedback` |
| REQ-21 | I | SE a geração de `feedback_text` pelo modelo de linguagem falhar ou passar de `LLM_TIMEOUT` (3 s), ENTÃO O SISTEMA DEVE usar o texto-modelo do `decision_reason`, marcar `feedback_source = template` e concluir a tentativa normalmente. | A04 | `submit_attempt` · ferramenta `generate_feedback` |
| REQ-22 | E | QUANDO a decisão `ADVANCE` ocorrer no último sinal de `sign_ids`, O SISTEMA DEVE mudar a sessão para `finished`, preencher `finished_at` e devolver `session_status = finished`. | A04, A05 | `submit_attempt`, `skip_sign` |
| REQ-23 | E | QUANDO o aluno chamar `skip_sign` com sessão ativa, O SISTEMA DEVE gravar `sign_outcome = skipped` para o sinal atual e avançar `current_position`, sem registrar decisão do agente. | A04 | `skip_sign` |
| REQ-24 | I | SE `submit_attempt` ou `skip_sign` chegar para sessão com `status` diferente de `active`, ENTÃO O SISTEMA DEVE responder `session_not_active` (409) sem gravar nada. | A03, A04 | `submit_attempt`, `skip_sign` |
| REQ-25 | I | SE `submit_attempt` chegar com um `client_attempt_id` já processado na mesma sessão, ENTÃO O SISTEMA DEVE devolver a tentativa já gravada (200) sem nova inferência, nova decisão ou mudança de estado. ([ADR-0003](adr/0003-garantir-idempotencia-e-versao-otimista.md)) | A03 | `submit_attempt` |
| REQ-26 | I | SE `expected_sign_id` em `submit_attempt` ou `skip_sign` for diferente do sinal atual da sessão, ENTÃO O SISTEMA DEVE responder `stale_sign` (409) com o estado atual da sessão e não gravar nada. | A03, A04 | `submit_attempt`, `skip_sign` |
| REQ-27 | I | SE o reconhecedor não responder em `RECOGNIZER_TIMEOUT` (5 s) ou sua fila estiver no `RECOGNIZER_QUEUE_LIMIT`, ENTÃO O SISTEMA DEVE responder `recognizer_unavailable` (503) com `Retry-After` e não gravar tentativa nem decisão. ([ADR-0004](adr/0004-isolar-inferencia-em-pool-de-processos.md)) | A03 | `submit_attempt` |
| REQ-28 | I | SE os `landmarks` enviados tiverem menos de `MIN_HAND_FRAMES` (10) quadros com mão detectada, ENTÃO O SISTEMA DEVE responder `no_hand_detected` (422) e a tentativa não conta para `attempt_number`. | A02, A03 | `submit_attempt` |
| REQ-29 | E | QUANDO o aluno abrir A01 ou A03 com sessão ativa (inclusive depois de recarregar a página), O SISTEMA DEVE devolver em `get_active_session` o sinal atual, a posição, o número da próxima tentativa e a `last_attempt`, para retomar de onde parou. | A01, A03, A04 | `get_active_session`, `get_home` |
| REQ-30 | U | O SISTEMA DEVE gravar a `attempt`, o `sign_outcome` (quando houver) e a mudança de `current_position`/`status`/`version` da sessão numa única transação; a sessão nunca avança sem a tentativa gravada. ([ADR-0005](adr/0005-usar-sqlite-em-modo-wal.md)) | A03, A04 | `submit_attempt`, `skip_sign` |
| REQ-53 | E | QUANDO o agente decidir `ADVANCE`, O SISTEMA DEVE gravar um `sign_outcome` por (`session_id`, `sign_id`) com `attempts_count` e `best_confidence` do sinal na sessão. | A04, A05 | `submit_attempt` |
| REQ-58 | E | QUANDO o aluno chamar `abandon_session` com sessão ativa, O SISTEMA DEVE mudar o status para `abandoned`, preencher `finished_at` e manter o resumo disponível. | A03, A05 | `abandon_session` |

## 4. Resultado, progresso, estudo e turma (aluno)

| ID | Molde | Requisito | Telas | Contrato |
|---|---|---|---|---|
| REQ-31 | E | QUANDO uma sessão estiver `finished` ou `abandoned`, O SISTEMA DEVE devolver em `get_session_summary`, para cada sinal da sessão, `gloss`, `outcome` (`mastered`, `not_mastered`, `skipped` ou `not_reached`), `attempts_count` e `best_confidence`; SE a sessão não for do aluno autenticado, ENTÃO deve responder `not_found` (404). | A05 | `get_session_summary` |
| REQ-32 | O | ONDE a funcionalidade `progress_history` estiver habilitada (pós-MVP), O SISTEMA DEVE devolver em `get_my_progress`, por sinal praticado, `sessions_count`, `mastered_count` e `last_practiced_at`; com a funcionalidade desligada, deve responder `feature_disabled` (404). | A06 | `get_my_progress` |
| REQ-33 | U | O SISTEMA DEVE expor aos alunos apenas sinais com `status = published`; `get_sign` de sinal em outro status deve responder `not_found` (404). | A07 | `get_sign` |
| REQ-34 | U | O SISTEMA DEVE devolver junto de todo sinal exibido ao aluno os campos `regional_variant` e `source_credit`, e O CLIENTE DEVE mostrá-los. | A03, A07 | `get_sign`, `get_active_session` |
| REQ-35 | E | QUANDO o aluno chamar `report_sign`, O SISTEMA DEVE criar `content_report` com `status = open`; SE o mesmo aluno já tiver reporte `open` para o mesmo sinal, ENTÃO deve responder `report_exists` (409). | A07 | `report_sign` |
| REQ-36 | E | QUANDO o aluno chamar `join_class` com `invite_code` válido, O SISTEMA DEVE criar `class_enrollment` (201); se já estiver matriculado, deve responder 200 sem duplicar; se o código não existir, deve responder `class_not_found` (404). | A08 | `join_class` |

## 5. Painel do professor

| ID | Molde | Requisito | Telas | Contrato |
|---|---|---|---|---|
| REQ-37 | I | SE o professor pedir turma ou aluno que não pertença a uma `class_group` com `teacher_id` igual ao seu, ENTÃO O SISTEMA DEVE responder `not_found` (404), sem revelar se o recurso existe. | P02, P03, P04 | `list_class_students`, `get_student_performance`, `list_class_difficulties` |
| REQ-38 | E | QUANDO o professor chamar `create_class`, O SISTEMA DEVE criar `class_group` com `invite_code` único de 8 caracteres (sem `0`, `O`, `1`, `I`, `L`). | P01 | `create_class` |
| REQ-39 | U | O SISTEMA DEVE mostrar ao professor apenas dados derivados de `attempt` e `sign_outcome` (contagens, taxas, datas, `outcome` por sinal); nenhum `landmark`, imagem ou vídeo do aluno é armazenado ou exibido. | P02, P03 | `list_class_students`, `get_student_performance` |
| REQ-40 | E | QUANDO o professor chamar `list_class_difficulties` para um período (padrão: 30 dias), O SISTEMA DEVE devolver os sinais em ordem decrescente de `not_mastered_rate` = (`not_mastered` + `skipped`) ÷ total de `sign_outcome` dos alunos da turma; SE menos de `MIN_GROUP_SIZE` (3) alunos distintos tiverem praticado o sinal, ENTÃO deve devolver `insufficient_data = true` e `not_mastered_rate = null` para esse sinal. | P04 | `list_class_difficulties` |

## 6. Curadoria e administração

| ID | Molde | Requisito | Telas | Contrato |
|---|---|---|---|---|
| REQ-41 | I | SE `admin_change_sign_status` pedir `published` para um sinal sem `reference_media` com `license` preenchida, `consent_confirmed = true` e `description_text` preenchido, ENTÃO O SISTEMA DEVE responder `publish_requirements_missing` (422) listando o que falta. | C02 | `admin_change_sign_status` |
| REQ-42 | E | QUANDO `admin_change_sign_status` for chamado, O SISTEMA DEVE aceitar só as transições `draft → in_review`, `in_review → published`, `in_review → draft`, `published → archived` e `archived → draft`, gravando `reviewed_by` e `reviewed_at` na publicação; qualquer outra deve responder `invalid_transition` (409). | C01, C02 | `admin_change_sign_status` |
| REQ-43 | I | SE `admin_update_sign` ou `admin_change_sign_status` chegar com `version` diferente da gravada, ENTÃO O SISTEMA DEVE responder `version_conflict` (409) com a versão atual e não gravar nada. | C02 | `admin_update_sign`, `admin_change_sign_status` |
| REQ-44 | I | SE o arquivo de `admin_upload_reference_media` passar de 50 MB ou não for `video/mp4` nem `video/webm`, ENTÃO O SISTEMA DEVE responder `file_too_large` (413) ou `unsupported_media_type` (415). | C02 | `admin_upload_reference_media` |
| REQ-45 | E | QUANDO um sinal passar para `archived`, O SISTEMA DEVE deixar de incluí-lo em novas sessões e mantê-lo nas sessões ativas que já o contêm. | C01, C02 | `admin_change_sign_status`, `start_session` |
| REQ-46 | I | SE `admin_resolve_report` vier sem `resolution_note` ou com `status` diferente de `resolved`/`dismissed`, ENTÃO O SISTEMA DEVE responder `invalid_payload` (422). | C03 | `admin_resolve_report` |
| REQ-47 | I | SE `admin_update_user` remover o papel `admin` ou desativar o último administrador ativo, ENTÃO O SISTEMA DEVE responder `last_admin` (409). | C04 | `admin_update_user` |
| REQ-48 | I | SE o `recognizer_label` de um sinal não estiver entre os rótulos devolvidos por `admin_list_recognizer_labels`, ENTÃO O SISTEMA DEVE responder `unknown_label` (422) em `admin_create_sign`/`admin_update_sign`. | C02 | `admin_create_sign`, `admin_update_sign`, `admin_list_recognizer_labels` |
| REQ-59 | E | QUANDO o administrador chamar `admin_create_user`, O SISTEMA DEVE criar a conta com o `role` informado (`teacher` ou `admin`); SE o `email` já existir, ENTÃO deve responder `email_taken` (409). | C04 | `admin_create_user` |

## 7. Qualidade, operação e comportamento do cliente

| ID | Molde | Requisito | Telas | Contrato |
|---|---|---|---|---|
| REQ-49 | U | O SISTEMA DEVE gravar em cada `attempt` `inference_ms` e `total_ms` e emitir um log estruturado (JSON) por tentativa com `session_id`, `attempt_id`, `decision`, `decision_reason`, `feedback_source`, `inference_ms` e `total_ms`. | — | `submit_attempt` |
| REQ-50 | U | O SISTEMA DEVE responder `submit_attempt` com p95 de `total_ms` ≤ 2.000 ms e p99 ≤ 4.000 ms sob 10 sessões simultâneas. † (meta provisória; ver [`qualidade.md`](qualidade.md) QS-04) | A03 | `submit_attempt` |
| REQ-51 | S | ENQUANTO O CLIENTE estiver sem conexão, ele DEVE exibir aviso persistente, desabilitar as ações que gravam dados e não enfileirar envios (os `landmarks` não são guardados para reenvio). | todas | — |
| REQ-52 | U | O CLIENTE DEVE apresentar todo resultado com texto + ícone + cor e oferecer no vídeo de referência pausar, repetir e velocidade 0,5×, além do `description_text` do sinal. | A03, A04, A07 | `get_active_session`, `get_sign` |
| REQ-54 | U | O CLIENTE DEVE exibir indicador de carregamento em toda operação que passe de 300 ms e não exibir dado anterior como se fosse atual. | todas | — |
| REQ-55 | U | O CLIENTE DEVE exibir resultado de tentativa, decisão do agente ou confirmação de gravação somente depois da resposta 200/201 da API. | A03, A04, C02 | `submit_attempt`, `admin_update_sign` |
| REQ-57 | I | SE a API responder 5xx ou não responder, ENTÃO O CLIENTE DEVE exibir mensagem em linguagem simples com a ação "Tentar de novo" e preservar os dados digitados. | todas | — |
| REQ-60 | U | O CLIENTE DEVE, quando uma lista ou um formulário estiver sem dados, exibir uma frase com o motivo e, quando existir, a ação que resolve, mantendo a ação primária desabilitada enquanto faltar campo obrigatório. | G01, G02, A02, A06, A08, C02, C03, C04, P01, P02, P03, P04 | — |
| REQ-61 | U | O CLIENTE DEVE anunciar por região `aria-live` toda confirmação de sucesso (gravação, envio, carga de resultado) e, quando a operação levar a outra tela, mover o foco para o título da tela de destino. | G01, G02, A02, A04, A05, A06, A07, C01, C03, C04, P02, P03 | — |

---

## Ids livres

Não há ids pulados: REQ-01 a REQ-61 existem todos. Novos requisitos recebem o próximo número livre, e um requisito removido mantém o id com o texto `[removido]`, para não quebrar a rastreabilidade.

## Requisitos × critérios de aceitação do FI-01

| AC do FI-01 | Requisitos | Teste previsto (ver [`qualidade.md`](qualidade.md) QS-05) |
|---|---|---|
| AC-1 Avançar após tentativa reconhecida corretamente | REQ-17, REQ-53 | `test_decide_next_advances_when_label_matches_and_confidence_above_threshold` |
| AC-2 Repetir após tentativa não reconhecida corretamente | REQ-18 | `test_decide_next_repeats_on_wrong_label`, `test_decide_next_repeats_on_low_confidence` |
| (novo) Término garantido | REQ-19 | `test_decide_next_advances_after_max_attempts` |
