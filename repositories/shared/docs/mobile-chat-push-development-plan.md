# TranslaCat 모바일 채팅과 푸시 개발 계획

# TranslaCat モバイルチャットとプッシュの開発計画

기준일은 2026년 10월 17일이다. React Native와 Expo 및 TypeScript로 Android와 iOS 앱을 함께 개발하고, 기존 Next.js 웹과 BE 및 CHAT 서비스를 계속 사용한다. 첫 공개 버전에 1대1 채팅, 친구 그룹 채팅, 오픈채팅, 기존 채팅 번역, 휴대폰 푸시를 포함한다. 언어학습, 독립 음성 번역, 가계부의 모바일 화면은 후속 버전으로 둔다.

基準日は 2026 年 10 月 17 日である。React Native・Expo・TypeScript を用いて Android と iOS のアプリを一緒に開発し、既存の Next.js Web と BE・CHAT サービスを引き続き利用する。最初の公開バージョンには、1 対 1 チャット、友達のグループチャット、オープンチャット、既存のチャット翻訳、携帯端末へのプッシュを含める。言語学習、独立した音声翻訳、家計簿のモバイル画面は後続バージョンとする。

첫 성공 기준은 웹에서 보낸 메시지가 상대 휴대폰의 잠금 화면에 알림으로 표시되고, 알림을 누르면 해당 방에서 답장할 수 있으며, 답장과 읽음 상태가 Android와 iPhone 및 웹에서 일치하는 것이다. 푸시는 메시지 저장소가 아니므로 누락된 대화는 서버 조회로 복구한다.

最初の成功基準は、Web から送ったメッセージが相手の携帯端末のロック画面に通知され、通知を押すと該当ルームで返信でき、返信と既読状態が Android・iPhone・Web で一致することである。プッシュはメッセージ保存先ではないため、欠落した会話はサーバー取得で復旧する。

이 문서는 구현 순서와 완료 기준을 정한다. 아래 신규 API와 테이블 이름은 설계 초안이며, 현재 구현된 계약과 구분한다. 앱 저장소 생성, 패키지 설치, DB 변경, 운영 배포와 스토어 등록은 개발 실행 단계에서 수행한다.

本書は実装順序と完了基準を定める。以下の新規 API・テーブル名は設計草案であり、現在実装されている契約とは区別する。アプリ用リポジトリの作成、パッケージのインストール、DB 変更、本番デプロイ、ストア登録は開発の実行段階で行う。

실행 중 사용자는 Apple Developer 유료 등록을 핵심 기능 준비 후로 미루기로 했다. 무료 Expo·Firebase 준비와 앱·서버 개발은 먼저 진행하고, iPhone의 실제 설치·푸시 검증 전에 유료 등록을 완료한다. Android·iOS 동시 출시 목표와 실기기 검증 기준은 유지한다. 계정별 현재 상태와 직접 할 일은 [모바일 계정 준비 절차](../../mobile/docs/phase0-account-setup.md)를 따른다.

実行中、ユーザーは Apple Developer の有料登録を主要機能の準備後に延期することにした。無料の Expo・Firebase の準備とアプリ・サーバー開発を先に進め、iPhone への実インストール・プッシュ検証前に有料登録を完了する。Android・iOS 同時リリースの目標と実機検証基準は維持する。アカウント別の現在状態と本人が行う作業は、[モバイルアカウント準備手順](../../mobile/docs/phase0-account-setup.md)に従う。

## 첫 출시 범위

## 初回リリースの範囲

| 영역 | 첫 공개 버전에 포함할 기능 |
| --- | --- |
| 계정 | 기존 이메일 및 Google 로그인, 가입 및 이메일 확인, 기기별 로그인 유지와 로그아웃, Apple 로그인, 인증 수단 연결, 계정 삭제 |
| 친구 | ID 검색, 친구 요청과 수락 및 거절, 친구 목록, 1대1 대화 시작, 친구 그룹 생성과 초대 |
| 공통 채팅 | 방 목록과 최근 메시지 요약, 텍스트 송수신, 과거 메시지, 번역 표시와 재시도, 언어 설정, 읽음과 안 읽은 개수 |
| 오픈채팅 | 공개방 탐색과 검색, 공개 및 링크 전용 방 생성, 참여와 나가기, 방별 프로필과 이미지, 공유 링크, 멤버 및 역할 관리, 강퇴와 해제, 방장 위임과 방 종료 |
| 푸시 | Android와 iOS 수신, 권한 안내, 전역 및 방별 설정, 알림에서 방과 메시지 열기, 로그아웃 및 계정 전환 시 이전 연결 해제 |
| 사용자 보호 | 메시지와 사용자 신고, 차단, 부적절한 콘텐츠 처리, 운영자 처리 화면, 문의 연락처, 이용약관과 개인정보 안내 |
| 표시 품질 | 기존 AI 및 SYSTEM 메시지 표시, 한국어와 일본어 중심의 기존 채팅 번역 문구, 글자 확대, 스크린리더와 키보드 대응 |

| 領域 | 最初の公開バージョンに含める機能 |
| --- | --- |
| アカウント | 既存メール・Google ログイン、登録・メール確認、端末別のログイン維持・ログアウト、Apple ログイン、認証手段の連携、アカウント削除 |
| 友達 | ID 検索、友達申請・承認・拒否、友達一覧、1 対 1 会話の開始、友達グループの作成・招待 |
| 共通チャット | ルーム一覧・最近のメッセージ要約、テキスト送受信、過去メッセージ、翻訳表示・再試行、言語設定、既読・未読数 |
| オープンチャット | 公開ルームの探索・検索、公開・リンク限定ルーム作成、参加・退室、ルーム別プロフィール・画像、共有リンク、メンバー・役割管理、強制退室・解除、オーナー委任・ルーム終了 |
| プッシュ | Android・iOS 受信、権限案内、全体・ルーム別設定、通知からルーム・メッセージを開く、ログアウト・アカウント切り替え時の以前の接続解除 |
| ユーザー保護 | メッセージ・ユーザーの通報、ブロック、不適切なコンテンツの処理、運営者の処理画面、問い合わせ先、利用規約・プライバシー案内 |
| 表示品質 | 既存 AI・SYSTEM メッセージ表示、韓国語・日本語を中心とする既存チャット翻訳文言、文字拡大、スクリーンリーダー・キーボード対応 |

채팅 이미지·동영상·파일 첨부, 음성 메시지와 통화, 무제한 오프라인 자동 전송, AI 멤버 설정 편집, 학습과 가계부 화면은 후속 범위다. 오픈 프로필 이미지는 첫 출시 범위이며, 채팅 첨부파일과 구분한다.

チャット画像・動画・ファイル添付、音声メッセージ・通話、無制限のオフライン自動送信、AI メンバー設定編集、学習・家計簿画面は後続範囲である。オープンプロフィール画像は初回リリース範囲であり、チャット添付ファイルとは区別する。

## 현재 구현과 선행 보강

## 現在の実装と先行補強

| 현재 소스에서 확인한 내용 | 개발 계획에 반영할 작업 |
| --- | --- |
| BE refresh_token은 userId가 기본 키이며 로그인마다 같은 행을 덮어쓴다. 로그아웃도 userId로 삭제한다. | 웹과 여러 휴대폰이 함께 로그인하도록 세션별 토큰 저장과 폐기로 전환한다. |
| Google 및 자체 로그인과 명시적 계정 연결은 있지만 Apple 제공자는 없다. | 기존 사용자 ID를 유지하며 복수 인증 수단과 Apple 로그인을 추가한다. |
| 공개 REST와 STOMP 진입점은 BE이고 실제 채팅 상태와 권한은 CHAT가 소유한다. | 앱도 같은 공개 진입점을 사용한다. 내부 서비스나 DB에 직접 연결하지 않는다. |
| 메시지 저장 후 메모리에 등록한 이벤트를 전달하며 실패 시 로그를 남긴다. | 푸시 작업은 메시지와 같은 DB 트랜잭션에 영속 저장하고 별도 작업자가 재시도한다. |
| 전송 요청은 content뿐이며 STOMP publish 성공은 저장 확정 응답이 아니다. | 앱은 REST 저장 응답으로 전송 성공을 판정하고 선택적 clientMessageId를 추가한다. |
| 이전 페이지와 after 및 anchor 조회, 서버 읽음 커서가 있다. | 앱 복귀와 알림 진입에 재사용한다. 최신 페이지 하나만 합치는 방식으로 복구를 끝내지 않는다. |
| 기기 토큰, 방별 푸시 설정과 전송 작업은 확인되지 않았다. | BE 기기 등록과 CHAT 푸시 정책 및 작업자를 새로 추가한다. |
| 기존 방 목록에는 최근 메시지 미리보기가 없고 미읽음 알림 목록에만 요약이 있다. | 전체 방 목록에 선택적 최근 메시지 요약을 추가한다. 미읽음 방 목록을 전체 목록으로 대신하지 않는다. |
| 문의의 relatedMessageId는 support_message를 참조한다. | 전용 채팅 신고와 증거 및 운영 처리 계약을 추가한다. 문의 쪽지 ID 자리에 채팅 ID를 넣지 않는다. |
| 조사한 BE 사용자 및 인증 코드에는 계정 삭제 흐름이 없다. | 계정 전체 삭제와 서비스별 데이터 정리 범위를 출시 선행 작업으로 확정한다. |

| 現在のソースで確認した内容 | 開発計画へ反映する作業 |
| --- | --- |
| BE の refresh_token は userId が主キーで、ログインのたびに同じ行を上書きする。ログアウトも userId で削除する。 | Web と複数端末で同時ログインできるよう、セッション別トークンの保存・破棄へ移行する。 |
| Google・独自ログインと明示的アカウント連携はあるが、Apple プロバイダーはない。 | 既存ユーザー ID を維持し、複数の認証手段と Apple ログインを追加する。 |
| 公開 REST と STOMP の入口は BE、実際のチャット状態と権限は CHAT が所有する。 | アプリも同じ公開入口を使用する。内部サービス・DB へ直接接続しない。 |
| メッセージ保存後、メモリへ登録したイベントを配信し、失敗時はログを残す。 | プッシュ作業をメッセージと同じ DB トランザクションへ永続保存し、別 worker が再試行する。 |
| 送信要求は content のみで、STOMP publish 成功は保存確定応答ではない。 | アプリは REST 保存応答で送信成功を判断し、任意の clientMessageId を追加する。 |
| 前ページ・after・anchor の取得とサーバー既読 cursor がある。 | アプリ復帰と通知からの遷移へ再利用する。最新の 1 ページだけを結合して復旧を終えない。 |
| 端末トークン、ルーム別プッシュ設定、送信作業は確認されなかった。 | BE の端末登録と CHAT のプッシュポリシー・worker を新規追加する。 |
| 既存のルーム一覧には最近のメッセージプレビューがなく、未読通知一覧だけに要約がある。 | 全ルーム一覧に任意の最近のメッセージ要約を追加する。未読ルーム一覧を全体一覧の代用にしない。 |
| 問い合わせの relatedMessageId は support_message を参照する。 | 専用チャット通報・証拠・運営処理の契約を追加する。問い合わせメッセージ ID の欄へチャット ID を入れない。 |
| 調査した BE ユーザー・認証コードにアカウント削除フローはない。 | アカウント全体の削除とサービス別データ整理の範囲をリリースの先行作業として確定する。 |

주요 소스는 문서 끝의 로컬 근거에서 확인한다. BE와 FE에는 다른 작업의 기존 변경이 있으므로 구현을 시작할 때 상태와 관련 diff를 다시 확인한다.

主なソースは文末のローカル根拠で確認する。BE・FE には別作業の既存変更があるため、実装開始時に状態と関連 diff を再確認する。

## 저장소와 서비스 책임

## リポジトリとサービスの責任

신규 앱 저장소의 가칭은 CatPjt-TranslaCat-mobile이다. 기존 저장소들을 모노레포로 옮기는 작업은 출시 선행 조건으로 삼지 않는다. 새 저장소를 실제로 만들 때 모바일 지침과 검증 명령도 함께 정의한다.

新しいアプリ用リポジトリの仮称は CatPjt-TranslaCat-mobile である。既存リポジトリをモノレポへ移すことはリリースの前提にしない。新リポジトリを実際に作成するとき、モバイル指針と検証コマンドも合わせて定義する。

| 대상 | 책임과 변경 범위 |
| --- | --- |
| Mobile 신규 | Expo 앱, 네이티브 화면, 기기 인증 adapter, STOMP 수신, 복구, 푸시 등록과 클릭 처리 |
| FE 기존 | 기존 웹 유지, 세션 전환 호환, Apple 인증 연결 및 계정 삭제 웹 경로, 필요 시 신고·알림 설정 UI 보완 |
| BE 기존 | 계정과 인증 세션, Google 및 Apple 증명 검증, 기기 등록과 소유권, 공개 gateway, 계정 제재 및 삭제 조정 |
| CHAT 기존 | 메시지 및 읽음과 방 권한, 신고 증거와 채팅 제재, 알림 설정, 푸시 outbox와 작업자, Expo 전송 adapter |
| LL 및 사용자 데이터 소유 서비스 | 계정 삭제에 필요한 데이터와 진행 작업 정리 계약을 조사한다. 필요한 수정은 별도 작업으로 산정한다. |
| Docs | 모바일 계약, 검증 결과와 출시 운영 기준을 중앙 문서에 유지 |

| 対象 | 責任と変更範囲 |
| --- | --- |
| Mobile 新規 | Expo アプリ、ネイティブ画面、端末認証 adapter、STOMP 受信、復旧、プッシュ登録・タップ処理 |
| FE 既存 | 既存 Web 維持、セッション移行互換性、Apple 認証連携・アカウント削除の Web 経路、必要な通報・通知設定 UI の補完 |
| BE 既存 | アカウント・認証セッション、Google・Apple 証明の検証、端末登録・所有権、公開 gateway、アカウント制裁・削除の調整 |
| CHAT 既存 | メッセージ・既読・ルーム権限、通報証拠・チャット制裁、通知設定、プッシュ outbox・worker、Expo 送信 adapter |
| LL とユーザーデータ所有サービス | アカウント削除に必要なデータと進行中作業の整理契約を調査する。必要な修正は別作業として見積もる。 |
| Docs | モバイル契約・検証結果・リリース運用基準を中央文書で維持する |

~~~mermaid
flowchart LR
    Web[기존 Next.js 웹] --> BE[기존 BE 공개 API와 인증]
    App[Android와 iOS Expo 앱] --> BE
    BE --> Chat[기존 CHAT 메시지와 읽음]
    Chat --> DB[(메시지와 푸시 outbox)]
    DB --> Worker[CHAT 푸시 작업자]
    Worker -->|현재 기기와 세션 조회| Devices[BE 기기 등록과 인증 세션]
    Worker --> Expo[Expo Push Service]
    Expo --> FCM[FCM Android]
    Expo --> APNs[APNs iOS]
    FCM --> App
    APNs --> App
~~~

上図のコードは変更せず保持する。既存 Next.js Web と Android・iOS の Expo アプリは、既存 BE の公開 API・認証へ接続する。BE は既存 CHAT のメッセージ・既読へ接続し、CHAT はメッセージとプッシュ outbox の DB へ記録する。DB から CHAT のプッシュ worker が作業を取得し、現在の端末・セッションを BE の端末登録・認証セッションへ問い合わせる。worker は Expo Push Service へ送り、そこから FCM（Android）・APNs（iOS）を通じてアプリへ配信する。

초기에는 별도 알림 서버를 만들지 않는다. BE가 계정과 기기 소유권을 관리하고, CHAT가 메시지 수신 정책과 푸시 작업자를 소유한다. CHAT는 인증된 내부 BE API로 현재 유효한 기기만 조회한다. 기존 Core 통신 패턴에 push 전용 목적과 scope를 추가하고 BE의 경로 및 scope 허용 목록을 함께 갱신한다. 일반 사용자나 기존 identity scope로 기기 토큰을 조회하지 못하게 한다. 무효 토큰 처리도 같은 내부 경계로 BE에 반영한다. 추후 다른 기능에서 푸시가 필요해지면 이 전송 port를 공통화할 수 있다.

初期は独立した通知サーバーを作らない。BE がアカウント・端末所有権を管理し、CHAT がメッセージ受信ポリシーとプッシュ worker を所有する。CHAT は認証済みの内部 BE API で現在有効な端末だけを取得する。既存 Core 通信パターンに push 専用の目的と scope を追加し、BE の経路・scope 許可リストも一緒に更新する。一般ユーザーや既存 identity scope では端末トークンを取得できないようにする。無効トークンの処理も同じ内部境界を通じて BE へ反映する。今後ほかの機能でプッシュが必要になれば、この送信 port を共通化できる。

## 모바일 코드 구성

## モバイルのコード構成

~~~text
app/
  (auth)/
  (tabs)/chats.tsx
  (tabs)/friends.tsx
  (tabs)/open.tsx
  (tabs)/settings.tsx
  chat/[roomId].tsx
  open/
src/
  api/
  auth/
  realtime/
  notifications/
  domain/chat/
  features/chat/
  features/friends/
  features/open-chat/
  features/settings/
  storage/
  i18n/
  ui/
~~~

上のディレクトリ構成はコードとして原文のまま保持する。app には認証、チャット・友達・オープンチャット・設定のタブ、ルーム別チャットと open 画面を置く。src は API、認証、リアルタイム接続、通知、チャットドメイン、チャット・友達・オープンチャット・設定の各 feature、ストレージ、国際化、共通 UI に分ける。

Expo Router를 화면 이동에, TanStack Query를 서버 조회 상태에 사용한다. 기존 STOMP 계약은 @stomp/stompjs와 네이티브 WebSocket으로 연결 검증한다. 푸시는 expo-notifications, 인증 비밀 저장은 expo-secure-store를 사용한다. 구체적인 Expo SDK와 라이브러리 버전은 첫 구현 시 호환 조합을 확인하고 lockfile로 고정한다.

Expo Router を画面遷移、TanStack Query をサーバー取得状態の管理に使う。既存 STOMP 契約は @stomp/stompjs とネイティブ WebSocket で接続検証する。プッシュは expo-notifications、認証の秘密情報の保存は expo-secure-store を使う。具体的な Expo SDK・ライブラリのバージョンは初回実装時に互換性のある組み合わせを確認し、lockfile で固定する。

앱 전체에서 실시간 연결의 소유자를 하나로 둔다. 메시지 전송은 REST를 우선하고, 수신은 STOMP를 사용한다. AppState와 화면 focus 및 메시지 노출 여부를 함께 판단하여 읽음을 전진시킨다. 백그라운드에서 소켓이 계속 살아 있다고 가정하지 않는다.

アプリ全体でリアルタイム接続の所有者を 1 つにする。メッセージ送信は REST を優先し、受信は STOMP を使う。AppState・画面 focus・メッセージ露出を合わせて判断して既読を進める。バックグラウンドでソケットが生存し続けるとは仮定しない。

웹에서 재사용할 것은 순수 DTO, 이벤트 parser, 언어 규칙, 정렬과 중복 병합 및 번역 병합 규칙이다. NextAuth 세션, DOM 화면, Next 라우팅과 웹 hooks는 모바일 adapter와 화면으로 작성한다. File이 들어 있는 웹 폼 타입은 API DTO와 분리한다. 공통 패키지 배포부터 시작하지 않고 필요한 순수 모듈부터 계약 테스트와 함께 공유한다.

Web から再利用するのは純粋な DTO、イベント parser、言語ルール、並び替え・重複結合・翻訳結合のルールである。NextAuth セッション、DOM 画面、Next routing、Web hooks はモバイル adapter と画面として作成する。File を含む Web フォーム型は API DTO と分離する。共通パッケージの配布から始めず、必要な純粋モジュールから契約テストとともに共有する。

## 인증과 계정 개발

## 認証とアカウントの開発

### 여러 기기에서 로그인 유지

### 複数端末でログインを維持

BE에 auth_session을 추가하는 안으로 시작한다. sessionId, userId, platform, refreshTokenHash, 생성과 만료 시각, revokedAt을 관리하고 access 및 refresh 토큰에 sid를 연결한다. refresh token 원문을 새 테이블에 저장하지 않는다.

BE に auth_session を追加する案から始める。sessionId、userId、platform、refreshTokenHash、作成・有効期限、revokedAt を管理し、access・refresh トークンに sid を結び付ける。refresh token の原文を新テーブルへ保存しない。

현재 기기 로그아웃과 전체 로그아웃을 구분한다. 기기 로그아웃은 해당 세션과 push installation만 폐기하며 다른 웹 및 앱 세션을 유지한다. 삭제·제재·전체 로그아웃은 모든 대상 세션과 연결 및 푸시를 차단한다. JWT 서명 확인만으로 끝내지 않고 BE와 WebSocket 경계에서 세션의 유효성과 만료를 확인한다.

現在端末のログアウトと全体ログアウトを区別する。端末ログアウトは当該セッションと push installation だけを破棄し、他の Web・アプリのセッションは維持する。削除・制裁・全体ログアウトは、対象の全セッション・接続・プッシュを遮断する。JWT 署名確認だけで終えず、BE・WebSocket 境界でセッションの有効性と期限を確認する。

현재 CHAT의 내부 identity 계약은 요청과 응답 필드를 엄격하게 검증한다. sid 및 계정 상태를 전달할 때 기존 계약에 필드만 덧붙이지 않고 버전 경로나 별도 세션 검증 계약을 둔다. BE 검증과 CHAT parser 및 gateway를 함께 배포하고 기존 웹 경로의 호환성을 확인한다.

現在の CHAT 内部 identity 契約は、要求・応答のフィールドを厳格に検証する。sid・アカウント状態を渡す際、既存契約へフィールドを足すだけにせず、バージョン経路または別のセッション検証契約を設ける。BE の検証と CHAT parser・gateway を一緒にデプロイし、既存 Web 経路との互換性を確認する。

기존 웹 토큰은 제한된 호환 기간에 sid가 있는 새 세션으로 전환하는 절차를 둔다. 새 로그인은 세션 행에만 저장하고 기존 사용자당 refresh 행을 덮어쓰지 않는다. 기존 refresh와 sid 토큰의 검증·로그아웃 동작을 구분하고 유예 종료를 명시한다. 토큰 수명 변경과 refresh rotation을 함께 도입한다면 NextAuth의 동시 갱신 경합까지 검증한다.

既存 Web トークンは、限定した互換期間内に sid 付きの新セッションへ移行する手順を設ける。新規ログインはセッション行だけに保存し、既存ユーザー別 refresh 行を上書きしない。既存 refresh と sid トークンの検証・ログアウト動作を区別し、猶予期間の終了を明示する。トークン寿命変更と refresh rotation を一緒に導入するなら、NextAuth の同時更新競合も検証する。

앱은 refresh token을 SecureStore에, access token은 메모리 중심으로 보관한다. 갱신 요청을 한 번만 실행하고, 일시적 통신 실패와 확정된 인증 거절을 구분한다. 계정이 바뀌면 이전 요청과 연결, 메시지 cache와 푸시 진입 대기를 정리한다. 재설치 시 저장소 동작 차이에 의존하지 않고 설치 식별자와 서버 세션을 다시 확인한다. [SecureStore 문서](https://docs.expo.dev/versions/latest/sdk/securestore/)

アプリは refresh token を SecureStore、access token はメモリを中心に保存する。更新要求は 1 回だけ実行し、一時的な通信失敗と確定した認証拒否を区別する。アカウント変更時は以前の要求・接続・メッセージ cache・プッシュからの遷移待ちを整理する。再インストール時のストレージ動作差に依存せず、インストール識別子とサーバーセッションを再確認する。[SecureStore 文書](https://docs.expo.dev/versions/latest/sdk/securestore/)

### Google과 Apple 로그인

### Google・Apple ログイン

이메일과 Google 로그인은 기존 BE 계약을 활용한다. Google 네이티브 SDK에서 얻은 증명을 BE가 검증하고 자체 토큰을 발급한다. 현재 audience 설정 하나를 임의로 완화하지 않고, serverClientId 사용 또는 명시적 플랫폼별 allowlist로 구성한다. client secret과 서버용 키는 앱에 넣지 않는다.

メール・Google ログインは既存 BE 契約を利用する。Google ネイティブ SDK で得た証明を BE が検証し、独自トークンを発行する。現在の単一 audience 設定を任意に緩和せず、serverClientId の使用または明示的なプラットフォーム別 allowlist で構成する。client secret とサーバー用キーはアプリに含めない。

Google 인증 라이브러리는 Android Credential Manager 지원과 현재 Expo 호환성을 확인하여 선택한다. 네이티브 로그인 및 푸시는 development build부터 검증한다. [Expo Google 인증 가이드](https://docs.expo.dev/guides/google-authentication/)

Google 認証ライブラリは Android Credential Manager 対応と現在の Expo 互換性を確認して選定する。ネイティブログイン・プッシュは development build から検証する。[Expo Google 認証ガイド](https://docs.expo.dev/guides/google-authentication/)

iOS에서 Google 로그인을 제공하므로 App Review 4.8에 대응하는 동등한 로그인 수단으로 Sign in with Apple을 포함한다. iOS는 네이티브 인증, Android와 웹은 Apple의 브라우저 인증 및 서버 callback으로 같은 계정 접근을 지원하는 안으로 잡는다. 브라우저 callback은 검증된 일회용 코드만 앱에 넘기고 장기 토큰을 URL에 넣지 않는다. Android와 웹의 Services ID를 기본 앱 ID에 연결하고 플랫폼별 audience 및 동일 계정 식별을 검증한다. [Apple 로그인 검토 기준](https://developer.apple.com/app-store/review/guidelines/#login-services) · [Apple 웹 인증 설정](https://developer.apple.com/help/account/capabilities/configure-sign-in-with-apple-for-the-web/)

iOS で Google ログインを提供するため、App Review 4.8 に対応する同等のログイン手段として Sign in with Apple を含める。iOS はネイティブ認証、Android・Web は Apple のブラウザー認証・サーバー callback で同じアカウントへアクセスする案とする。ブラウザー callback は検証済みの使い捨てコードだけをアプリへ渡し、長期トークンを URL に入れない。Android・Web の Services ID を基本アプリ ID へ結び付け、プラットフォーム別 audience と同一アカウント識別を検証する。[Apple ログイン審査基準](https://developer.apple.com/app-store/review/guidelines/#login-services)・[Apple Web 認証設定](https://developer.apple.com/help/account/capabilities/configure-sign-in-with-apple-for-the-web/)

인증 제공자별 고유 subject를 user_identity로 연결하고 기존 Google 연결을 보존한다. 사용자 id와 publicId 및 기존 대화는 유지한다. 이메일이 같다는 이유로 계정을 자동 합치지 않는다. 기존 계정으로 먼저 로그인한 뒤 재인증하여 Apple을 연결한다. Apple 비공개 이메일과 최초 인증에서만 제공되는 이름, callback 취소와 재사용을 검증한다.

認証プロバイダーごとの一意な subject を user_identity へ結び付け、既存 Google 連携を維持する。ユーザー id・publicId・既存会話は保持する。メールが同じという理由でアカウントを自動統合しない。既存アカウントへ先にログインし、再認証してから Apple を連携する。Apple の非公開メール・初回認証のみで提供される名前・callback のキャンセルと再使用を検証する。

## 채팅과 오픈채팅 개발

## チャット・オープンチャットの開発

### 메시지와 복구

### メッセージと復旧

1. 기존 REST 생성 요청에 선택적 clientMessageId를 추가한다. 서버는 송신자·방·키에 유일성을 보장하고 같은 키의 동일 요청에 기존 저장 결과를 반환한다. 같은 키로 내용이 달라지면 충돌을 반환한다.
2. 앱은 보내는 중, 저장됨, 실패 상태를 구분한다. 응답 유실 후에는 같은 키로만 재시도하며 REST 응답과 STOMP 이벤트를 서버 messageId 및 clientMessageId로 병합한다. 이전 웹 요청은 키 없이 계속 동작한다.
3. background 복귀와 네트워크 재연결 후 마지막 확인 위치부터 after 페이지를 조회하여 끝까지 따라간다. 동시에 도착한 STOMP 이벤트와 중복 제거한다. 번역, 읽음, 멤버 및 방 상태도 다시 조회한다.
4. 알림의 messageId는 anchor 조회에 사용한다. 삭제되었거나 권한이 없으면 방 목록 등 안전한 화면으로 이동한다. 푸시 payload로 멤버십을 인정하지 않는다.
5. 읽음은 앱이 활성 상태이고 해당 방이 focus이며 실제 메시지가 노출될 때만 전진한다. 푸시 수신과 background fetch만으로 읽음 처리하지 않는다.
6. 방 목록에는 최근 메시지와 첫 미읽음 위치를 선택적 요약 필드로 추가한다. 기존 읽음 및 attention 집계 규칙을 유지한다.

1. 既存 REST 作成要求へ任意の clientMessageId を追加する。サーバーは送信者・ルーム・キーの一意性を保証し、同じキーの同一要求には既存保存結果を返す。同じキーで内容が違えば競合を返す。
2. アプリは送信中・保存済み・失敗を区別する。応答喪失後は同じキーだけで再試行し、REST 応答と STOMP イベントをサーバー messageId・clientMessageId で結合する。以前の Web 要求はキーなしで引き続き動作する。
3. background 復帰・ネットワーク再接続後、最後に確認した位置から after ページを取得して最後まで追う。同時到着した STOMP イベントと重複排除する。翻訳・既読・メンバー・ルーム状態も再取得する。
4. 通知の messageId は anchor 取得に使う。削除済み・権限なしならルーム一覧など安全な画面へ移動する。プッシュ payload によってメンバーシップを認めない。
5. 既読はアプリがアクティブ、該当ルームが focus、実際にメッセージが露出しているときだけ進める。プッシュ受信・background fetch だけで既読にしない。
6. ルーム一覧へ、最近のメッセージと最初の未読位置を任意の要約フィールドとして追加する。既存の既読・attention 集計ルールを維持する。

기존 숫자 ID와 cursor의 안전 범위를 확인한다. JSON 숫자를 파싱한 뒤 문자열로 바꾸는 것만으로 이미 잃은 정밀도가 복구되지는 않는다. 64비트 전체 범위가 필요하면 서버 직렬화와 웹 호환을 포함한 명시적 문자열 ID 계약을 추가한다. 정렬 기준은 현재 서버 메시지 순서를 보존하고 시각의 timezone을 확인한다.

既存の数値 ID・cursor の安全範囲を確認する。JSON 数値を parse 後に文字列へ変えるだけでは、すでに失った精度は復旧しない。64 ビット全範囲が必要なら、サーバー直列化・Web 互換性を含む明示的な文字列 ID 契約を追加する。並び順は現在のサーバーメッセージ順を維持し、時刻の timezone を確認する。

### 오픈채팅

### オープンチャット

공개 및 링크 전용 방의 탐색·생성·참여·프로필·멤버·관리 API는 유지하고 앱 화면과 네이티브 이미지 및 공유 adapter를 작성한다. OWNER와 ADMIN의 대상별 관리 권한을 서버와 동일하게 표시하며, 강퇴·역할 변경·방 종료 이벤트를 즉시 화면에 반영한다.

公開・リンク限定ルームの探索・作成・参加・プロフィール・メンバー・管理 API は維持し、アプリ画面とネイティブ画像・共有 adapter を作成する。OWNER・ADMIN の対象別管理権限をサーバーと同様に表示し、強制退室・役割変更・ルーム終了イベントを即座に画面へ反映する。

방별 닉네임과 이미지를 사용하며 일반 계정 publicId, 이메일과 친구 관계를 노출하지 않는다. 신고와 차단은 roomId 및 openChatMemberId를 받고 서버가 실제 계정을 해석하는 새 계약을 둔다. 익명 프로필을 이유로 내부 책임 계정까지 없애지 않는다.

ルーム別のニックネーム・画像を使い、一般アカウントの publicId・メール・友達関係は露出させない。通報・ブロックは roomId・openChatMemberId を受け、サーバーが実アカウントを解釈する新契約を設ける。匿名プロフィールを理由に、内部で責任を持つアカウントまでなくさない。

재참여 전 메시지 접근 제한을 보존하고 membership 경계가 바뀌면 이전 cache를 비운다. iPhone 이미지가 기존 JPG·PNG·WEBP 계약에 맞지 않으면 업로드 전에 변환하거나 형식 안내를 제공한다. 방 참여 후 이미지만 실패했다면 이미지만 재시도한다.

再参加前のメッセージアクセス制限を維持し、membership 境界が変わったら以前の cache を消去する。iPhone 画像が既存 JPG・PNG・WEBP 契約に合わなければ、アップロード前に変換するか形式を案内する。ルーム参加後に画像だけ失敗した場合は画像だけを再試行する。

차단은 기존 친구 관계 차단과 메시지·푸시 차단을 구분하여 명세한다. 출시안에서는 차단 대상의 새 메시지 표시와 푸시를 억제하고, 기존 대화와 운영 증거를 일괄 삭제하지 않는다. OPEN 요청과 응답은 실제 계정 식별자를 노출하지 않아야 한다.

ブロックは既存の友達関係ブロックとメッセージ・プッシュのブロックを区別して仕様化する。リリース案では対象の新規メッセージ表示・プッシュを抑え、既存会話・運営証拠を一括削除しない。OPEN の要求・応答は実アカウント識別子を露出させてはならない。

## 푸시 알림 개발

## プッシュ通知の開発

### 발송과 기기 관리

### 送信と端末管理

CHAT는 메시지가 저장되는 트랜잭션 안에 push outbox를 기록한다. worker가 재시작되어도 작업을 찾도록 상태와 lease, 다음 시도 시각, 시도 횟수 및 만료를 관리한다. 작업 재시도나 동일 메시지 재전송이 별도 이벤트를 계속 만들지 않도록 eventId와 수신자 및 installation 기준 중복을 방지한다.

CHAT はメッセージが保存されるトランザクション内で push outbox を記録する。worker の再起動後も作業を見つけられるよう、状態・lease・次回試行時刻・試行回数・期限を管理する。作業再試行や同じメッセージの再送で別イベントが繰り返し作られないよう、eventId・受信者・installation を基準に重複を防ぐ。

OPEN 공개 메시지 DTO는 실제 송신 계정 ID를 숨기므로 outbox를 공개 DTO만으로 만들지 않는다. 내부 저장 모델의 송신자 ID로 본인 제외와 차단을 판단하고, 외부 payload와 로그에는 이를 노출하지 않는다. AI 메시지 알림을 지원할 경우 별도 AI 메시지 저장 트랜잭션에도 같은 outbox 기록을 적용한다.

OPEN の公開メッセージ DTO は実送信アカウント ID を隠すため、公開 DTO だけで outbox を作らない。内部保存モデルの送信者 ID で本人除外・ブロックを判断し、外部 payload・ログへは露出させない。AI メッセージ通知に対応する場合は、別の AI メッセージ保存トランザクションにも同じ outbox 記録を適用する。

worker는 수신자를 선정한 뒤 발송 직전에 방 상태·활성 멤버·가입 시점·차단·읽음·음소거를 재확인한다. BE에서 현재 세션 및 installation의 유효성과 소유권 및 만료를 확인하고 기기 토큰을 조회한다. 내부 응답은 installationId, sid, binding generation, tokenVersion과 provider 및 환경 정보를 포함한다. 재시도마다 다시 조회하고, 무효 토큰은 그 토큰과 등록 generation이 여전히 일치하는 경우에만 비활성화하여 새로 교체된 토큰을 지우지 않는다.

worker は受信者の選定後、送信直前にルーム状態・アクティブメンバー・加入時点・ブロック・既読・ミュートを再確認する。BE で現在セッション・installation の有効性・所有権・期限を確認し、端末トークンを取得する。内部応答には installationId・sid・binding generation・tokenVersion・provider・環境情報を含める。再試行ごとに再取得し、無効トークンは当該トークンと登録 generation が依然一致する場合だけ無効化し、新しく交換されたトークンを消さない。

Expo·FCM·APNs 및 전송 네트워크 장애는 메시지 저장과 분리한다. 반면 필수 outbox 행의 DB 기록이 실패하면 메시지와 함께 rollback하여 저장 원자성을 유지한다.

Expo・FCM・APNs・送信ネットワークの障害はメッセージ保存から分離する。一方、必須 outbox 行の DB 記録が失敗したらメッセージと一緒に rollback し、保存の原子性を維持する。

초기 전송 provider는 Expo Push Service로 한다. 네트워크 오류와 일시 실패를 재시도하고 영구 토큰 오류를 정리한다. ticket과 receipt를 구분해 결과를 수집하며, receipt 성공을 사용자 화면 표시나 읽음 성공으로 처리하지 않는다. Expo 요청용 access token을 서버에서 관리하는 추가 보안 설정도 적용한다. [Expo 발송과 오류 처리](https://docs.expo.dev/push-notifications/sending-notifications/)

初期の送信 provider は Expo Push Service とする。ネットワークエラー・一時失敗を再試行し、恒久的なトークンエラーを整理する。ticket・receipt を区別して結果を収集し、receipt 成功をユーザー画面への表示や既読成功として扱わない。Expo 要求用 access token をサーバーで管理する追加のセキュリティ設定も適用する。[Expo 送信とエラー処理](https://docs.expo.dev/push-notifications/sending-notifications/)

Expo는 FCM과 APNs 전달을 중개한다. 첫 실기기 빌드 전에 Firebase FCM 자격과 Apple APNs 자격, EAS projectId를 연결한다. dev 및 preview와 production의 기기·토큰·자격은 구분한다. [Expo 푸시 설정](https://docs.expo.dev/push-notifications/push-notifications-setup/)

Expo は FCM・APNs 配信を仲介する。最初の実機ビルド前に Firebase FCM 資格情報・Apple APNs 資格情報・EAS projectId を接続する。dev・preview・production の端末・トークン・資格情報は区別する。[Expo プッシュ設定](https://docs.expo.dev/push-notifications/push-notifications-setup/)

### 최초 알림 정책

### 初期の通知ポリシー

| 항목 | 출시안 |
| --- | --- |
| 친구 1대1과 그룹 | 메시지 알림을 기본 ON으로 두되 OS 권한과 사용자 설정을 따른다. |
| 오픈채팅 | 기본 OFF, 참여하거나 방 설정에서 방별 ON을 선택한다. 다수 공개방의 과도한 알림을 방지한다. |
| 본인이 보낸 메시지 | 본인의 다른 기기에도 새 메시지 푸시를 보내지 않는다. |
| AI와 SYSTEM | SYSTEM과 번역 완료만으로 새 메시지 푸시를 만들지 않는다. AI 메시지는 방 설정에 따른다. |
| 웹 접속 중 | 웹 online Presence만으로 휴대폰 푸시를 끄지 않는다. |
| 같은 방을 보고 있음 | 앱은 해당 방의 foreground 알림을 중복 표시하지 않는다. 서버 억제는 해당 installation의 신선한 상태가 있을 때만 적용한다. |
| 잠금 화면 본문 | 기본은 본문 없는 새 메시지 안내다. 본문 미리보기는 사용자 선택으로 두며 OPEN 이름은 방별 프로필만 쓴다. |
| 읽음과 badge | 서버 읽음 및 unread를 기준으로 앱 내 개수를 맞춘다. OS 배지 표시는 플랫폼 지원 범위에서 갱신한다. |

| 項目 | リリース案 |
| --- | --- |
| 友達との 1 対 1・グループ | メッセージ通知は既定 ON とし、OS 権限とユーザー設定に従う。 |
| オープンチャット | 既定 OFF。参加時またはルーム設定でルーム別 ON を選ぶ。多数の公開ルームによる過剰通知を防ぐ。 |
| 本人が送ったメッセージ | 本人の別端末にも新規メッセージプッシュを送らない。 |
| AI・SYSTEM | SYSTEM や翻訳完了だけで新規メッセージプッシュを作らない。AI メッセージはルーム設定に従う。 |
| Web 接続中 | Web の online Presence だけで携帯端末のプッシュを止めない。 |
| 同じルームを閲覧中 | アプリは当該ルームの foreground 通知を重複表示しない。サーバー抑制は当該 installation の新鮮な状態がある場合だけ適用する。 |
| ロック画面の本文 | 既定は本文なしの新規メッセージ案内。本文プレビューはユーザー選択とし、OPEN の名前はルーム別プロフィールだけを使う。 |
| 既読・badge | サーバーの既読・unread を基準にアプリ内件数を合わせる。OS バッジはプラットフォームの対応範囲内で更新する。 |

payload는 schemaVersion, eventId, roomId, messageId와 불투명한 계정 바인딩 식별자 등 최소한의 라우팅 정보만 담는다. 앱은 현재 계정과 일치하는지 검사한 뒤 서버에서 권한과 내용을 조회한다. 로그아웃 중 눌린 알림은 로그인 후 재검증하고 처리한다. 문자열 URL을 그대로 실행하거나 다른 계정의 방을 자동으로 열지 않는다.

payload は schemaVersion・eventId・roomId・messageId・不透明なアカウント binding 識別子など、最小限の routing 情報だけを含める。アプリは現在のアカウントと一致するか検査後、サーバーから権限・内容を取得する。ログアウト中に押された通知はログイン後に再検証して処理する。文字列 URL をそのまま実行したり、別アカウントのルームを自動で開いたりしない。

푸시 작업을 기존 activity 알림으로 중복 저장하지 않는다. 기존 unread message와 unread activity를 합치는 규칙 때문에 사용자 주의 개수가 이중으로 올라갈 수 있다. 읽음 변경과 이미 provider로 넘긴 알림 사이의 경합은 완전히 제거되지 않으므로 앱 복귀 시 알림과 개수를 정리한다.

プッシュ作業を既存 activity 通知として重複保存しない。既存 unread message・unread activity を合算するルールにより、注意を要する件数が二重に増える場合がある。既読変更と provider に渡した通知間の競合は完全には除けないため、アプリ復帰時に通知・件数を整理する。

푸시를 기기까지 정확히 한 번 전달한다고 보장하지 않는다. 알림 권한, 오프라인, 방해 금지와 OS 정책을 반영하고, Android 설정에서 강제 중지한 상태는 일반 앱 종료와 구분한다. 복귀 시 REST 복구가 대화를 완성해야 한다. [전달 보장 범위](https://docs.expo.dev/push-notifications/faq/#delivery-guarantees) · [앱 상태별 동작](https://docs.expo.dev/push-notifications/what-you-need-to-know/)

プッシュを端末まで正確に 1 回だけ配信するとは保証しない。通知権限・オフライン・通知を制限するモード・OS ポリシーを考慮し、Android 設定で強制停止した状態は通常のアプリ終了と区別する。復帰時の REST 復旧で会話を完全にする必要がある。[配信保証の範囲](https://docs.expo.dev/push-notifications/faq/#delivery-guarantees)・[アプリ状態ごとの動作](https://docs.expo.dev/push-notifications/what-you-need-to-know/)

## 서버 계약과 데이터 초안

## サーバー契約とデータの草案

다음 경로는 구현할 계약의 예시다. 기존 네이밍과 gateway allowlist 및 인증 규칙을 검토한 뒤 최종 경로를 고정한다.

以下の経路は実装する契約の例である。既存の命名・gateway allowlist・認証ルールをレビューしてから最終経路を固定する。

| 계약 초안 | 소유자 | 핵심 조건 |
| --- | --- | --- |
| 기존 auth/login 및 social/google와 token/refresh 확장 | BE | 세션별 sid, 기존 응답 호환, 기기별 로그아웃 |
| POST /api/v1/auth/social/apple 및 연결 경로 | BE | provider subject와 nonce 및 audience 검증, 명시적 연결 |
| PUT/DELETE /api/v1/users/me/push-installations/{installationId} | BE | 사용자는 인증에서 결정, platform과 provider token 및 환경과 generation 관리 |
| GET/PATCH /api/v1/chat/notification-settings | CHAT | 사용자 전역 알림과 미리보기 설정 |
| GET/PATCH /api/v1/chat/rooms/{roomId}/notification-settings | CHAT | 멤버십 확인, 방별 설정 |
| 기존 POST /api/v1/chat/rooms/{roomId}/messages 확장 | CHAT | 선택적 clientMessageId와 응답 연계 |
| 기존 room 목록 DTO 확장 | CHAT | 선택적 recent message 요약, 기존 웹 역호환 |
| POST /api/v1/chat/rooms/{roomId}/reports | CHAT | message 또는 member 신고, 권한과 증거 확인, 요청 제한 |
| POST /api/v1/chat/rooms/{roomId}/members/{memberId}/blocks | CHAT와 BE | OPEN 프로필 ID만 노출, 실제 계정 차단은 내부 계약 |
| POST /api/v1/users/me/deletion-requests | BE | 최근 재인증, 멱등 요청, 전체 세션·푸시 차단과 작업 추적 |
| 내부 push target 조회 및 무효 토큰 반영 | BE | 서비스 인증, 계정과 세션 확인, version 조건부 변경 |

| 契約草案 | 所有者 | 主な条件 |
| --- | --- | --- |
| 既存 auth/login・social/google・token/refresh の拡張 | BE | セッション別 sid、既存応答との互換性、端末別ログアウト |
| POST /api/v1/auth/social/apple と連携経路 | BE | provider subject・nonce・audience 検証、明示的連携 |
| PUT/DELETE /api/v1/users/me/push-installations/{installationId} | BE | ユーザーは認証から決定、platform・provider token・環境・generation 管理 |
| GET/PATCH /api/v1/chat/notification-settings | CHAT | ユーザー全体の通知・プレビュー設定 |
| GET/PATCH /api/v1/chat/rooms/{roomId}/notification-settings | CHAT | メンバーシップ確認、ルーム別設定 |
| 既存 POST /api/v1/chat/rooms/{roomId}/messages の拡張 | CHAT | 任意の clientMessageId と応答の対応付け |
| 既存 room 一覧 DTO の拡張 | CHAT | 任意の recent message 要約、既存 Web との後方互換性 |
| POST /api/v1/chat/rooms/{roomId}/reports | CHAT | message または member の通報、権限・証拠の確認、要求制限 |
| POST /api/v1/chat/rooms/{roomId}/members/{memberId}/blocks | CHAT・BE | OPEN プロフィール ID だけを露出、実アカウントのブロックは内部契約 |
| POST /api/v1/users/me/deletion-requests | BE | 直近の再認証、冪等要求、全セッション・プッシュ遮断と作業追跡 |
| 内部 push target 取得・無効トークン反映 | BE | サービス認証、アカウント・セッション確認、version 条件付き変更 |

BE의 신규 저장 후보는 auth_session, user_identity, push_installation 및 deletion_job이다. CHAT의 후보는 push_outbox, push_delivery, notification_preference와 chat_report이다. 기존 schema와 migration 관례에 맞춰 확정하고, 적용된 migration이나 기존 DB 데이터를 편의로 수정하지 않는다.

BE の新規保存候補は auth_session・user_identity・push_installation・deletion_job。CHAT の候補は push_outbox・push_delivery・notification_preference・chat_report である。既存 schema・migration の慣例に合わせて確定し、適用済み migration や既存 DB データを便宜的に変更しない。

계정 삭제는 앱 화면만으로 완료할 수 없다. BE가 계정 상태와 재인증, 전체 세션 차단 및 삭제 작업을 소유하고 CHAT 및 LL 등 데이터 소유 서비스에 멱등 삭제·익명화 작업을 요청한다. 오픈방 방장 처리, 공유 대화와 가계부, 신고 증거, 저장 객체, 진행 중 AI 및 학습 작업의 늦은 결과를 포함한 보존·삭제 기준을 먼저 확정한다. 서비스 전체를 하나의 DB 트랜잭션으로 묶지 않는다.

アカウント削除はアプリ画面だけでは完了できない。BE がアカウント状態・再認証・全セッション遮断・削除作業を所有し、CHAT・LL などのデータ所有サービスへ冪等な削除・匿名化作業を依頼する。オープンルームのオーナー処理、共有会話・家計簿、通報証拠、保存オブジェクト、進行中 AI・学習作業の遅延結果を含め、保持・削除基準を先に確定する。サービス全体を 1 つの DB トランザクションにはまとめない。

## 계정과 출시 준비

## アカウントとリリースの準備

개인 개발자로 시작한다면 기존 Gmail 주소를 Google 서비스의 소유 계정과 Apple Account의 이메일로 사용할 수 있다. Apple은 같은 이메일을 사용해도 별도 계정이며 Apple Developer 가입과 2단계 인증이 필요하다. 조직 등록을 선택하면 회사 도메인 이메일 등 별도 요건을 확인한다. 자격증명·서명키·실제 푸시 토큰과 개인 이메일은 저장소 문서나 로그에 기록하지 않는다. [Apple Account 생성](https://support.apple.com/en-us/108647)

個人開発者として始める場合、既存 Gmail アドレスを Google サービスの所有アカウント・Apple Account のメールとして使える。Apple は同じメールでも別アカウントであり、Apple Developer 登録と 2 段階認証が必要である。組織登録を選ぶ場合は会社ドメインのメールなど別要件を確認する。資格情報・署名キー・実際のプッシュトークン・個人メールはリポジトリ文書・ログへ記録しない。[Apple Account の作成](https://support.apple.com/en-us/108647)

| 준비 항목 | 용도와 일정 |
| --- | --- |
| Google Play Console 개발자 등록 | Google 계정으로 별도 등록하고 본인 및 계정 유형 확인. 공식 등록비는 일회 US$25다. |
| Firebase와 Google Cloud 설정 | Android FCM 자격, Google 로그인 Android 및 iOS client와 서명 인증서 등록 |
| Apple Account 및 Developer Program | 같은 Gmail 사용 가능. 개인 등록은 실명이 판매자 이름으로 표시되며, 공식 회원비는 연 US$99 또는 지역별 금액이다. |
| Expo 계정과 EAS 프로젝트 | 빌드 및 자격 관리, 환경 구분, push projectId와 배포 설정. 클라우드 빌드의 사용량과 비용을 확인한다. |
| 앱 식별자와 도메인 | Android package와 iOS bundle ID, 개발·운영 구분, OAuth callback 및 공유 링크 도메인 고정 |
| 테스트 기기 | 사용자 Android와 상대방 iPhone을 초기 실기기로 확보. OS 및 앱 버전과 테스트 계정을 기록한다. |

| 準備項目 | 用途と日程 |
| --- | --- |
| Google Play Console 開発者登録 | Google アカウントで別途登録し、本人・アカウント種別を確認。公式登録料は一度限りの US$25。 |
| Firebase・Google Cloud 設定 | Android FCM 資格情報、Google ログインの Android・iOS client と署名証明書登録 |
| Apple Account・Developer Program | 同じ Gmail を利用可能。個人登録では実名が販売者名として表示され、公式会費は年 US$99 または地域別の金額。 |
| Expo アカウント・EAS プロジェクト | ビルド・資格情報管理、環境区分、push projectId・デプロイ設定。クラウドビルドの使用量・費用を確認する。 |
| アプリ識別子・ドメイン | Android package・iOS bundle ID、開発・本番の区別、OAuth callback・共有リンクドメインの固定 |
| テスト端末 | ユーザーの Android と相手の iPhone を初期実機として確保。OS・アプリのバージョンとテストアカウントを記録する。 |

[Google Play 등록과 비용](https://support.google.com/googleplay/android-developer/answer/6112435?hl=en) · [Apple 등록과 비용](https://developer.apple.com/programs/enroll/)

[Google Play 登録と費用](https://support.google.com/googleplay/android-developer/answer/6112435?hl=en)・[Apple 登録と費用](https://developer.apple.com/programs/enroll/)

신규 개인 Google Play 계정이 적용 대상이면 공개 출시 전 12명이 14일 연속 참여하는 비공개 테스트와 production access 신청이 필요하다. 두 사용자의 초기 실기기 테스트와 공개 출시용 모집을 구분하고 계정 유형 확인을 첫 단계에 둔다. [Google Play 비공개 테스트 요건](https://support.google.com/googleplay/android-developer/answer/14151465?hl=en)

新規個人 Google Play アカウントが対象なら、公開リリース前に 12 人が 14 日連続で参加するクローズドテストと production access の申請が必要である。2 人の初期実機テストと公開リリース用の募集を区別し、アカウント種別の確認を最初の段階に置く。[Google Play クローズドテスト要件](https://support.google.com/googleplay/android-developer/answer/14151465?hl=en)

먼저 무료 계정 준비와 Android development build를 진행한다. iPhone development build와 원격 푸시 검증은 유료 Apple Developer 등록 후 시작하고, 이후 Android 내부 테스트와 iOS TestFlight로 배포본을 검증한다. Windows에서는 EAS 클라우드 빌드 및 제출을 사용할 수 있고, 로컬 iOS simulator나 Xcode 디버깅이 필요하면 macOS 실행 환경을 따로 확보한다. [EAS 빌드](https://docs.expo.dev/build/setup/) · [iOS 제출](https://docs.expo.dev/submit/ios/)

無料アカウントの準備と Android development build を先に進める。iPhone development build・リモートプッシュ検証は有料 Apple Developer 登録後に始め、その後 Android 内部テスト・iOS TestFlight で配布版を検証する。Windows では EAS クラウドビルド・提出を利用でき、ローカル iOS simulator・Xcode デバッグが必要なら macOS 実行環境を別に確保する。[EAS ビルド](https://docs.expo.dev/build/setup/)・[iOS 提出](https://docs.expo.dev/submit/ios/)

오픈채팅 공개 출시에는 신고·차단·운영 대응 및 부적절한 콘텐츠 처리 흐름이 필요하다. 기존 문의 화면을 운영 기반으로 활용하되 채팅 증거 검증과 제재 API를 구현한다. 운영 담당자와 연락처, 처리 기록, 이용약관 동의를 출시 전에 준비한다. [Apple UGC 기준](https://developer.apple.com/app-store/review/guidelines/#user-generated-content) · [Google Play UGC 기준](https://support.google.com/googleplay/android-developer/answer/9876937?hl=en)

オープンチャットの公開リリースには、通報・ブロック・運営対応・不適切なコンテンツ処理の流れが必要である。既存問い合わせ画面を運営基盤として利用し、チャット証拠の検証・制裁 API を実装する。運営担当者・連絡先・処理記録・利用規約同意をリリース前に準備する。[Apple UGC 基準](https://developer.apple.com/app-store/review/guidelines/#user-generated-content)・[Google Play UGC 基準](https://support.google.com/googleplay/android-developer/answer/9876937?hl=en)

가입을 제공하므로 앱 안에서 계정 삭제를 시작할 수 있게 하고, 웹에서도 삭제 요청을 제출할 수 있는 경로를 제공한다. App Privacy와 Data safety에는 메시지·번역 처리·프로필 이미지·기기 토큰 및 사용 SDK의 실제 수집과 공유를 반영한다. 외부 AI 공급자에게 메시지가 전달되는 번역과 AI 기능은 공유 범위를 설명하고 필요한 명시적 동의를 받는 흐름을 포함한다. 삭제와 보존 범위도 일치시킨다. [Apple 개인정보 기준](https://developer.apple.com/app-store/review/guidelines/#privacy) · [Google Play 계정 삭제 요건](https://support.google.com/googleplay/android-developer/answer/13327111?hl=en)

登録を提供するため、アプリ内でアカウント削除を開始できるようにし、Web でも削除要求を提出する経路を提供する。App Privacy・Data safety にはメッセージ・翻訳処理・プロフィール画像・端末トークン・利用 SDK の実際の収集と共有を反映する。外部 AI プロバイダーへメッセージが渡る翻訳・AI 機能は、共有範囲を説明して必要な明示的同意を得る流れを含める。削除・保持範囲も一致させる。[Apple プライバシー基準](https://developer.apple.com/app-store/review/guidelines/#privacy)・[Google Play アカウント削除要件](https://support.google.com/googleplay/android-developer/answer/13327111?hl=en)

양쪽 스토어 심사와 배포 반영 시간은 독립적이다. 첫 공개 버전에 두 플랫폼을 모두 준비하되, 정확히 같은 분에 노출되는 것을 약속하지 않는다. Apple의 수동 공개를 이용해 출시일을 조정하며, Google 관리형 게시의 최초 출시 제한을 확인한다. [Apple 공개 옵션](https://developer.apple.com/help/app-store-connect/manage-your-apps-availability/select-an-app-store-version-release-option) · [Google 게시 제어](https://support.google.com/googleplay/android-developer/answer/9859654?hl=en)

両ストアの審査・配信反映時間は独立している。最初の公開バージョンに両プラットフォームを用意するが、厳密に同じ分に表示されるとは約束しない。Apple の手動公開でリリース日を調整し、Google の管理対象公開にある初回リリース制限を確認する。[Apple 公開オプション](https://developer.apple.com/help/app-store-connect/manage-your-apps-availability/select-an-app-store-version-release-option)・[Google 公開制御](https://support.google.com/googleplay/android-developer/answer/9859654?hl=en)

## 개발 순서와 예상 작업량

## 開発順序と想定作業量

한 명이 집중 개발하며 기존 서버를 이해하고 있다는 가정의 초기 추정이다. 단계 합계는 38~56 개발일, 약 8~12주다. 개발자 등록·테스터 모집·심사 대기 및 서비스 전체 계정 삭제의 조사 결과에 따른 추가 작업은 별도로 영향을 준다. 첫 단계가 끝나면 일정을 다시 산정한다.

1 人が集中して開発し、既存サーバーを理解しているという仮定での初期見積もりである。段階の合計は 38～56 開発日、およそ 8～12 週間。開発者登録・テスター募集・審査待ち・サービス全体のアカウント削除調査による追加作業は、別途影響する。最初の段階が終わったら日程を再見積もりする。

| 단계 | 주요 작업 | 완료 기준 | 예상 개발일 |
| --- | --- | --- | --- |
| 0 준비와 계약 | 계정 유형과 테스터 일정, 앱 식별자, Expo 호환 조합, 기존 API 고정, 삭제 범위 조사, 양쪽 최소 build와 테스트 push | Android와 iPhone에 앱 설치, 양쪽 테스트 알림 수신, 미정 계약 목록 확정 | 2~3 |
| 1 인증 | 기기별 session, 기존 웹 유예 전환, Google와 Apple 및 계정 연결, native 저장과 갱신 | 같은 계정으로 웹과 양쪽 앱 동시 사용, 한 기기 logout이 다른 기기를 끊지 않음 | 6~9 |
| 2 친구와 기본 채팅 | 친구 연결, DIRECT와 GROUP, 메시지 멱등성, 번역, 읽음, 이력과 복구 | 웹·Android·iOS 간 전송과 읽음 일치, 응답 유실 재시도 중복 없음 | 6~8 |
| 3 오픈채팅 | 탐색·생성·참여·방별 프로필·공유·역할·강퇴·위임·종료 | 실제 계정 ID 비노출, 권한 변경과 재참여 이력 제한 일치 | 6~9 |
| 4 운영 푸시 | BE 기기 lifecycle, CHAT outbox 및 worker, 설정과 receipt, deep link와 badge | 실제 메시지에서 잠금 화면 알림과 방 진입, 재시도·mute·로그아웃 검증 | 6~9 |
| 5 공개 서비스 준비 | 신고·차단·필터 및 운영 처리, 계정 삭제·제재, 정책·스토어 정보와 리뷰 계정 | 오픈채팅 보호 흐름과 계정 삭제 동작, 검토 가능한 스토어 제출물 | 7~10 |
| 6 회귀와 출시 | 양쪽 beta, 네트워크·기기 matrix, 기존 웹 회귀, 서버 순차 배포와 스토어 심사 대응 | 필수 검증 통과, 양쪽 배포본 준비, 푸시 비활성화와 장애 대응 가능 | 5~8 |

| 段階 | 主な作業 | 完了基準 | 想定開発日 |
| --- | --- | --- | --- |
| 0 準備・契約 | アカウント種別・テスター日程、アプリ識別子、Expo 互換組み合わせ、既存 API 固定、削除範囲調査、両方の最小 build・テスト push | Android・iPhone へのアプリインストール、両方のテスト通知受信、未定契約一覧の確定 | 2～3 |
| 1 認証 | 端末別 session、既存 Web 猶予移行、Google・Apple・アカウント連携、native 保存・更新 | 同アカウントで Web・両アプリを同時利用し、1 端末の logout が他端末を切断しない | 6～9 |
| 2 友達・基本チャット | 友達連携、DIRECT・GROUP、メッセージ冪等性、翻訳、既読、履歴・復旧 | Web・Android・iOS 間の送信・既読一致、応答喪失後の再試行で重複なし | 6～8 |
| 3 オープンチャット | 探索・作成・参加・ルーム別プロフィール・共有・役割・強制退室・委任・終了 | 実アカウント ID を非公開にし、権限変更・再参加の履歴制限が一致 | 6～9 |
| 4 本番プッシュ | BE 端末 lifecycle、CHAT outbox・worker、設定・receipt、deep link・badge | 実メッセージからロック画面通知・ルーム遷移、再試行・mute・ログアウト検証 | 6～9 |
| 5 公開サービス準備 | 通報・ブロック・フィルター・運営処理、アカウント削除・制裁、ポリシー・ストア情報・審査用アカウント | オープンチャット保護フロー・アカウント削除動作、レビュー可能なストア提出物 | 7～10 |
| 6 回帰・リリース | 両方の beta、ネットワーク・端末 matrix、既存 Web 回帰、サーバー順次デプロイ・ストア審査対応 | 必須検証通過、両配布版の準備、プッシュ無効化・障害対応が可能 | 5～8 |

단계 0의 테스트 알림은 provider와 기기 검증이고 단계 4의 메시지 기반 운영 푸시와 구분한다. 푸시 기반 작업은 오픈채팅 화면과 병행할 수 있다. 신고 및 계정 삭제는 마지막에 처음 착수하지 않고 단계 0에서 데이터 범위를 정해 인증과 채팅 개발 중 함께 구현한다. 비공개 테스트는 충분히 사용 가능한 beta가 나온 뒤 시작하며, 달력상 14일을 코드 작업량과 혼동하지 않는다.

段階 0 のテスト通知は provider・端末の検証であり、段階 4 のメッセージに基づく本番プッシュと区別する。プッシュ基盤作業はオープンチャット画面と並行できる。通報・アカウント削除は最後に初着手せず、段階 0 でデータ範囲を定め、認証・チャット開発中に一緒に実装する。クローズドテストは十分に利用可能な beta ができてから始め、カレンダー上の 14 日とコード作業量を混同しない。

Apple 유료 등록을 연기하는 동안 단계 0의 코드 기반과 무료 준비를 먼저 마치고 단계 1 이후 개발을 진행할 수 있다. 이때 iPhone 빌드·테스트 푸시는 대기로 남기며 단계 0 전체 완료로 표시하지 않는다. 유료 등록 후 이 검증을 보충하고, 공개 출시 전에는 양쪽 실기기 및 배포본 완료 기준을 모두 충족한다.

Apple の有料登録を延期する間に、段階 0 のコード基盤・無料準備を先に終え、段階 1 以降の開発を進められる。その場合 iPhone ビルド・テストプッシュは待機に残し、段階 0 全体を完了とは表示しない。有料登録後にこの検証を補い、公開リリース前には両実機・配布版の完了基準をすべて満たす。

## 필수 검증과 출시 완료 기준

## 必須検証とリリース完了基準

| 검증 | 통과 조건 |
| --- | --- |
| 세션과 기존 웹 | 웹·Android·iPhone 로그인 및 갱신 병행, 현재 기기 및 전체 logout 구분, 기존 웹 유예 토큰 회귀 없음 |
| 로그인 제공자 | 기존 계정 유지, Apple 비공개 이메일, 명시적 연결, OAuth 취소·만료·잘못된 audience·nonce 거부 |
| 전송과 중복 | REST 응답 유실, 중복 STOMP, 동일 키 재시도, 다른 내용의 같은 키 충돌, 서버 저장과 앱 표시 일치 |
| 복구와 정렬 | 비행기 모드 및 Wi-Fi·모바일망 전환, background 동안 여러 페이지 메시지, 번역·읽음 역순 도착, 중간 누락 없음 |
| 오픈채팅 권한 | PUBLIC과 UNLISTED, 역할별 관리, 강퇴·차단·방 종료, 재참여 경계, 실제 프로필 비노출 |
| 푸시와 계정 | 양쪽 foreground·background·잠금·일반 종료, cold start와 로그인 대기 진입, 재설치·토큰 교체·다중 기기·계정 전환 |
| 설정과 개인정보 | OS 권한 거절과 재허용, 전역 및 방별 OFF, 미리보기 OFF, 웹 online 중 휴대폰 수신, 탈퇴 계정 전달 차단 |
| 서버 내구성 | 메시지 rollback 시 작업 미생성, outbox 기록 실패 시 메시지 rollback, commit 직후 재시작, worker 동시 실행, 무효 토큰, 외부 전송 장애가 메시지 저장을 막지 않음 |
| 신고와 계정 삭제 | 다른 방 증거 제출 거부, OPEN 신고자·대상 신원 비노출, 운영 처리 감사기록, 전체 세션 폐기와 서비스별 삭제 재시도 |
| 네이티브 사용성 | 작은 화면과 큰 글씨, 한일영 입력·IME·긴 문장, 키보드와 스크롤, 화면 reader, 프로필 이미지 재시도 |
| 배포본 | development build뿐 아니라 Android 및 TestFlight beta에서 검증, 실제 서버 계약과 서명·push 자격 일치 |

| 検証 | 通過条件 |
| --- | --- |
| セッション・既存 Web | Web・Android・iPhone のログイン・更新並行、現在端末・全体 logout の区別、既存 Web 猶予トークンの回帰なし |
| ログインプロバイダー | 既存アカウント維持、Apple 非公開メール、明示的連携、OAuth キャンセル・期限切れ・不正 audience・nonce の拒否 |
| 送信・重複 | REST 応答喪失、STOMP 重複、同キー再試行、内容が異なる同キーの競合、サーバー保存とアプリ表示の一致 |
| 復旧・並び順 | 機内モード・Wi-Fi とモバイル回線の切り替え、background 中の複数ページのメッセージ、翻訳・既読の逆順到着、中間欠落なし |
| オープンチャット権限 | PUBLIC・UNLISTED、役割別管理、強制退室・ブロック・ルーム終了、再参加境界、実プロフィールの非公開 |
| プッシュ・アカウント | 両方の foreground・background・ロック・通常終了、cold start・ログイン待ちからの遷移、再インストール・トークン交換・複数端末・アカウント切り替え |
| 設定・個人情報 | OS 権限拒否・再許可、全体・ルーム別 OFF、プレビュー OFF、Web online 中の携帯受信、退会アカウントへの配信遮断 |
| サーバー耐久性 | メッセージ rollback 時は作業を作らない、outbox 記録失敗時はメッセージ rollback、commit 直後の再起動、worker 同時実行、無効トークン、外部送信障害がメッセージ保存を妨げない |
| 通報・アカウント削除 | 別ルームの証拠提出を拒否、OPEN 通報者・対象の身元を非公開、運営処理監査記録、全セッション破棄・サービス別削除の再試行 |
| ネイティブの使いやすさ | 小画面・大きな文字、韓日英入力・IME・長文、キーボード・スクロール、スクリーンリーダー、プロフィール画像の再試行 |
| 配布版 | development build だけでなく Android・TestFlight beta で検証、実サーバー契約・署名・push 資格情報の一致 |

모바일은 타입 검사와 lint, 메시지 병합·재시도·읽음 및 계정 전환에 대한 집중 테스트를 둔다. BE는 세션 및 identity와 기기 소유권, CHAT는 메시지 트랜잭션과 DB unique 및 outbox·Redis·권한 통합 테스트를 확장한다. mock 검증을 실제 기기 푸시나 운영 DB 적용 근거로 사용하지 않는다.

モバイルには型検査・lint、およびメッセージ結合・再試行・既読・アカウント切り替えの集中テストを設ける。BE はセッション・identity・端末所有権、CHAT はメッセージトランザクション・DB unique・outbox・Redis・権限の統合テストを拡張する。mock 検証を実機プッシュや本番 DB 適用の根拠には使わない。

알림 지연과 worker backlog, 재시도와 영구 실패, 토큰 무효 및 로그인 실패를 관측한다. 메시지 본문·인증 토큰·개인정보를 로그에 남기지 않는다. 개발 환경의 정상 네트워크에서 알림 왕복 시간을 측정해 출시 수용 기준을 정하며 OS 전달 시간을 일괄 보장하지 않는다.

通知遅延・worker backlog、再試行・恒久失敗、トークン無効・ログイン失敗を観測する。メッセージ本文・認証トークン・個人情報をログへ残さない。開発環境の正常ネットワークで通知の往復時間を測り、リリース受け入れ基準を定め、OS の配信時間を一律には保証しない。

## 배포와 운영 순서

## デプロイと運用の順序

1. 검증 환경에 추가 schema와 호환 API를 반영하고 기존 웹 계약 테스트를 통과시킨다. 계정 삭제·기기 세션 및 식별자 계약도 포함한다.
2. BE와 CHAT에 기능을 추가하되 운영 푸시는 기본 비활성으로 둔다. 기존 웹 요청은 계속 처리한다.
3. 두 플랫폼의 native beta와 해당 테스트 계정을 연결해 실제 메시지 기반 푸시와 복구 및 사용자 보호 기능을 검증한다.
4. 필요한 비공개 테스트와 개인정보 정보 및 스토어 심사 자료를 완료한다. 심사용 계정과 backend 접근을 유지한다.
5. 제한한 기기와 계정부터 푸시를 활성화하고 오류와 backlog를 확인한 뒤 대상 범위를 넓힌다.
6. 두 스토어 배포를 조율한다. 장애 시 새 푸시와 worker를 끌 수 있게 하고, 기존 메시지 저장과 웹을 계속 제공한다. 이미 배포된 앱은 서버 API가 갑자기 사라지지 않도록 호환 기간을 유지한다.

1. 検証環境へ追加 schema・互換 API を反映し、既存 Web の契約テストを通す。アカウント削除・端末セッション・識別子契約も含める。
2. BE・CHAT に機能を追加するが、本番プッシュは既定で無効にする。既存 Web 要求は引き続き処理する。
3. 両プラットフォームの native beta と該当テストアカウントを接続し、実メッセージを使うプッシュ・復旧・ユーザー保護機能を検証する。
4. 必要なクローズドテスト・プライバシー情報・ストア審査資料を完了する。審査用アカウント・backend アクセスを維持する。
5. 限定した端末・アカウントからプッシュを有効にし、エラー・backlog を確認してから対象範囲を拡大する。
6. 両ストア配布を調整する。障害時は新規プッシュ・worker を停止できるようにし、既存メッセージ保存・Web の提供を続ける。配布済みアプリ向けのサーバー API が突然なくならないよう互換期間を維持する。

현재 실행 순서는 단계 0의 앱 기반과 무료 Expo·Firebase 준비, Android 실기기 build·테스트 push 준비, 단계 1의 다중 세션 계약 확정이다. iPhone 실기기 build·테스트 push는 Apple 유료 등록 후 보충해 플랫폼 자격 문제를 확인하며, 기존 웹 로그인 충돌은 세션 계약 단계에서 함께 검증한다.

現在の実行順序は、段階 0 のアプリ基盤・無料 Expo と Firebase の準備、Android 実機 build・テスト push の準備、段階 1 の複数セッション契約の確定である。iPhone 実機 build・テスト push は Apple 有料登録後に補い、プラットフォームの資格問題を確認し、既存 Web ログインの競合はセッション契約段階で合わせて検証する。

## 로컬 근거

## ローカルの根拠

아래는 계획 수립 시 읽은 소스이며 실제 배포 상태나 테스트 통과를 의미하지 않는다.

以下は計画策定時に読んだソースであり、実際のデプロイ状態やテスト通過を意味しない。

- BE 사용자별 refresh 저장 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-be/src/main/java/jp/co/translacat/domain/user/entity/RefreshToken.java:18`) 및 발급과 로그아웃 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-be/src/main/java/jp/co/translacat/domain/user/service/UserService.java:143`)
- BE 인증 수단 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-be/src/main/java/jp/co/translacat/domain/user/entity/User.java:15`) 및 Google 증명 검증 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-be/src/main/java/jp/co/translacat/domain/user/auth/account/GoogleIdentityVerifier.java:19`)
- BE 문의 관련 쪽지 검증 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-be/src/main/java/jp/co/translacat/domain/support/service/SupportInquiryService.java:63`)
- BE 공개 채팅 WebSocket 중계 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-be/src/main/java/jp/co/translacat/infrastructure/chat/gateway/ChatGatewayWebSocketHandler.java:119`)
- CHAT 메시지 생성 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-chat/TranslaCat.Chat.Application/Messaging/ChatMessageService.cs:29`) 및 커밋 뒤 전달 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-chat/TranslaCat.Chat.Infrastructure/Persistence/Repositories/EfChatMessageTransaction.cs:32`)
- CHAT activity와 메시지 attention 합산 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-chat/TranslaCat.Chat.Application/Notifications/ChatNotificationService.cs:7`)
- CHAT 이력과 복구 API (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-chat/TranslaCat.Chat.Api/Controllers/ChatMessagesController.cs:30`)
- FE NextAuth 의존 API client (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-fe/src/lib/apiClient.ts:1`) 및 채팅 타입 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-fe/src/types/chat.ts:275`)
- FE STOMP 전송 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-fe/src/hooks/chat/useChatRoomWebSocket.ts:528`) 및 재연결 동기화 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-fe/src/hooks/chat/useChatRoom.ts:733`)
- FE 오픈채팅 API (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-fe/src/services/chat/openChatService.ts:19`), 역할별 관리 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-fe/src/utils/chat/openChatModeration.ts:11`), 계정 ID 기반 기존 차단 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-fe/src/services/block/userBlockService.ts:29`)

- BE のユーザー別 refresh 保存 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-be/src/main/java/jp/co/translacat/domain/user/entity/RefreshToken.java:18`)・発行とログアウト (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-be/src/main/java/jp/co/translacat/domain/user/service/UserService.java:143`)
- BE 認証手段 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-be/src/main/java/jp/co/translacat/domain/user/entity/User.java:15`)・Google 証明検証 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-be/src/main/java/jp/co/translacat/domain/user/auth/account/GoogleIdentityVerifier.java:19`)
- BE 問い合わせ関連メッセージ検証 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-be/src/main/java/jp/co/translacat/domain/support/service/SupportInquiryService.java:63`)
- BE 公開チャット WebSocket 中継 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-be/src/main/java/jp/co/translacat/infrastructure/chat/gateway/ChatGatewayWebSocketHandler.java:119`)
- CHAT メッセージ作成 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-chat/TranslaCat.Chat.Application/Messaging/ChatMessageService.cs:29`)・commit 後の配信 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-chat/TranslaCat.Chat.Infrastructure/Persistence/Repositories/EfChatMessageTransaction.cs:32`)
- CHAT activity とメッセージ attention の合算 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-chat/TranslaCat.Chat.Application/Notifications/ChatNotificationService.cs:7`)
- CHAT 履歴・復旧 API (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-chat/TranslaCat.Chat.Api/Controllers/ChatMessagesController.cs:30`)
- FE の NextAuth 依存 API client (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-fe/src/lib/apiClient.ts:1`)・チャット型 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-fe/src/types/chat.ts:275`)
- FE STOMP 送信 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-fe/src/hooks/chat/useChatRoomWebSocket.ts:528`)・再接続同期 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-fe/src/hooks/chat/useChatRoom.ts:733`)
- FE オープンチャット API (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-fe/src/services/chat/openChatService.ts:19`)、役割別管理 (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-fe/src/utils/chat/openChatModeration.ts:11`)、アカウント ID に基づく既存ブロック (당시 로컬 소스 / 当時のローカルソース: `C:/workspace/project-cat/CatPjt-TranslaCat-fe/src/services/block/userBlockService.ts:29`)
