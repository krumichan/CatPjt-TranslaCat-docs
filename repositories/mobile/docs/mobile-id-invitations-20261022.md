# 웹·앱 ID 초대와 웹 방장 독립 위임

# Web・アプリの ID 招待と Web の独立したオーナー委任

2026-10-22 사용자는 이전 앱을 테스트했다고 보고하면서 두 개선을 요청했다. 웹 OPEN 방장 위임이 나가기와 묶인 점을 분리하고, 웹과 앱 모두 상대 TranslaCat ID로 초대를 보내 상대 채팅에서 링크를 받아 직접 참여할 수 있게 한다. 이전 ID 복사·공유 등의 `ASSUMED_BY_USER` 기록을 실제 관찰로 바꾸거나 이번 기능의 수락으로 확대하지 않는다.

2026-10-22、ユーザーは以前のアプリをテストしたと報告し、2 つの改善を依頼した。Web の OPEN オーナー委任と退室の結び付きを分離し、Web・アプリの両方から相手の TranslaCat ID 宛てに招待を送り、相手がチャットでリンクを受け取って自分で参加できるようにする。以前の ID コピー・共有などの `ASSUMED_BY_USER` 記録を実観察に置き換えたり、今回の機能の受け入れへ広げたりしない。

## 분담과 추적

## 分担と追跡

웹 담당 기존 「조사 및 이행 계획 수립」 채팅이 FE 독립 위임과 초대 UI를 맡는다. 이 앱 개발 채팅은 CHAT API·영속 초대 푸시와 모바일을 병행하고 통합 검증한다. 초기 읽기 준비와 단계별 인계 뒤 「배포 책임과 규칙 기억하기」가 **CHAT·migration·공개 경로의 1단계와 FE 27파일만의 2단계 운영 배포를 모두 완료**했다. 새 Android APK도 파일 검증까지 완료했으며 실제 초대·푸시·참여의 단말 수락은 대기다. 단계별 상세와 과거 상태는 아래에서 구분한다.

Web 担当の既存「조사 및 이행 계획 수립」チャットが FE の独立委任と招待 UI を担当する。このアプリ開発チャットは CHAT API・永続招待プッシュとモバイルを並行して進め、統合検証する。初期の読み取り準備と段階別の引き継ぎ後、「배포 책임과 규칙 기억하기」が **CHAT・migration・公開経路の第 1 段階と、FE 27 ファイルだけの第 2 段階の本番デプロイをともに完了**した。新しい Android APK もファイル検証まで完了し、実際の招待・プッシュ・参加の端末受け入れは待機中である。段階別の詳細と過去の状態は以下で区別する。

초대는 [기존 Project 12](https://github.com/users/krumichan/projects/12)의 [전용 #54](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/54), 웹 독립 위임은 [기존 #45 댓글](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/45#issuecomment-6332367644)로 추적한다.

招待は[既存 Project 12](https://github.com/users/krumichan/projects/12) の[専用 #54](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/54)、Web の独立委任は[既存 #45 コメント](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/45#issuecomment-6332367644)で追跡する。

## 동작과 공통 계약

## 動作と共通契約

현재 활성 OPEN 사람 참여자는 역할과 무관하게 한 명의 TranslaCat ID로 초대를 보낼 수 있다. 이는 기존 링크 공유 권한과 맞춘다. 서버는 방 상태·참여자·대상 존재·자기 초대·양방향 차단·방 재입장 차단·이미 참여를 검사한다. 대상은 자동 가입되지 않는다. 일반 DIRECT 친구 정책이나 GROUP 즉시 가입 API를 바꾸지 않는다.

現在アクティブな OPEN の人間参加者は、役割に関係なく 1 人の TranslaCat ID 宛てに招待できる。これは既存リンク共有の権限に合わせる。サーバーはルーム状態・参加者・対象の存在・自己招待・双方向ブロック・再入室ブロック・参加済みを検査する。対象は自動加入しない。一般 DIRECT の友達ポリシーや GROUP の即時加入 API は変更しない。

채팅 목록의 **오픈채팅 초대**는 저장된 수신 초대를 대화형 카드로 보여주는 시스템 수신함이다. 가짜 관리자 계정이나 일반 DIRECT 방을 만들지 않는다. 메시지는 방별 초대한 사람의 이름과 멤버 코드만 사용하고 일반 계정 정보는 공개하지 않는다. 검증한 roomId로 내부 OPEN 미리보기를 열고, 현재 상태와 방별 닉네임을 확인해 기존 JOIN API로 참여한다.

チャット一覧の **오픈채팅 초대（オープンチャット招待）** は、保存済みの受信招待を対話型カードとして表示するシステム受信箱である。架空の管理者アカウントや一般 DIRECT ルームは作らない。メッセージは招待者のルーム別の名前とメンバーコードだけを使用し、一般アカウント情報を公開しない。検証済み roomId で内部 OPEN プレビューを開き、現在の状態とルーム別ニックネームを確認してから既存 JOIN API で参加する。

| 경로 | 계약 |
| --- | --- |
| POST `/chat/open-rooms/{roomId}/invitations` | `{targetPublicId, requestId: UUID}` → `{invitationId, roomId, createdAt, duplicate}` |
| GET `/chat/open-invitations` | descending `cursorId`, `size` → `{items, nextCursorId, hasNext}` |
| GET `/chat/open-invitations/summary` | `{unreadCount, latestInvitation}` |
| GET `/chat/open-invitations/{id}` | 현재 계정 소유의 초대 항목 |
| PATCH `/chat/open-invitations/{id}/read` | 현재 계정 소유의 초대를 읽음 처리한 항목 |

| 経路 | 契約 |
| --- | --- |
| POST `/chat/open-rooms/{roomId}/invitations` | `{targetPublicId, requestId: UUID}` → `{invitationId, roomId, createdAt, duplicate}` |
| GET `/chat/open-invitations` | 降順 `cursorId`、`size` → `{items, nextCursorId, hasNext}` |
| GET `/chat/open-invitations/summary` | `{unreadCount, latestInvitation}` |
| GET `/chat/open-invitations/{id}` | 現在のアカウントが所有する招待項目 |
| PATCH `/chat/open-invitations/{id}/read` | 現在のアカウントが所有する招待を既読にした項目 |

항목은 기존 활동 DTO `{id, notificationType, roomId, payload, isRead, readAt, createdAt}`를 쓴다. `notificationType=CHAT_INVITATION`, payload는 `{schemaVersion:1, invitationKind:"OPEN_LINK", roomType:"OPEN", roomName, inviterOpenNickname, inviterMemberCode}`다. source prefix/type 필터와 recipient scope를 pagination 이전에 적용한다. 기존 개인 activity queue는 갱신 신호로 사용하며 화면 복귀·재연결의 REST 조회로 누락을 보완한다.

項目は既存の活動 DTO `{id, notificationType, roomId, payload, isRead, readAt, createdAt}` を使う。`notificationType=CHAT_INVITATION`、payload は `{schemaVersion:1, invitationKind:"OPEN_LINK", roomType:"OPEN", roomName, inviterOpenNickname, inviterMemberCode}`。source prefix/type フィルターと recipient scope を pagination より前に適用する。既存の個人 activity queue は更新の合図として使い、画面復帰・再接続時の REST 取得で取りこぼしを補う。

같은 시도의 requestId는 유지하고 서버는 발신자·대상·방과 결합해 중복 생성을 막는다. 모바일은 계정 세션·방·대상별로 결과 불명 시도의 키를 보존해, A에게 보낸 뒤 B에게 성공했거나 늦은 A 응답이 도착해도 A의 재확인 키를 잃지 않는다. 클라이언트는 POST를 자동 재전송하지 않는다. 발신자 전체 60초 10건과 발신자·대상·방 조합의 60초 제한을 영속 잠금으로 검사한다. 기존 비즈니스 오류 경계에 맞춰 HTTP400과 `OPEN_INVITATION_*` 코드를 사용한다.

同じ試行の requestId は維持し、サーバーは送信者・対象・ルームと組み合わせて重複生成を防ぐ。モバイルはアカウントセッション・ルーム・対象ごとに結果不明な試行のキーを保持し、A への送信後に B への送信が成功した場合や、遅い A の応答が届いた場合も A の再確認キーを失わない。クライアントは POST を自動再送しない。送信者全体の 60 秒 10 件制限と、送信者・対象・ルームの組み合わせの 60 秒制限を永続ロックで検査する。既存の業務エラー境界に合わせ、HTTP400 と `OPEN_INVITATION_*` コードを使う。

## 영속 푸시와 배포 조건

## 永続プッシュとデプロイ条件

기존 메시지 outbox는 USER/TEXT와 가입한 참여자를 전제로 하므로 비회원 초대에 재사용하지 않는다. 별도 초대 outbox·delivery와 작은 발신자 gate를 additive migration으로 추가하고 초대 저장·outbox 생성·제한 갱신을 같은 transaction에 묶는다. 기존 Expo 전송·receipt parser, BE 기기 등록 조회와 세대 검사를 재사용한다.

既存メッセージ outbox は USER/TEXT と加入済み参加者を前提とするため、非メンバーへの招待には再利用しない。専用の招待 outbox・delivery と小さな送信者 gate を additive migration で追加し、招待保存・outbox 作成・制限更新を同じ transaction にまとめる。既存の Expo 送信・receipt parser、BE の端末登録取得・世代検査を再利用する。

새 push data는 `{schemaVersion:2, kind:"OPEN_INVITATION", eventId, notificationId, roomId, accountBindingId}`다. 기존 schema1 채팅 알림은 보존한다. 앱은 계정 binding과 수신 초대 소유권·방 일치를 REST로 확인한 뒤 OPEN 미리보기로 이동한다. 앱에서 아직 참여하지 않은 방이라는 이유만으로 초대 탭을 거절하지 않는다. 공급자에게 이미 전송한 푸시는 회수할 수 없으며 실제 단말 전달을 서버 단위 검사로 완료 처리하지 않는다.

新しい push data は `{schemaVersion:2, kind:"OPEN_INVITATION", eventId, notificationId, roomId, accountBindingId}`。既存 schema1 のチャット通知は維持する。アプリはアカウント binding と受信招待の所有権・ルーム一致を REST で確認後、OPEN プレビューへ移動する。アプリでまだ参加していないルームという理由だけで招待タップを拒否しない。プロバイダーへ送信済みのプッシュは回収できず、実端末への配信をサーバー単位の検査で完了扱いにはしない。

전송 직전에 초대 소유 관계와 미읽음·미삭제 상태, 현재 방과 발신자 참여, 수신자 미참여·재입장 제한·양방향 차단, 기기의 현재 account binding과 token 세대, lease 소유권을 재검사한다. Redis 이벤트는 저장된 초대의 갱신 신호이며 누락은 REST로 복구한다. 영속 재시도는 at-least-once이고 외부 공급자 수락 뒤에는 회수할 수 없다. 공급자 전송 뒤 ticket 저장 전 프로세스가 중단되면 같은 안정적인 eventId가 중복 전달될 수 있다. 다른 서비스의 관계·기기 변경과 CHAT transaction 사이에 분산 원자성을 보장하지 않는다.

送信直前に、招待の所有関係・未読と未削除の状態、現在のルームと送信者の参加、受信者の未参加・再入室制限・双方向ブロック、端末の現在の account binding と token 世代、lease 所有権を再検査する。Redis イベントは保存済み招待の更新の合図であり、欠落は REST で復旧する。永続再試行は at-least-once で、外部プロバイダー受理後は回収できない。プロバイダー送信後、ticket 保存前にプロセスが停止すると、同じ安定した eventId が重複配信される場合がある。他サービスの関係・端末変更と CHAT transaction 間の分散原子性は保証しない。

새 migration과 공개 역프록시 경로 허용 여부는 배포의 선행 조건이었다. migration `20261022041228_OpenInvitationDurablePush`는 `chat_open_invitation_sender_gate`, `open_invitation_push_outbox`, `open_invitation_push_delivery` 세 테이블과 기존 `chat_notification`의 actor/source 조회 index만 추가한다. 기존 enum·payload·사용자 데이터의 일괄 변경은 없다. 최초 검증에서는 격리 DB에만 적용했고, 이후 1단계 배포 담당의 완료 영수증에서 운영 migration 이력·새 테이블·index와 opaque 식별자 6컬럼의 `ascii_bin` collation, runtime SELECT/INSERT/UPDATE 권한을 확인했다. 추가 grant 변경은 없었다.

新しい migration と公開リバースプロキシ経路の許可は、デプロイの前提条件だった。migration `20261022041228_OpenInvitationDurablePush` は `chat_open_invitation_sender_gate`、`open_invitation_push_outbox`、`open_invitation_push_delivery` の 3 テーブルと、既存 `chat_notification` の actor/source 取得 index だけを追加する。既存 enum・payload・ユーザーデータの一括変更はない。初回検証では隔離 DB にのみ適用し、その後、第 1 段階デプロイ担当の完了記録で、本番 migration 履歴・新テーブル・index・opaque 識別子 6 カラムの `ascii_bin` collation・runtime の SELECT/INSERT/UPDATE 権限を確認した。追加の grant 変更はなかった。

1단계 인계는 검증된 CHAT 24파일과 아래 다섯 method·공개 경로만 포함한다. 기존 공개 40경로·TLS/WSS·내부 접근 경계를 유지한다. FE·BE·모바일·Docs Git 변경을 이 인계에 포함하지 않으며 BE 소스 변경·재배포는 필요하지 않다. 문제가 있으면 이전 image·공개 경로 설정으로 복구하되, 추가 테이블·index·이미 저장된 데이터와 migration 이력을 삭제하지 않는다. 기존 키·의존성·유료 서비스의 변경도 없다.

第 1 段階の引き継ぎは、検証済み CHAT 24 ファイルと以下の 5 method・公開経路だけを含む。既存公開 40 経路・TLS/WSS・内部アクセス境界を維持する。FE・BE・モバイル・Docs の Git 変更をこの引き継ぎには含めず、BE ソース変更・再デプロイは不要である。問題時は以前の image・公開経路設定へ復旧するが、追加テーブル・index・保存済みデータ・migration 履歴は削除しない。既存キー・依存関係・有料サービスの変更もない。

- POST `/api/v1/chat/open-rooms/{roomId}/invitations`
- GET `/api/v1/chat/open-invitations`
- GET `/api/v1/chat/open-invitations/summary`
- GET `/api/v1/chat/open-invitations/{invitationId}`
- PATCH `/api/v1/chat/open-invitations/{invitationId}/read`

- POST `/api/v1/chat/open-rooms/{roomId}/invitations`
- GET `/api/v1/chat/open-invitations`
- GET `/api/v1/chat/open-invitations/summary`
- GET `/api/v1/chat/open-invitations/{invitationId}`
- PATCH `/api/v1/chat/open-invitations/{invitationId}/read`

## 검증과 보존 상태

## 検証と保存状態

아래 표는 **2026-10-22 13:47 KST의 배포·APK GO 시점에 보존한 검증 스냅샷**이다. 표의 FE 검토 중·운영 반영 확인·APK 대기는 당시 상태이며, 이어지는 최신 단계별 기록이 이를 갱신한다. 문서 갱신 과정에서 시험을 재실행하지 않았다. 원래 실패·집중 재검증 범위는 그대로 유지한다.

以下の表は **2026-10-22 13:47 KST のデプロイ・APK GO 時点に保存した検証スナップショット**。表中の FE レビュー中・本番反映確認・APK 待機は当時の状態で、その後の最新の段階別記録がこれを更新する。文書更新の際に試験は再実行していない。元の失敗と集中再検証の範囲はそのまま維持する。

| 범위 | 확인한 결과 | 남은 범위 |
| --- | --- | --- |
| CHAT 소스 | 동결 입력 327개, 수정 9개·신규 15개로 총 변경 24개, 검토 시 소스 차이 0개 | 운영 image·migration·경로 반영 확인 |
| CHAT Unit·API | Unit 242/242, API 596/596 통과, 각 fail·skip 0 | 실제 공급자·운영 사용자 시나리오 |
| 격리 MySQL 통합 | 마지막 전체 실행 342개 중 341개 통과·1개 실패. 새 자체 outbox FK를 외부 FK로 오판한 기존 검사 heuristic을 **테스트만 수정한 뒤 해당 1개 재실행 통과** | 그 수정 뒤 전체 342개는 재실행하지 않음. 전체 342/342 통과로 표시하지 않음 |
| 초대 집중 검사 | 앞선 초대 DB 집중 39/39 통과. 이후 권한 사례를 더한 초대 관련 42개는 위 전체 통합 실행에 포함 | 집중 수와 전체 수를 중복 합산하지 않음 |
| 모바일 소스 | 직전 128개 보존, 최종 입력 138개, 기존 11개 수정·10개 신규로 총 변경 21개 | 실제 Android·iPhone UI와 두 계정 흐름 |
| 모바일 검사 | 418/418 통과, fail·skip 0. lint 경고 0·타입·공개 HTTPS 설정·Android/iOS/web JS export 모두 exit 0, 검사 중 소스 차이 0개 | JS export는 native 설치·렌더링·실제 푸시 수용 증거가 아님 |
| FE | 담당 인계상 ID 초대 발신 UI와 방에 남아 있는 독립 방장 위임 수정 범위는 준비됨 | 초대 수신함의 늦은 이전 계정 401 처리·이미 불러온 pagination 유지에 대한 검토 결함을 수정 중. 최종 FE 완료·2단계 배포 인계는 아직 아님 |
| Android APK | 최종 138입력·418검사를 근거로 기존 device profile의 **무료 cloud build 1회** 요청 승인을 빌드 담당에게 전달 | 빌드 완료·파일 검증·설치 링크·단말 수락 대기. 유료 빌드 승인 아님 |

| 範囲 | 確認結果 | 残る範囲 |
| --- | --- | --- |
| CHAT ソース | 凍結入力 327 件、変更 9 件・新規 15 件で計 24 件の変更、レビュー時のソース差分 0 件 | 本番 image・migration・経路反映の確認 |
| CHAT Unit・API | Unit 242/242、API 596/596 通過、各 fail・skip 0 | 実プロバイダー・本番ユーザーシナリオ |
| 隔離 MySQL 統合 | 最後の全体実行 342 件中 341 件通過・1 件失敗。新しい自己 outbox FK を外部 FK と誤判定した既存検査の heuristic を、**テストだけ修正して該当 1 件を再実行し通過** | 修正後に全体 342 件は再実行していない。全体 342/342 通過とは表示しない |
| 招待の集中検査 | 先行する招待 DB 集中 39/39 が通過。その後権限例を加えた招待関連 42 件は、上記全体統合実行に含まれる | 集中数と全体数を重複合算しない |
| モバイルソース | 直前 128 件を保存、最終入力 138 件、既存変更 11 件・新規 10 件で計 21 件の変更 | 実際の Android・iPhone UI と 2 アカウントの流れ |
| モバイル検査 | 418/418 通過、fail・skip 0。lint 警告 0・型・公開 HTTPS 設定・Android/iOS/web JS export はすべて exit 0、検査中のソース差分 0 件 | JS export は native インストール・描画・実プッシュ受け入れの証拠ではない |
| FE | 担当の引き継ぎによると、ID 招待送信 UI とルームに残る独立オーナー委任の修正範囲は準備済み | 招待受信箱の古いアカウントからの遅延 401 処理・取得済み pagination 保持のレビュー不具合を修正中。最終 FE 完了・第 2 段階デプロイ引き継ぎにはまだ至っていない |
| Android APK | 最終 138 入力・418 検査を根拠に、既存 device profile の**無料 cloud build 1 回**の要求承認をビルド担当へ伝達 | ビルド完了・ファイル検証・インストールリンク・端末受け入れ待ち。有料ビルドの承認ではない |

채팅 목록에서 **오픈채팅 초대 수신함으로 받는 구조는 구현**됐다. 수신함은 저장된 OPEN_LINK 초대 카드만 별도로 보여주며 실제 DIRECT 방이나 가짜 관리자 계정을 만들지 않는다. 이 구조 설명은 별도의 브라우저 시험 통과를 뜻하지 않고, 운영·기기에서의 초대 수신은 미검증이다.

チャット一覧で **オープンチャット招待受信箱として受け取る構造は実装済み**。受信箱は保存済み OPEN_LINK 招待カードだけを別表示し、実際の DIRECT ルームや架空の管理者アカウントは作らない。この構造の説明は別途のブラウザー試験通過を意味せず、本番・端末での招待受信は未検証である。

독립 검토는 recipient·source 필터의 pagination 이전 적용, 계정별 상세·읽음 소유권, v2 push의 account binding·초대 소유권·방 일치 확인, payload URL 대신 내부 경로만 사용, 명시적 JOIN과 결과 불명 POST 자동 재시도 금지, 화면·계정 변경 경계를 확인했다. 검토에서 발견한 대상별 requestId 유실은 대상별 map과 회귀 검사로 보완했다. 최종 서버 독립 검토의 새 차단 결함은 0개이며, 원래 실패 기록과 테스트만 수정한 후의 집중 재검증도 구분해 보존한다. 독립 검토 담당이 실제 Expo·기기·운영 시험을 실행한 것은 아니다.

独立レビューは、recipient・source フィルターの pagination 前適用、アカウント別の詳細・既読所有権、v2 push の account binding・招待所有権・ルーム一致確認、payload URL でなく内部経路だけの使用、明示的 JOIN と結果不明 POST の自動再試行禁止、画面・アカウント変更境界を確認した。レビューで見つかった対象別 requestId の喪失は、対象別 map と回帰検査で補った。最終サーバー独立レビューの新たな阻害不具合は 0 件で、元の失敗記録とテストのみ修正後の集中再検証も区別して保存する。独立レビュー担当が実際の Expo・端末・本番試験を実行したわけではない。

중앙 근거는 `.codex-workspace/verification/shared/mobile-id-invitations-20261022/`다.

中央の根拠は `.codex-workspace/verification/shared/mobile-id-invitations-20261022/`。

- `mobile/completion.json`, `mobile/checks-final.json`, `mobile/validation-sources-final.json`: 모바일 138입력·418검사와 다섯 검사 job 결과. 최종 source manifest SHA-256은 `cb69bbf59f9a8a203710b382a51ca5960c0abfcd664102e368a7c01e593c2ab7`, checks SHA-256은 `85570ba64c6ff30c7a0064066271d7c625ccb53ff8a89cc0a301096a09a729cf`다.
- `chat/server-completed.json`, `review/final-server-review.json`: 서버 327입력·24변경, 원본 TRX·migration과 독립 해시 대조. 서버 source pin SHA-256은 `329ed96971810ad1e79c53e4a8c8bd3f9a50ab2e319eb56ee3684a6a35b5c6f9`다.
- `push/push-completed.json`, `mobile/implementation-handoff.json`: v2 영속 푸시와 수신함·미리보기·동일 시도 보존 계약 및 검토 보완 이력. 푸시 영수증의 앞선 전체 통합 339/342 기록 뒤 최신 서버 전체 실행은 위 표의 341/342이며 어느 쪽도 전체 통과 기록이 아니다.
- `stage1-chat-release-manifest.json`: CHAT·공개 경로만의 1단계 배포 인계. `root-apk-go.json`: 무료 Android 빌드 단일 요청 GO. 둘 다 배포·빌드 완료 영수증이 아니다.
- `issues/initial-tracking-confirmed.json`: #54 OPEN·Project 12 In progress와 #45 요청 댓글의 재조회 일치. 기존 체크·이슈 상태·`ASSUMED_BY_USER` 이력을 보존한다.

- `mobile/completion.json`、`mobile/checks-final.json`、`mobile/validation-sources-final.json`：モバイル 138 入力・418 検査と 5 件の検査 job の結果。最終 source manifest の SHA-256 は `cb69bbf59f9a8a203710b382a51ca5960c0abfcd664102e368a7c01e593c2ab7`、checks の SHA-256 は `85570ba64c6ff30c7a0064066271d7c625ccb53ff8a89cc0a301096a09a729cf`。
- `chat/server-completed.json`、`review/final-server-review.json`：サーバー 327 入力・24 変更、元の TRX・migration と独立したハッシュ照合。サーバー source pin の SHA-256 は `329ed96971810ad1e79c53e4a8c8bd3f9a50ab2e319eb56ee3684a6a35b5c6f9`。
- `push/push-completed.json`、`mobile/implementation-handoff.json`：v2 永続プッシュと受信箱・プレビュー・同一試行保持の契約、レビュー補強履歴。プッシュ記録の先行全体統合 339/342 の後、最新サーバー全体実行は上表の 341/342 であり、いずれも全体通過記録ではない。
- `stage1-chat-release-manifest.json`：CHAT・公開経路だけの第 1 段階デプロイ引き継ぎ。`root-apk-go.json`：無料 Android ビルド 1 回の要求 GO。いずれもデプロイ・ビルド完了記録ではない。
- `issues/initial-tracking-confirmed.json`：#54 OPEN・Project 12 In progress と #45 依頼コメントの再取得一致。既存チェック・issue 状態・`ASSUMED_BY_USER` 履歴を保持する。

모바일 원본 128개와 서버 원본 312개, 수정 전 문서, 초기 실패·중간·최종 로그·manifest, 격리 DB·컨테이너·volume·network·데이터와 관계없는 기존 사용자 변경을 보존한다. 이 개발·문서 갱신 담당은 Git staging·commit·push, 운영 배포·DB 변경, 키·의존성·유료 서비스 변경이나 파일·데이터 삭제를 직접 실행하지 않았다. 배포 담당의 승인된 실행은 아래 별도 완료 영수증과 구분한다.

モバイル原本 128 件とサーバー原本 312 件、変更前文書、初期失敗・中間・最終ログと manifest、隔離 DB・コンテナー・volume・network・データ、および無関係な既存ユーザー変更を保存する。この開発・文書更新担当は Git staging・commit・push、本番デプロイ・DB 変更、キー・依存関係・有料サービスの変更、ファイル・データ削除を直接実行していない。デプロイ担当による承認済み実行は、以下の別個の完了記録と区別する。

## 1단계 운영 완료와 2단계 웹 배포 완료

## 第 1 段階の本番完了と第 2 段階の Web デプロイ完了

`deployment/stage1-completed.json`은 2026-10-22 **14:30:13 KST에 passed=true**로 기록됐다. CHAT 커밋 `18f9283fcd26995385d9198b6ee9add1c342ea86`의 [공식 실행 39726482761](https://github.com/krumichan/CatPjt-TranslaCat-chat/actions/runs/39726482761)에서 verify·release가 성공했고, 같은 immutable image·revision의 운영 실행을 확인했다. 새 migration 1회, 새 테이블 3개·기존 notification index·제약·식별자 collation·권한, 기존 메시지 push 스키마 보존과 정상 준비 상태가 완료 영수증에 포함된다.

`deployment/stage1-completed.json` は、2026-10-22 **14:30:13 KST に passed=true** と記録された。CHAT コミット `18f9283fcd26995385d9198b6ee9add1c342ea86` の[公式実行 39726482761](https://github.com/krumichan/CatPjt-TranslaCat-chat/actions/runs/39726482761)で verify・release が成功し、同じ immutable image・revision の本番稼働を確認した。新 migration 1 回、新テーブル 3 件・既存 notification index・制約・識別子 collation・権限、既存メッセージ push スキーマ保持、正常な準備状態が完了記録に含まれる。

공개 경로는 **기존 40패턴을 유지하고 신규 5패턴만 추가**했다. 외부 HTTPS 경계 검사 **33개(GET 30·HEAD 3) 통과**, WSS handshake **HTTP 101**을 확인했다. 이는 인증 쿠키·STOMP 업무 데이터 없이 수행한 익명 라우팅·인증 경계 검사다. POST·PATCH의 실제 초대 전송·읽음 변경, 가입 변경, provider 전송이나 단말 푸시 수신을 시험한 결과로 확대하지 않는다. 고정 관찰 구간에서 worker 실패 marker가 0개였다는 결과도 실제 provider 전달 증거는 아니다. 최초 로그 관찰 한계로 실패했던 postflight는 덮어쓰지 않고, 같은 runtime의 구간 전체를 읽은 별도 follow-up 영수증으로 해당 한계만 해소했다.

公開経路は**既存 40 パターンを保持し、新規 5 パターンだけを追加**した。外部 HTTPS 境界検査 **33 件（GET 30・HEAD 3）が通過**し、WSS handshake **HTTP 101** を確認した。これは認証 Cookie・STOMP の業務データなしで行った匿名ルーティング・認証境界検査である。POST・PATCH の実招待送信・既読変更、加入変更、provider 送信、端末プッシュ受信を試験した結果へ広げない。固定観察期間で worker の失敗 marker が 0 件だった結果も、実際の provider 配信の証拠ではない。初回ログの観察限界で失敗した postflight は上書きせず、同じ runtime の期間全体を読んだ別の follow-up 記録で、その限界だけを解消した。

공식 CI의 **Unit 242·API 596 통과**와 개발 중 격리 MySQL의 **전체 341/342 후 테스트만 수정·해당 1개 통과**는 서로 다른 검사 기록이다. 공식 배포 성공이 전체 MySQL 342/342 재실행을 뜻하지 않는다.

公式 CI の **Unit 242・API 596 通過**と、開発中の隔離 MySQL の**全体 341/342 の後にテストだけを修正し、該当 1 件が通過**した記録は別の検査記録である。公式デプロイ成功は、MySQL 全体 342/342 を再実行したことを意味しない。

웹은 이전 검토에서 남아 있던 늦은 이전 계정 401 처리·이미 불러온 수신함 페이지 유지·계정 전환 경계를 수정했다. 방에 남아 있는 독립 `TRANSFER`는 `transferOwner`만 실행하며 기존 ‘위임하고 나가기’·종료와 구분한다. 발신 시도의 세션·방·대상별 UUID 보존, 응답 기한과 늦은 응답 차단, 수신함의 append/read/refresh 동기화도 최종 검토에 반영됐다. `review/fe-fix-verification-post-lint.json`의 소스와 최종 인계 pin을 대조했고 남은 차단 결함은 0개다.

Web は以前のレビューで残っていた古いアカウントの遅延 401 処理・取得済み受信箱ページの保持・アカウント切り替え境界を修正した。ルームに残る独立した `TRANSFER` は `transferOwner` だけを実行し、既存の『委任して退室』・終了とは区別する。送信試行のセッション・ルーム・対象ごとの UUID 保持、応答期限と遅延応答の遮断、受信箱の append/read/refresh 同期も最終レビューに反映した。`review/fe-fix-verification-post-lint.json` のソースと最終引き継ぎ pin を照合し、残る阻害不具合は 0 件である。

`web/final-manifest.json`은 **이번 소유 변경 27파일**과 **기존 스크롤·레이아웃 변경 5파일의 보존**을 분리한다. 최종 저장 검사는 다음 범위로 통과했다.

`web/final-manifest.json` は、**今回担当した変更 27 ファイル**と、**既存スクロール・レイアウト変更 5 ファイルの保存**を分離する。最終の保存済み検査は、以下の範囲で通過した。

- Playwright mock Chromium: OPEN ID 초대 5개, OWNER lifecycle 3개, 기존 CHAT-03 desktop/mobile 스크롤 2개. 서로 다른 집중 범위이며 실제 BE/CHAT·계정과 연결한 운영 시험이 아니다.
- 타입 검사, 집중 lint 경고 0, ko/ja/en/learning의 i18n 17파일 검사, diff 공백 검사 모두 exit 0.
- `web/root-thread-validation.json`은 마지막 제품 수정 index 526 뒤 ID 검사 539, lifecycle 검사 605, 최종 타입 624·lint 634가 실행됐고 그 뒤 제품 수정이 없음을 별도로 확인했다. 앞선 실패·locale 불일치와 중단된 넓은 실행은 보존하며 전체 E2E 통과로 표시하지 않는다.

- Playwright mock Chromium：OPEN ID 招待 5 件、OWNER lifecycle 3 件、既存 CHAT-03 desktop/mobile スクロール 2 件。それぞれ別の集中範囲であり、実 BE/CHAT・アカウントへ接続した本番試験ではない。
- 型検査、集中 lint 警告 0、ko/ja/en/learning の i18n 17 ファイル検査、diff 空白検査がすべて exit 0。
- `web/root-thread-validation.json` は、最後の製品変更 index 526 の後に ID 検査 539、lifecycle 検査 605、最終の型 624・lint 634 が実行され、その後製品変更がないことを別途確認した。先行失敗・locale 不一致・中断した広範囲実行は保存し、全 E2E 通過とは表示しない。

mock 브라우저 검사는 기존 스크롤·레이아웃 5파일이 있는 작업 트리에서 실행됐다. **이 5파일은 실제 2단계 커밋에서 제외했고 배포 뒤에도 원본 작업 바이트 그대로 unstaged로 보존**했다. 따라서 집중 브라우저 결과를 최종 27파일 커밋 전체의 독립 재실행 증거로 바꾸지 않는다. 정확한 새 FE 커밋의 Vercel·GitHub Production 성공은 배포 담당이 아래와 같이 별도로 확인했다. 상세 Vercel build log를 열어 읽은 것은 아니다.

mock ブラウザー検査は、既存スクロール・レイアウト 5 ファイルがある作業ツリーで実行された。**この 5 ファイルは実際の第 2 段階コミットから除外し、デプロイ後も元の作業バイトのまま unstaged で保存**した。したがって、集中ブラウザー結果を最終 27 ファイルコミット全体の独立再実行証拠に置き換えない。正確な新 FE コミットの Vercel・GitHub Production 成功は、デプロイ担当が以下のとおり別途確認した。詳細な Vercel build log を開いて読んだわけではない。

`stage2-fe-release-manifest.json`의 FE 27파일만 인계한 GO 뒤, `deployment/stage2-completed.json`이 **2026-10-22 14:46:45 KST passed=true**로 완료됐다. FE 커밋은 `0f0b5b5cc70f3d60e5d634930d7dcedc08d94a4c`이며 정확한 27파일·동결 Git blob과 정상 fast-forward push를 확인했다.

`stage2-fe-release-manifest.json` の FE 27 ファイルだけを引き継いだ GO 後、`deployment/stage2-completed.json` は **2026-10-22 14:46:45 KST passed=true** で完了した。FE コミットは `0f0b5b5cc70f3d60e5d634930d7dcedc08d94a4c` で、正確な 27 ファイル・凍結 Git blob と正常な fast-forward push を確認した。

| 새 Production 근거 | 실제 기록 |
| --- | --- |
| Vercel deployment | `DPsvZvvaMJEnNVWWrRs62HtxR4Pz`, 상태 `success` |
| Vercel commit status | `56959996005`, 생성·갱신 `2026-10-22T05:38:57Z` |
| GitHub Production deployment/status | `7244700007` / `20227438596`, 상태 `success`, 생성 `2026-10-22T05:38:58Z` |
| 새 고유 URL | https://translacat-f9n9cvha9-bellline0403-1248s-projects.vercel.app |
| 실제 서비스 alias | https://translacat.vercel.app |

| 新 Production の根拠 | 実記録 |
| --- | --- |
| Vercel deployment | `DPsvZvvaMJEnNVWWrRs62HtxR4Pz`、状態 `success` |
| Vercel commit status | `56959996005`、作成・更新 `2026-10-22T05:38:57Z` |
| GitHub Production deployment/status | `7244700007` / `20227438596`、状態 `success`、作成 `2026-10-22T05:38:58Z` |
| 新しい固有 URL | https://translacat-f9n9cvha9-bellline0403-1248s-projects.vercel.app |
| 実サービス alias | https://translacat.vercel.app |

새 ID·시각·정확한 FE SHA로 확인했으며 이전 성공 상태를 재사용하지 않았다. 서비스 alias의 `/ko/login`은 신뢰 TLS·앱 HTML 200, `/`와 `/ko/chat/open/invitations`는 예상된 익명 로그인 보호 307을 확인했다. 고유 deployment URL은 Vercel protection 302여서 해당 앱 HTML을 직접 검사하지 않았고 보호를 우회하지 않았다. 익명 HTML·리다이렉트와 새 Production 성공은 브라우저 hydration·인증된 초대 화면·실제 사용자 동작 또는 단말 전달을 증명하지 않는다.

新しい ID・時刻・正確な FE SHA で確認し、以前の成功状態は再利用していない。サービス alias の `/ko/login` は信頼できる TLS・アプリ HTML 200、`/` と `/ko/chat/open/invitations` は想定どおりの匿名ログイン保護 307 を確認した。固有 deployment URL は Vercel protection 302 であるためアプリ HTML を直接検査せず、保護も迂回していない。匿名 HTML・リダイレクトと新 Production 成功は、ブラウザー hydration・認証済み招待画面・実ユーザー操作・端末配信の証明ではない。

2단계는 완료된 `deployment/stage1-completed.json`을 해시로 결합해 재사용했고 **CHAT 커밋·migration·배포·edge 적용의 중복 실행은 없었다**. BE 재배포·모바일·Docs Git, 기존 5파일과 관계없는 파일도 포함하지 않았다. 배포 후 index는 비어 있고 기존 5개 수정 및 AGENTS/CLAUDE 미추적 파일은 보존됐다. 새 키·설정·의존성·유료 작업도 없었다.

第 2 段階は完了済み `deployment/stage1-completed.json` をハッシュで結び付けて再利用し、**CHAT コミット・migration・デプロイ・edge 反映の重複実行はなかった**。BE 再デプロイ・モバイル・Docs の Git、既存 5 ファイル、無関係なファイルも含めなかった。デプロイ後の index は空で、既存 5 件の変更と AGENTS/CLAUDE の未追跡ファイルを保持した。新しいキー・設定・依存関係・有料作業もなかった。

새 APK `95ab7e46-1800-4f95-ba4d-7fda5e35145a`는 아래 완료 절 및 `root-apk-verified.json`대로 파일 확인을 마쳤다. 이 APK 준비와 배포·경계 검사는 두 계정의 실제 초대 수신·v2 푸시·미리보기·참여 수락을 대신하지 않는다. 2단계 완료 영수증을 읽은 뒤 [#54 최종 진행 댓글](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/54#issuecomment-6333748082)과 [#45 독립 위임 후속 댓글](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/45#issuecomment-6332367644)을 갱신·재조회해 내용 일치를 확인했다. 두 이슈는 OPEN·Project 12 In progress를 유지하며 기존 수동 체크와 `ASSUMED_BY_USER`를 바꾸지 않는다. 근거는 `issues/final-tracking-confirmed.json`이다.

新 APK `95ab7e46-1800-4f95-ba4d-7fda5e35145a` は、以下の完了節と `root-apk-verified.json` のとおりファイル確認を完了した。この APK 準備とデプロイ・境界検査は、2 アカウントによる実招待受信・v2 プッシュ・プレビュー・参加の受け入れを代替しない。第 2 段階完了記録を読んだ後、[#54 最終進行コメント](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/54#issuecomment-6333748082)と [#45 独立委任の後続コメント](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/45#issuecomment-6332367644)を更新・再取得し、内容一致を確認した。両 issue は OPEN・Project 12 In progress を維持し、既存手動チェックと `ASSUMED_BY_USER` は変更しない。根拠は `issues/final-tracking-confirmed.json`。

## 새 버전에서 확인할 순서

## 新バージョンで確認する順序

아래 항목은 운영 배포와 새 APK 설치가 끝난 뒤 확인할 수용 절차이며, 아직 실기기 통과 기록이 아니다.

以下は本番デプロイと新 APK インストール後に確認する受け入れ手順であり、まだ実機通過記録ではない。

1. 웹의 방장 계정으로 OPEN 방 메뉴에서 다른 사람 참여자를 방장으로 위임한다. 원래 계정이 같은 방에 남고, 두 계정의 역할이 갱신되는지 확인한다.
2. 초대받을 계정은 아직 그 방에 참여하지 않은 계정을 사용한다. 설정에서 확인한 TranslaCat ID를 방 참여자의 웹 또는 앱 초대 보내기 화면에 입력한다.
3. 수신 계정의 웹·앱 채팅 목록에서 **오픈채팅 초대**를 연다. 초대한 방과 방별 닉네임이 보이고, 방 미리보기를 열어도 즉시 가입되지 않는지 확인한다.
4. 수신 앱을 백그라운드로 두거나 정상 종료한 상태에서 새 초대를 보내 푸시를 확인한다. 알림을 누르면 해당 방 미리보기가 열리고, 방별 닉네임을 정한 후 직접 참여할 수 있어야 한다. Android의 강제 중지 상태는 일반 앱 종료와 구분한다.
5. 전송 결과가 불확실할 때 수동 재시도하고, 방 이탈·재진입과 다른 대상 입력 후에도 같은 시도의 초대가 중복 생성되지 않는지 확인한다. 계정 전환 시 이전 계정의 초대·읽음 수·버튼 잠금이 새 계정에 남지 않아야 한다.

1. Web のオーナーアカウントで OPEN ルームメニューから別の人間参加者へオーナーを委任する。元のアカウントが同じルームに残り、両アカウントの役割が更新されるかを確認する。
2. 招待先には、そのルームに未参加のアカウントを使う。設定で確認した TranslaCat ID を、ルーム参加者の Web またはアプリの招待送信画面に入力する。
3. 受信アカウントの Web・アプリのチャット一覧で **오픈채팅 초대（オープンチャット招待）** を開く。招待されたルームとルーム別ニックネームが表示され、ルームプレビューを開いても即座に加入しないかを確認する。
4. 受信アプリをバックグラウンドまたは通常終了の状態にして新しい招待を送り、プッシュを確認する。通知を押すとそのルームのプレビューが開き、ルーム別ニックネームを決めて自分で参加できる必要がある。Android の強制停止状態は通常のアプリ終了と区別する。
5. 送信結果が不確かなときに手動再試行し、ルーム離脱・再入場・別の対象入力後も、同じ試行の招待が重複生成されないかを確認する。アカウント切り替えで、前のアカウントの招待・既読数・ボタンロックが新アカウントに残ってはならない。

웹의 독립 방장 위임과 ID 초대 수신함은 서버·웹 배포 결과로, 앱의 실제 초대·푸시·미리보기·참여는 새 APK의 단말 결과로 각각 기록한다.

Web の独立オーナー委任と ID 招待受信箱はサーバー・Web のデプロイ結果として、アプリの実際の招待・プッシュ・プレビュー・参加は新 APK の端末結果として、それぞれ記録する。

## Android APK 완료 기록

## Android APK 完了記録

2026-10-22 14:15:23 KST에 [95ab7e46 빌드](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/95ab7e46-1800-4f95-ba4d-7fda5e35145a)가 FINISHED 됐다. 무료 device 빌드 1회와 파일 다운로드 1회를 사용했고, 재요청·유료 작업은 없었다. APK는 93,514,175 bytes이며 SHA-256은 `55c1861a4849e3108746fec526123248127e1ecc14ec2d6296f0d1ede3d80397`이다.

2026-10-22 14:15:23 KST に [95ab7e46 ビルド](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/95ab7e46-1800-4f95-ba4d-7fda5e35145a)が FINISHED となった。無料 device ビルド 1 回・ファイルダウンロード 1 回を使用し、再要求・有料作業はなかった。APK は 93,514,175 bytes、SHA-256 は `55c1861a4849e3108746fec526123248127e1ecc14ec2d6296f0d1ede3d80397`。

완료 metadata, 공개 설정 7개, ZIP 구조와 직전 633f 대비 새 JS bundle, 소스 138개의 빌드 전후 일치가 확인됐다. compiled Android 리소스의 AppTheme→TranslaCatTextView 참조와 `useBoundsForWidth=false`, 스타일 부모·모든 variant가 유지되며 resource table도 633f와 같다. 이는 실제 단말의 렌더링이나 초대·푸시 수신을 확인했다는 뜻은 아니다.

完了 metadata、公開設定 7 件、ZIP 構造、直前 633f と異なる新 JS bundle、ソース 138 件のビルド前後一致を確認した。compiled Android リソースの AppTheme→TranslaCatTextView 参照と `useBoundsForWidth=false`、スタイルの親・全 variant が維持され、resource table も 633f と同一である。これは実端末の描画や招待・プッシュ受信を確認したという意味ではない。

`apk/apk-completed.json`과 `apk/verification/`에 원본 완료·파일 검증 근거가 있다. root도 APK 실물 SHA·크기, evidence 9개와 현재 소스 138개를 직접 대조해 `root-apk-verified.json`에 기록했다. 설치 후 두 계정 초대·수신함·v2 푸시·미리보기·참여는 사용자 수용 확인 대기로 유지한다. 앞 절의 APK 대기는 이 완료 기록 이전 상태이며, 서버·웹 운영 완료 기록은 각각 별도다.

`apk/apk-completed.json` と `apk/verification/` に元の完了・ファイル検証根拠がある。root も APK 実物の SHA・サイズ、evidence 9 件と現在ソース 138 件を直接照合し、`root-apk-verified.json` に記録した。インストール後の 2 アカウント招待・受信箱・v2 プッシュ・プレビュー・参加は、ユーザー受け入れ確認待ちを維持する。前節の APK 待機はこの完了記録前の状態で、サーバー・Web の本番完了記録はそれぞれ別である。

## 사용자 총괄 확인 보고 · 2026-10-22 15:50 KST

## ユーザーの総括確認報告 · 2026-10-22 15:50 KST

새 95ab7e46 APK와 웹 방장 위임·ID 초대·수신함·푸시·미리보기·참여의 확인 순서를 안내한 뒤 사용자가 “좋아! 확인 완료했엄 다음은 어떤 작업하면 될까?”라고 보고했다. 이번 안내 범위의 **USER_REPORTED_CONFIRMED(사용자 총괄 확인)**로 기록한다. 기록 기준 시각은 `2026-10-22T06:50:27Z`다.

新しい 95ab7e46 APK と Web のオーナー委任・ID 招待・受信箱・プッシュ・プレビュー・参加の確認順序を案内後、ユーザーは『いいね、確認できた。次は何の作業がいい？』と報告した。今回の案内範囲の **USER_REPORTED_CONFIRMED（ユーザーによる総括確認）** として記録する。記録基準時刻は `2026-10-22T06:50:27Z`。

개별 단계의 실행 로그·설치 바이너리 확인·iPhone 검증을 새로 받은 것은 아니다. 앞 절의 대기 상태와 이전 `ASSUMED_BY_USER`는 당시 이력으로 보존하며, 이번 보고를 OPEN 관리 전체·양 OS 전체 수락으로 확대하지 않는다. #54·#45의 기존 진행 댓글에 이 보고를 덧붙였고 이슈 본문·수동 체크·OPEN 상태·프로젝트 상태는 변경하지 않는다.

個々の手順の実行ログ・インストール済みバイナリ確認・iPhone 検証を新たに受け取ったわけではない。前節の待機状態と以前の `ASSUMED_BY_USER` は当時の履歴として保持し、この報告を OPEN 管理全体・両 OS 全体の受け入れへ広げない。#54・#45 の既存進行コメントへこの報告を追記し、issue 本文・手動チェック・OPEN 状態・project 状態は変更しない。

이번 수용 기록은 `.codex-workspace/verification/shared/mobile-next-phase-after-id-invitations-20261022/`에 보존한다. 새 제품 소스·시험·배포·APK·Git 작업은 실행하지 않았다.

今回の受け入れ記録は `.codex-workspace/verification/shared/mobile-next-phase-after-id-invitations-20261022/` に保存する。新しい製品ソース・試験・デプロイ・APK・Git 操作は実行していない。
