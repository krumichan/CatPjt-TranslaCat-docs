# 이메일 비밀번호 복구 계약 / メールによるパスワード復旧契約

## 공개 경계 / 公開境界

개발 계약이며 운영 활성화 기록이 아니다. 배포 담당이 DB·BE·공개 edge를 검토하고 적용한다. 실제 사용자 계정의 비밀번호를 바꾸는 운영 테스트는 자동 실행하지 않는다.

開発契約であり本番有効化記録ではない。配備担当がDB・BE・公開edgeを確認して適用する。実ユーザーのパスワードを変更する本番テストは自動実行しない。

2026-10-24 데스크톱의 새 소스 사본에서 전체 Java 컴파일과 인증 13클래스·84테스트를 통과했다. H2의 실제 migration 적용·JPA 매핑 검증을 포함한다. 운영 DB 적용·SMTP 실발송·전체 BE 테스트 통과를 뜻하지 않는다.

2026-10-24、デスクトップの新規ソースコピーで全Javaコンパイルと認証13クラス・84テストに通過した。H2への実migration適用・JPAマッピング検証を含む。本番DBへの適用・SMTP実送信・BE全テストの成功を意味しない。

| POST 경로 / パス | JSON 입력 / 入力 | body 응답 / 応答 |
| --- | --- | --- |
| `/api/v1/auth/password-reset/challenges` | `email` | `challengeId`, `expiresIn`, `resendAfterSeconds` |
| `/api/v1/auth/password-reset/verify` | `challengeId`, `code` | `verificationToken`, `expiresIn` |
| `/api/v1/auth/password-reset/complete` | `email`, `verificationToken`, `password` | `success: true` |

별도 인증 없는 exact POST만 허용한다. GET·임의 하위 경로를 열지 않는다. 앱 가입에도 기존 `/api/v1/auth/register`, `/api/v1/auth/email/challenges`, `/api/v1/auth/email/verify`의 exact POST 개방이 필요하다. 현재 공개 HTTPS 경계에는 이 세 기존 경로도 노출되지 않았다는 읽기 전용 운영 관측이 있다. 각 요청의 본문 한도(제안 8KiB)와 기존 인증 rate limit, TLS·proxy 설정을 함께 적용하고 내부 경계나 포괄적 `/auth/**`를 열지 않는다.

認証不要のexact POSTだけを許可し、GET・任意の下位経路を開放しない。アプリ登録には既存の`/api/v1/auth/register`、`/api/v1/auth/email/challenges`、`/api/v1/auth/email/verify`のexact POST開放も必要。現在の公開HTTPS境界には、この既存3経路も公開されていないという読み取り専用観測がある。各リクエストの本文上限（提案8KiB）と既存認証rate limit、TLS・proxy設定を併せて適用し、内部経路や包括的な`/auth/**`を開放しない。

## DB·메일·세션 / DB・メール・セッション

- 신규 migration: `scripts/sql/20261024_password_reset.sql`. 기존 `user`에 nullable `password_reset_at DATETIME(6)`를 추가한다. 이미 있으면 실제 형식·nullable을 확인하고 중복 실행하지 않는다. 다른 계정의 기존 값은 추정해 채우지 않는다. 기존 local-auth/session migration과 email quota 행 1이 선행 조건이다. quota의 현 카운터를 초기화하지 않는다.

  新migration：`scripts/sql/20261024_password_reset.sql`。既存`user`へnullableの`password_reset_at DATETIME(6)`を追加する。既存の場合は実際の型・nullableを確認し、重複実行しない。他アカウントの値を推測で補完しない。既存local-auth/session migrationとemail quota行1が前提。quotaの現在カウンターを初期化しない。
- `RESET_PASSWORD`는 기존 challenge의 별도 purpose다. 미가입·Google 전용·비활성 계정도 같은 코드 메일과 응답 형태를 받는다. 증명은 발송 당시 유효한 비밀번호 계정 ID에만 묶인다. 이후 같은 이메일로 가입한 계정에는 사용할 수 없다. 가입·비밀번호 추가용 증명도 복구에 사용할 수 없다.

  `RESET_PASSWORD`は既存challengeの別purpose。未登録・Google専用・非アクティブのアカウントにも同じコードメールと応答形式を返す。証明は送信時に有効なパスワードアカウントIDだけに結び付ける。後から同じメールで登録したアカウントには使えず、登録・パスワード追加用の証明も復旧に使えない。
- 기존 SMTP/HMAC·60초 재발송·이메일별 시간당 5회·전역 일일 quota·코드 5회 검증 제한을 공유한다. 발송은 DB 잠금 밖에서 실행한다. complete는 단일 BE의 `AuthAttemptLimiter`도 사용한다. 다중 인스턴스 제한으로 확대해 설명하지 않는다.

  既存SMTP/HMAC、再送60秒、メール別毎時5回、全体日次quota、コード検証5回の制限を共有する。送信はDBロック外で実行する。completeは単一BEの`AuthAttemptLimiter`も使用する。複数インスタンス全体の制限とは説明しない。
- 사용자 잠금 → 증명 소비 → BCrypt 변경 → 기존 sid 폐기·legacy refresh 삭제는 동일 트랜잭션이다. 실패 시 함께 롤백된다. reset 계정은 sid 없는 legacy access/refresh를 허용하지 않는다. 새 로그인에는 독립 sid를 사용한다. 기존 WebSocket의 즉시 강제 종료까지 검증했다고 주장하지 않는다.

  ユーザーロック→証明消費→BCrypt変更→既存sid失効・legacy refresh削除は同一トランザクションで、失敗時は一括ロールバック。reset済みアカウントにはsidなしlegacy access/refreshを許可しない。新ログインには独立sidを使用する。既存WebSocketの即時強制切断まで検証したとは主張しない。

## 배포 순서·복구 / 配備順序・復旧

1. 정확한 소스의 CI·집중 검증과 합성 DB migration을 확인한다. 실제 SMTP 설정은 값 대신 활성화·필수 항목 존재 여부만 확인한다.

   正確なソースのCI・集中検証と合成DB migrationを確認する。実SMTP設定は値ではなく有効化・必須項目の存在だけを確認する。
2. DB additive 컬럼과 기존 prerequisite를 확인한 뒤 새 BE를 배포한다. 새 endpoint는 edge 개방 전까지 외부에서 호출할 수 없게 유지할 수 있다.

   DBの追加カラムと既存前提を確認してから新BEを配備する。edge開放前まで新endpointを外部から呼べない状態に保てる。
3. 검토한 exact POST 경로만 edge에 반영한다. 테스트 APK 준비·전달과 정식 정책 공개를 구분한다. 현재 정책 문서는 검토용 초안이며 앱 URL은 정식 공개 전까지 `null`로 유지한다. 실메일·가입·복구의 최종 확인은 사용자 소유 테스트 계정으로 사용자가 수행한다.

   確認したexact POST経路だけをedgeへ反映する。テストAPKの準備・配布と正式ポリシー公開を区別する。現在の文書は確認用の案であり、正式公開までアプリURLを`null`に保つ。実メール・登録・復旧の最終確認は、ユーザー所有のテストアカウントでユーザーが行う。
4. 문제 시 edge에서 신규 복구 POST를 먼저 닫는다. 이미 변경된 비밀번호·증명 소비·폐기 세션을 되돌리거나 컬럼을 삭제하지 않는다. reset 계정의 legacy-token 차단을 모르는 이전 BE로 단순 롤백하면 보안 경계가 약해질 수 있으므로 해당 차단을 유지하는 수정 배포를 우선 검토한다.

   問題時はedgeで新規復旧POSTを先に閉じる。変更済みパスワード・消費済み証明・失効セッションを戻したりカラムを削除したりしない。reset済みアカウントのlegacy-token拒否を認識しない旧BEへ単純に戻すと認証境界が弱まるため、その拒否を保持する修正配備を優先して検討する。
