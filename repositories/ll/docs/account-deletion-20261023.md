# LL 계정 삭제 계약과 운영 조건

LL은 Core에서 확정한 계정 삭제 요청을 받아 개인 학습 DB 행과 사용자 소유 오디오를 정리한다. 삭제가 접수되면 learner를 잠가 일반 요청과 늦은 결과 저장을 막는다. 저장 파일 삭제와 DB 정리가 모두 끝난 경우에만 `COMPLETED`를 반환한다. Core는 같은 작업 ID로 `PENDING`을 재호출한다.

## 내부 API

| 항목 | 계약 |
| --- | --- |
| 삭제 | `POST /internal/v1/language-learning/accounts/{userId}/deletion` |
| 준비 확인 | `GET /internal/v1/language-learning/accounts/{userId}/deletion/capabilities` |
| 인증 | 기존 내부 JWT의 서명과 issuer, audience, service를 확인하며 전용 `tokenUse=ll-account-deletion`을 요구한다. |
| 권한 | `scopes`는 정확히 `["accounts:delete"]`, `roles`는 생략 또는 빈 배열이어야 한다. |
| 소유자 | JWT `sub`와 경로의 양수 정수 `userId`가 같아야 한다. 앞에 0을 붙인 식별자는 허용하지 않는다. |
| 수명 | JWT의 발급 시각과 만료 시각이 필요하고 TTL은 기존 내부 최대 수명과 120초 중 작은 값 이하다. |
| 준비 응답 | HTTP 200과 정확히 `{"version":"ll-account-deletion-v1","ready":true}`를 반환해야 접수할 수 있다. 미준비는 HTTP 503이다. |

POST 본문은 `operationId`와 `requestedAtUtc` 문자열 두 개다. 작업 ID는 표준 소문자 UUID이며 최초 요청의 값을 재사용한다. 시각은 UTC Instant이며 1970년 이후와 현재 시각보다 최대 5초 앞까지 허용한다. 본문은 UTF-8 JSON으로 2048바이트 이하다.

```json
{"operationId":"aaaaaaaa-bbbb-4ccc-8ddd-eeeeeeeeeeee","requestedAtUtc":"2026-10-23T03:00:00Z"}
```

성공한 호출은 HTTP 200과 정확히 네 필드를 반환한다.

```json
{"operationId":"aaaaaaaa-bbbb-4ccc-8ddd-eeeeeeeeeeee","userId":73001,"status":"PENDING","pendingObjects":1}
```

`status`는 `PENDING` 또는 `COMPLETED`이고 `pendingObjects`는 0 이상의 정수다. 진행 중 쓰기와 삭제 대기 파일 수가 겹칠 수 있으므로 단순 전체 파일 개수로 사용하지 않는다. 같은 계정의 다른 작업 ID는 HTTP 409로 거부한다. 완료 후 같은 작업 ID를 재호출해도 데이터를 다시 만들지 않는다. 응답은 `Cache-Control: no-store`다.

## 실제 삭제 범위

개인 학습 그래프 56개 테이블에서 사용자 설정과 키워드, Writing 답안과 평가 및 생성 요청, Listening 결과와 DB 오디오 바이트, Level Test와 Practice의 세션과 자식 자료, Speaking 세션과 평가 및 사용량, 성장·활동 지표, Web AI 실행과 결과를 소유자 조건 및 FK 순서에 맞춰 삭제한다. learner에는 재생성을 막는 `DELETED` 상태와 식별 버전이 남고, 삭제 작업에는 계정 ID와 작업 ID 및 상태·시각만 남는다.

Level Test 오디오 저장소와 Speaking 오디오 저장소의 실제 삭제 함수를 호출한다. 아직 메타데이터에 연결되지 않은 파일도 쓰기 의도 기록에서 수집한다. 공용 Level Test pool과 타 사용자 파일은 삭제 대상에 포함하지 않는다. 모든 객체 삭제가 확인되기 전에는 개인 DB 행의 정리 완료를 알리지 않는다.

사용자가 생성한 Writing catalog가 다른 사용자의 고정 문항에 참조된 경우 catalog의 원문과 내용 해시, 주제 및 의미 키와 승인자 연결을 익명화하고 `RETIRED`로 만든다. 타 사용자의 고정된 문항·답안은 보존한다. 참조가 없는 개인 생성 catalog는 삭제한다. 공용 정책 값과 감사 이력은 유지하되 삭제한 계정의 actor 연결은 비운다.

## 경합과 미확정 파일 쓰기

파일 쓰기는 learner 잠금 아래 영속 의도를 남긴 뒤 DB 트랜잭션 밖에서 실행한다. 정상 성공을 확인했거나 실제 전송 전 종료를 확인한 경우에만 의도를 종료한다. 전송 뒤 취소·timeout·프로세스 중단 등으로 원격 PUT의 종료 여부를 알 수 없으면 의도를 유지하고 `PENDING`으로 남긴다. 기록된 lease 시각이 지났다는 이유로 자동 완료하지 않는다.

미확정 상태는 일반 오류 이유 `AUDIO_WRITE_UNCERTAIN`으로 구분한다. 운영자가 실제 원격 쓰기의 종료와 파일 상태를 확인하기 전에는 완료를 강제로 알리지 않는다. 일반 상태·오류 로그에는 작업 ID와 일반 이유만 쓰고 원본 객체 키, 음성, 사용자 답안과 공급자 응답을 넣지 않는다. 이 기능에는 임의로 잔여 상태를 지우는 운영 API가 없다.

현재 LL 프로세스의 Speaking 성공 캐시와 대기 중 Writing 피드백은 삭제 시 비운다. 다른 새 버전 노드에서도 캐시 응답 직전과 공급자 응답 뒤 learner 및 세션의 DB 상태를 확인해 접근을 거부한다. 다른 노드의 RAM 바이트를 즉시 물리적으로 소거했다는 보장은 하지 않는다.

## 배포와 준비 확인

1. 기존 schema를 보존하고 새 `V042__account_deletion.sql`을 적용하는 LL 버전을 배포한다. 적용된 이전 migration과 Flyway checksum을 바꾸지 않는다.
2. 모든 LL HTTP 및 worker 인스턴스를 이 버전으로 교체하고 구 프로세스와 기존 PUT의 종료를 확인한다. LB에서 HTTP 트래픽만 제외한 채 구 worker를 남겨두지 않는다. 혼합 버전에서 삭제 기능을 먼저 열지 않는다.
3. 내부 API와 DB를 사용하며 Level Test 및 Speaking의 실제 저장소를 모두 구성한다. readiness는 새 테이블 조회와 저장소 DI를 확인한다. readiness가 실제 S3 삭제 권한이나 특정 객체의 삭제 성공을 미리 보장하지 않으므로 실제 호출 실패는 `PENDING`으로 남는다.
4. Core의 adapter capability 확인이 성공한 뒤 전체 계정 삭제 기능을 활성화한다. Core는 LL `PENDING`을 전체 완료로 해석하지 않는다.
5. 새 migration 이후 구 LL 이미지의 Flyway 검증이 실패할 수 있다. schema와 삭제 이력을 되돌려 계정을 되살리는 복구를 하지 않고 기능을 닫고 호환 버전으로 전진 수정한다.

V042 이전에 만들어졌고 DB 메타데이터에도 없는 UUID 오디오 파일은 사용자별 소유자를 이 계약으로 복원할 수 없다. 기능을 활성화하기 전에 운영자가 저장소와 메타데이터를 대조하여 미매핑 개인 파일의 존재 여부를 확인한다. 잔여가 있다면 소유 확인이나 정리 범위가 확정되기 전에는 전체 개인 파일 삭제를 보장하는 기능으로 활성화하지 않는다. 타인 자료가 있는 저장소 전체를 무차별 삭제하지 않는다.

## 완료 문구의 범위

완료는 이 서비스가 관리하는 활성 DB와 해당 오디오 저장 파일의 정리 및 접근 차단 범위다. 외부 AI 공급자의 로그와 법적 보존 자료, 백업의 삭제 시점까지 확인한 것으로 표시하지 않는다. 외부 보존·백업 정책은 별도 운영 범위로 관리한다.
