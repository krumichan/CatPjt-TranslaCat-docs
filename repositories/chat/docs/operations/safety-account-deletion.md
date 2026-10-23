# CHAT 안전 정책과 계정 삭제 운영

CHAT은 신고 증거를 일반 대화와 분리하고, 차단 관계에 따라 본문과 알림을 가리며, 탈퇴한 작성자를 `탈퇴한 사용자`로 표시한다. 공유 메시지·번역·AI 답변의 본문과 연결은 유지한다. 이 기능의 운영 활성화는 아래 migration, 기존 작업 종료, 이미지 목록 대조를 마친 뒤 별도로 진행한다.

## 설정과 배포 순서

`Chat:Safety:Enabled`와 `Chat:Safety:AccountDeletionEnabled`의 기본값은 모두 `false`다. .NET 환경 변수로 설정하면 각각 `Chat__Safety__Enabled`, `Chat__Safety__AccountDeletionEnabled`를 사용한다. `NormalWriteCoverageComplete=true`는 제품 코드의 쓰기 경계 검증이 끝났다는 의미다. 운영 준비를 대신하지 않는다.

OCI workflow에서는 공개 Actions Variables `CHAT_SAFETY_ENABLED`, `CHAT_ACCOUNT_DELETION_ENABLED`를 step 환경으로 읽는다. `ship.py`는 이를 runtime JSON의 `safety.enabled`, `safety.account_deletion_enabled` boolean으로 전달하고, `deploy.py`가 위 두 .NET 환경 행을 새 runtime.env에 기록한다. 기존 runtime.env를 조용히 덮어쓰지 않는다. 두 변수는 각각 정확한 소문자 `true` 또는 `false`만 허용하며, 공백·대문자·숫자·추가 줄은 거절한다.

변수가 비어 있으면 기존의 유효한 JSON boolean을 보존하고 최초 미구성만 false로 시작한다. 명시적으로 끄려면 공개 변수를 `false`로 설정한다. `safety`의 알 수 없는 필드나 boolean이 아닌 기존 값은 공개 override로 복구하지 않고 배포 전에 거절한다. CHAT JSON 원문의 루트 `safety` 중복이나 그 하위 필드 중복도 두 CLI 입구에서 고정 오류로 거절한다. 다른 기존 JSON 필드의 중복 처리와 LL 설정은 바꾸지 않는다. 두 flag 조합은 독립적으로 유지한다. 이 전달 경로는 Secret·인증 키·다른 runtime 설정을 바꾸지 않으며, DB 전용 migration 환경에는 worker flag를 추가하지 않는다. schema-before-code와 아래 활성화 준비 조건은 그대로 적용한다.

| 설정 | 일반 개인 OPEN 이미지 | 신고와 차단 | 내부 탈퇴 접수 |
| --- | --- | --- | --- |
| 두 설정 모두 false | 기존 업로드·교체·삭제 경로, journal 미등록 | 비활성 | 503 |
| Safety만 true | PUT 전 journal 필수 | 활성 | 503 |
| AccountDeletion만 true | PUT 전 journal 필수 | 신고 비활성, 탈퇴 보호 정책 적용 | 준비 검사 후 허용 |
| 두 설정 모두 true | PUT 전 journal 필수 | 활성 | 준비 검사 후 허용 |

이번 릴리스는 schema를 코드보다 먼저 배포한다. 기존 migration은 수정하거나 재적용 기록을 초기화하지 않는다. 현재 DB의 적용 이력을 확인하고 다음 additive migration을 순서대로 적용한다.

1. `20261022160852_ChatSafetyAndAccountDeletion`: 탈퇴 tombstone, 정확한 객체 키 정리 큐, 신고, 신고 검토 이력.
2. `20261022164144_ChatAccountWriteFence`: 계정별 DB 쓰기 gate.
3. `20261023014452_ChatOpenProfileUploadIntents`: 개인 OPEN 이미지 PUT와 이전 키의 durable journal.

마지막 기존 기준은 `20261022071017_ChatNotificationPreferences`이며, 최신 모델은 26개 업무 테이블이다. 두 safety 설정이 모두 false인 일반 이미지 endpoint는 journal 테이블을 읽지 않지만, 하나라도 켜면 새 schema와 journal DI가 필수다. 준비가 빠지면 외부 PUT 전에 거절한다.

운영 활성화 전에는 다음 조건을 함께 충족한다.

1. BE의 pending 이메일 예약, 현재 계정 상태 확인, 세션 폐기 및 전체 소유 서비스 완료 판정 코드가 배포되어 있어야 한다.
2. 구 BE·CHAT 노드와 journal 도입 전 진행 중이던 이미지 PUT가 종료되어 있어야 한다. 두 설정이 모두 false인 새 노드에서 시작한 PUT도 같은 이전 작업으로 취급한다.
3. 이후 모든 개인 OPEN 이미지 쓰기는 journal을 사용하는 노드로만 보낸다. 기존 `open-chat-profiles/` 객체는 소유 DB 참조·공유 참조·제공자의 실제 작업 완료 증거와 대조한다. journal은 이미 사라진 DB 참조나 도입 전 orphan을 역으로 발견하지 못한다.
4. 공유 AI 이미지나 다른 사람의 참조를 보존하고, 정리 대상으로 확정한 정확한 키만 삭제한다. prefix 전체 삭제, 오래되었다는 이유만의 삭제, 불명 PUT의 시간 기반 완료는 사용하지 않는다.
5. CHAT·LL 및 BE 소유 정리 서비스의 준비 응답이 모두 확인된 뒤 BE의 공개 탈퇴 접수를 활성화한다. 운영 데이터 확인과 실제 삭제 실행은 배포 담당 절차로 수행한다.

## 내부 탈퇴 계약

`GET /internal/v1/chat/accounts/{userId}/deletion/capabilities`는 성공 시 `{ "version": "chat-account-deletion-v1", "ready": true }`를 반환한다. 설정, Core 객체 삭제 adapter, journal DI 및 필수 테이블의 전체 컬럼 projection을 검사한다. 필수 컬럼 하나가 빠져도 503이다. 이 조회는 업무 데이터를 변경하지 않는다.

`POST /internal/v1/chat/accounts/{userId}/deletion`은 `operationId`의 정규 소문자 UUID, UTC offset이 0인 `requestedAtUtc`, 선택적인 내부 `auditIdentity`를 받는다. `auditIdentity`는 최대 320자이며 로그·공개 응답에 넣지 않는다. 같은 계정과 operation의 재시도는 기존 정리를 이어 가고, 다른 operation으로 충돌하면 409다.

두 endpoint 모두 별도 서비스 토큰을 요구한다. `tokenUse=chat-account-delete`, 단일 scope `chat:account-delete`, 토큰 subject와 경로 userId 일치, 최대 120초 수명을 적용한다. 사용자 JWT와 일반 `chat:http` ingress 토큰은 허용하지 않는다. 응답은 `no-store`다.

결과는 `operationId`, 내부 `userId`, `status=PENDING|COMPLETED`, `completedAtUtc`다. BE는 내부 userId를 공개 상태 DTO로 전달하지 않는다. DB 정리와 모든 개인 객체 정리가 끝나야 COMPLETED다. 불명 PUT가 남으면 PENDING이며 시간 경과만으로 성공 처리하지 않는다.

## 계정 정리와 잠금 순서

정상 쓰기와 삭제는 같은 계정 gate를 사용한다. 여러 계정은 ID 오름차순으로 잠그고, 그 뒤 ledger와 OPEN·방·멤버 업무 row를 잠근다. 실제로 획득한 계정 집합은 현재 DB transaction ID에만 유효하다. pooled DbContext의 이전 transaction 기록은 잠금 증명이 될 수 없다.

삭제는 멤버십을 비활성화하고 개인 설정·알림 수신 업무·대기 push를 제거하거나 억제한다. 방장은 남은 관리자 우선, 가입 시각과 멤버 ID 순으로 승계한다. 남은 사람이 없으면 방을 종료한다. OPEN 닉네임과 개인 이미지, 감사 식별값을 익명화한다. 내부 최소 계정 ID와 tombstone은 늦은 요청과 재삽입을 거절하는 데 사용한다.

BE는 모든 소유 서비스가 끝날 때까지 같은 이메일 재가입을 예약한다. CHAT은 계정 gate 획득 시각까지 이전 감사값을 정리한다. 이미 local 정리가 끝난 operation의 재시도는 감사값 정리를 반복하지 않아 이후 새 계정의 행을 건드리지 않는다.

## 개인 OPEN 이미지 journal

새 PUT 전에 현재 키와 새 키의 intent를 별도 context에서 독립 commit한다. 이 별도 transaction은 부모의 계정 gate를 다시 얻지 않으며 계정·멤버 FK를 만들지 않는다. PUT ACK도 독립 기록하고, 이후 개인 profile 참조를 commit한다. 교체나 명시적 이미지 제거는 이전 키를 잃기 전에 기록한다.

worker는 잠금 없는 후보 조회 뒤 계정 gate, ledger row, 현재 참조 순서로 확인한다. 따라서 PUT가 ACK되었지만 부모 transaction이 아직 commit하지 않은 순간의 참조 부재를 삭제 근거로 쓰지 않는다. 실제 DB 참조는 이전 PUT 완료의 양의 증거다. 활성 참조는 보존하고, 탈퇴 뒤 다른 사용자·AI가 참조하는 공유 객체도 보존한다.

현재 참조가 없고 PUT 완료가 확인된 정확한 키는 실제 Core DELETE ACK 뒤 journal에서 제거한다. DELETE 실패는 durable 재시도 상태로 남는다. PUT 응답 유실이나 프로세스 종료로 완료를 모르면 `PUT_UNCERTAIN`으로 보존한다. DELETE ACK나 365일 경과도 진행 중 PUT의 종료 증거가 되지 않는다. 늦은 PUT ACK는 기존 intent만 깨우며 개인 profile·membership을 다시 만들지 않는다.

## 차단과 지연된 실시간 전달

BE의 기존 양방향 차단 관계를 재사용한다. 차단 중에는 과거 메시지도 `contentVisibility=BLOCKED` placeholder로 반환하고 원문·번역·계정 표시를 가린다. ID와 cursor를 보존하고, 해제하면 저장된 원문을 다시 표시한다. 차단 발신자는 unread·badge 집계와 push 대상에서 제외한다.

REST와 실시간 DTO는 `contentVisibility=VISIBLE|BLOCKED|MODERATED`, `authorDeleted`를 제공한다. 삭제된 작성자의 전역 계정 ID·이메일·이전 닉네임·개인 이미지가 공개 표시로 복원되지 않도록 현재 상태를 투영한다.

실시간 전송은 socket sendLock을 얻은 뒤 수신자·멤버십·작성자 상태를 확인한다. 원격 관계 조회를 기다린 뒤 tombstone을 다시 확인하고, 준비를 기다리는 동안 닫힌 socket이나 해제된 구독에는 보내지 않는다. 지연된 OPEN profile 이벤트는 현재 작성자 상태로 거절하며, private 초대 이벤트는 현재 notification ID·수신자·삭제 상태를 다시 조회한다. 이미 socket이나 push provider에 넘긴 바이트의 회수를 보장하지 않는다.

## 신고와 증거 보관

신고 생성은 `POST /api/v1/chat/rooms/{roomId}/reports`다. MESSAGE 대상은 해당 방에서 접근 가능한 messageId 하나만 받는다. 자기 USER 메시지와 SYSTEM은 제외하고 AI 메시지는 허용한다. MEMBER 대상은 OPEN에서 targetOpenChatMemberId, 그 밖의 방에서 targetUserId 하나만 받는다. 혼합 대상과 자기 신고를 거절한다.

이유는 `SPAM`, `HARASSMENT`, `HATE`, `SEXUAL`, `ILLEGAL`, `PERSONAL_INFORMATION`, `OTHER`다. detail은 최대 2,000자이며 clientRequestId UUID로 중복을 식별한다. 같은 신고자가 같은 UUID를 다른 내용에 재사용하면 409다. 공개 receipt는 실제 신고자·대상 계정 ID를 노출하지 않는다.

운영 조회·처리는 `/api/v1/admin/chat/reports`, `/{id}`, `/{id}/status`를 사용하며 ADMIN 역할과 `chat:admin` scope가 모두 필요하다. 상태는 PENDING, IN_REVIEW, RESOLVED, DISMISSED다. PATCH는 expectedVersion과 최대 2,000자의 reviewNote를 받고, 같은 값의 명시적 저장도 version과 감사 이력을 증가시킨다. 신고나 상태 변경만으로 자동 제재하지 않는다.

증거 보관은 사용자가 선택한 서비스 정책인 90일이다. 법정 보관 기간을 주장하지 않는다. 기한부터 detail·본문 snapshot·작성자 표시·처리 메모를 응답에서 숨기고, 분당 worker가 100건씩 실제 저장값도 정리한다. 신고자·대상 계정·대상 멤버·메시지 연결, 자유 서술, fingerprint, 검토자의 계정 연결과 과거 검토 메모를 지운다. 기한 뒤 새 자유 서술 메모는 받지 않는다.

남는 기록은 reportId, roomId, 임의 clientRequestId, 대상 종류·이유 enum·상태·version·시각·만료/정리 시각과 상태 변경 이력이다. 일반 공유 대화의 원문 보존은 신고 증거 보관과 별개다.

## 검증과 복구

중앙 검증 경로는 `.codex-workspace/verification/shared/mobile-safety-deletion-20261023/chat/`다. `final-source-freeze/manifest.json`은 현재 CHAT 변경 80파일의 SHA와 사본을 담는다. 이전 제품 79파일은 그대로이며, 마지막 추가 한 파일은 API parser fixture의 합성 JWT·JSON 준비 예열이다. 최초 26파일 승인 후보, guard=false 이미지 검토 사본, 실패 재현 로그도 별도로 보존한다.

| 최종 검증 | 결과 | 근거 |
| --- | --- | --- |
| 전체 solution build | 경고 0, 오류 0 | build-release-candidate.log |
| Unit | 264 통과, 실패 0, skip 0 | tests/unit-release-candidate.trx |
| API | 647 통과, 실패 0, skip 0 | tests/api-final-serial.trx |
| MySQL Redis HTTP STOMP 통합 | 402 통과, 실패 0, skip 0 | tests/integration-release-candidate.trx |

API 최종 결과는 `api-serial.runsettings`로 test collection을 직렬 실행한 결과다. 기본 병렬 실행에서는 기존 1초 loopback fixture가 upstream 진입 전에 취소된 실패를 보존했다. 별도 진단에서 첫 요청은 JWT·JSON cold 준비 뒤 RequestStart가 1,723ms에 발생했고, 같은 host 두 번째 요청은 122ms에 실제 응답을 받았다. parser fixture 준비만 예열했으며 제품의 인증 판정·1초 제한·assertion·전용 timeout 테스트는 바꾸지 않았다. 정확한 커밋의 CI 결과는 배포 담당이 별도로 확인한다.

검증은 실행 소유 MySQL 8.4·Redis·Kestrel/STOMP와 합성 계정·메시지를 사용한다. 외부 계정·파일 provider는 합성이므로 실제 운영 prefix inventory나 제공자의 저장소 완료 증거를 대신하지 않는다. 격리 검증 리소스는 종료 후 데이터·volume·network를 보존한다.

문제가 있으면 BE의 공개 탈퇴 접수를 먼저 비활성화하고 CHAT의 `AccountDeletionEnabled`를 내려 새 접수를 중단한다. 정상 쓰기 차단과 이미지 journal이 필요한 동안은 `Safety:Enabled`를 유지한다. 이것도 내려야 한다면 먼저 해당 노드의 쓰기를 멈추고 진행 중 요청을 배출한다. 이미 시작된 BE 예약·tombstone·객체 journal을 지우거나 성공으로 바꾸지 않는다. 코드와 정확한 pending 상태를 함께 대조해 재시도한다. 운영 rollback에서 additive 테이블을 자동 drop하거나 전역 reset/clean으로 소스를 덮어쓰지 않는다.
