# 모바일 방 관리 상태 실시간 반영

# モバイルのルーム管理状態をリアルタイムで反映

2026-10-22 사용자는 ID 복사·공유, 친구 초대와 방 관리 동작 확인을 **“확인한걸루 치고”** 다음 작업으로 진행하도록 요청했다. 이 항목은 `ASSUMED_BY_USER`이며 실제 휴대폰 시험 관찰로 기록하지 않는다. 이번 구현 범위는 다른 기기·웹에서 역할 변경, 강퇴, 방 종료가 발생했을 때 모바일 참여자·방 관리·AI 관리·채팅의 권한과 가능한 동작을 갱신하는 것이다.

2026-10-22、ユーザーは ID のコピー・共有、友達の招待とルーム管理動作について、**『確認したことにして』**次の作業へ進むよう依頼した。この項目は `ASSUMED_BY_USER` であり、実際の端末試験を観察した記録として扱わない。今回の実装範囲は、別の端末や Web で役割変更、強制退室、ルーム終了が発生した際に、モバイルの参加者・ルーム管理・AI 管理・チャットの権限と利用可能な操作を更新することである。

## 기존 서버 계약

## 既存のサーバー契約

- CHAT은 room topic에 `chat.members.changed`, `chat.member.role.updated`, `chat.member.banned`, `chat.room.closed`를 발행한다. 역할 값이나 대상 ID는 권한 결정의 근거로 직접 적용하지 않고 HTTP 재조회의 신호로 사용한다.
- 강퇴 대상은 멤버십 해제 후 room topic을 받을 수 없다. 이미 연결된 `/user/queue/chat/open-rooms/{roomId}` 개인 구독에도 마지막 강퇴 이벤트가 전달되므로 기존 `connectRoom(open: true)`의 두 경로를 모두 사용한다.
- 종료는 OPEN 상태만 `CLOSED`로 바꾸고 멤버십·일반 방의 `active`는 유지한다. `GET /chat/rooms/{id}`만으로 종료를 판단할 수 없어 `GET /chat/open-rooms/{id}`의 상태를 함께 읽는다. 종료한 방의 기존 기록을 읽는 것과 메시지·관리 변경을 구분한다.
- 강퇴 후 room GET은 코드 없는 HTTP 400이며, members/me의 `OPEN_CHAT_BANNED`도 모바일에서는 `HTTP_400`으로 남는다. room 조회 단계의 접근 상실과 변경 요청의 잘못된 입력을 구분한다. 모든 쓰기 400을 강퇴로 취급하지 않는다.
- Redis pub/sub의 내구성 있는 재생은 보장되지 않는다. 재연결·화면 복귀의 HTTP 확인과 낮은 빈도 조회가 누락 보완 수단이다.

- CHAT は room topic に `chat.members.changed`、`chat.member.role.updated`、`chat.member.banned`、`chat.room.closed` を発行する。役割値や対象 ID を権限判断へ直接適用せず、HTTP 再取得の合図として使う。
- 強制退室の対象者はメンバーシップ解除後に room topic を受信できない。接続済みの `/user/queue/chat/open-rooms/{roomId}` 個人購読にも最後の強制退室イベントが届くため、既存の `connectRoom(open: true)` の両経路を使用する。
- 終了時は OPEN の状態のみを `CLOSED` に変更し、メンバーシップと一般ルームの `active` は維持する。`GET /chat/rooms/{id}` だけでは終了を判断できず、`GET /chat/open-rooms/{id}` の状態も読む。終了したルームの既存履歴の閲覧と、メッセージ・管理状態の変更を区別する。
- 強制退室後の room GET はコードなしの HTTP 400 であり、members/me の `OPEN_CHAT_BANNED` もモバイルでは `HTTP_400` のままとなる。room 取得段階のアクセス喪失と、変更要求の不正入力を区別する。書き込みの 400 をすべて強制退室として扱わない。
- Redis pub/sub の永続的な再生は保証されない。再接続・画面復帰時の HTTP 確認と低頻度の取得で取りこぼしを補う。

읽기 근거는 CHAT의 `OpenMembershipRealtimeDelivery.cs`, `OpenModerationExtensions.cs`, `ChatRealtimeBroker.cs`, `EfChatRoomStore.cs`, `EfOpenMembershipStore.Ownership.cs`, BE의 `ChatGatewayForwardingFilter.java`, 기존 FE의 `useChatRoomWebSocket.ts`와 `useOpenChatRoomEventHandlers.ts`다. 실제 운영 서버를 변경하거나 관리 요청을 보내서 조사하지 않았다.

参照した根拠は CHAT の `OpenMembershipRealtimeDelivery.cs`、`OpenModerationExtensions.cs`、`ChatRealtimeBroker.cs`、`EfChatRoomStore.cs`、`EfOpenMembershipStore.Ownership.cs`、BE の `ChatGatewayForwardingFilter.java`、既存 FE の `useChatRoomWebSocket.ts` と `useOpenChatRoomEventHandlers.ts` である。実際の本番サーバーを変更したり管理要求を送信したりして調査したわけではない。

## 앱 처리

## アプリの処理

참여자·방 관리 화면에 현재 로그인 세대와 활성 화면에 묶인 구독을 연결한다. 외부 변경 신호를 받으면 권한·관리 자료를 먼저 무효화하고 새 HTTP 결과가 일치할 때 다시 공개한다. 열린 확인창은 세대가 달라지면 닫으며, 오래된 확인으로 변경 요청을 보내지 않는다. 조회 중 도착한 신호와 이벤트 폭주는 최신 조회로 합친다. 처리 중이던 POST는 자동으로 재전송하지 않는다.

参加者・ルーム管理画面に、現在のログイン世代とアクティブ画面に結び付いた購読を接続する。外部変更の合図を受けたら権限・管理データを先に無効化し、新しい HTTP 結果が一致した時点で再表示する。開いている確認ダイアログは世代が変われば閉じ、古い確認から変更要求を送らない。取得中に届いた合図や大量イベントは最新の取得へ集約する。処理中だった POST は自動再送しない。

AI 관리 화면도 같은 변경을 받아 persona 목록과 편집·삭제 UI를 정리하고, 변경 직전 현재 역할·종료 상태를 재확인한다. 일반 주기 확인에서 권한이 그대로라면 작성 중인 초안을 보존한다.

AI 管理画面も同じ変更を受け、persona 一覧と編集・削除 UI を整理し、変更直前に現在の役割・終了状態を再確認する。通常の定期確認で権限が変わっていなければ、入力中の下書きを保持する。

채팅은 관리 이벤트·재연결·복귀 때 권한 확인 전 메뉴·전송·읽음 요청을 잠근다. 캐시를 화면에서 숨겨도 목록 자체는 유지해 기존 스크롤 복원 방식을 보존한다. 접근 거절이 확인되면 이전 이력을 정리한다. 종료된 OPEN에서는 기록을 보존하고 메시지 전송과 AI 관리 진입을 막는다.

チャットは管理イベント・再接続・復帰時に、権限確認までメニュー・送信・既読要求をロックする。キャッシュを画面から隠しても一覧自体は保持し、既存のスクロール復元方式を維持する。アクセス拒否が確認されたら以前の履歴を整理する。終了済み OPEN では履歴を保持し、メッセージ送信と AI 管理への移動を防ぐ。

## 검증과 설치

## 検証とインストール

128개 동결 입력으로 **전체 테스트 383/383**, lint 경고 0, TypeScript, device 공개 HTTPS config와 Android/iOS/web JavaScript export가 모두 exit 0을 통과했다. 검사 중 변경된 입력은 0개다. 동결 manifest SHA-256은 `ed371df9b0529471932ccbcea82e607b782aac06f6fbd4d8c71e742c23ddee95`, `checks.json` SHA-256은 `6120edefa9fd406287c9048e52e23e6930d46c3f84a12de8d0c81f9f3e078ab7`이다. export는 합성 loopback API를 쓰는 JS 검사이며 공개 device 설정 검사를 따로 수행했다.

凍結した入力 128 件で、**全テスト 383/383**、lint 警告 0、TypeScript、device の公開 HTTPS config、Android/iOS/web の JavaScript export がすべて exit 0 で通過した。検査中に変わった入力は 0 件である。凍結 manifest の SHA-256 は `ed371df9b0529471932ccbcea82e607b782aac06f6fbd4d8c71e742c23ddee95`、`checks.json` の SHA-256 は `6120edefa9fd406287c9048e52e23e6930d46c3f84a12de8d0c81f9f3e078ab7`。export は合成 loopback API を使用する JS 検査であり、公開 device 設定の検査は別に実行した。

방 관리 집중 45개, AI 집중 38개, root의 채팅·서비스 집중 47개는 전체 검증과 겹치는 범위이므로 합산하지 않는다. 외부 역할·강퇴·CLOSED, 실제 서버의 HTTP400, 이벤트 폭주·조회/변경 경합, 재연결·계정/화면 전환, 15초 요청 제한·늦은 응답과 POST 비재전송을 검증했다. 정상 주기 조회는 권한이 바뀌지 않으면 동의·AI 초안·이미 펼친 차단 페이지와 cursor를 보존한다.

ルーム管理の集中 45 件、AI の集中 38 件、root のチャット・サービス集中 47 件は全体検証と重なるため合算しない。外部からの役割変更・強制退室・CLOSED、実サーバーの HTTP400、イベント集中・取得と変更の競合、再接続・アカウントと画面の切り替え、15 秒の要求制限・遅延応答・POST 非再送を検証した。通常の定期取得では権限が変わらなければ、同意・AI の下書き・展開済みのブロック一覧ページと cursor を保持する。

독립 리뷰에서 실제 bans HTTP400 재확인 누락과 정상 poll의 두 번째 차단 페이지 소실을 찾아 수정했다. 같은 방으로 복귀하는 첫 렌더의 이전 권한 노출도 lifetime 소유권으로 차단했다. 수정 후 독립 회귀 4개도 통과했고 추가 blocker는 없었다. 근거는 `review-summary.json`이다. 실제 React 화면·단말 스크롤·운영 STOMP 전달을 이 리뷰로 검증한 것은 아니다.

独立レビューで、実際の bans HTTP400 の再確認漏れと、通常 poll によるブロック一覧 2 ページ目の消失を発見して修正した。同じルームへ戻る最初のレンダーで以前の権限が露出する問題も lifetime の所有権で遮断した。修正後の独立回帰 4 件も通過し、追加 blocker はなかった。根拠は `review-summary.json`。このレビューで実際の React 画面・端末スクロール・本番 STOMP 配信を検証したわけではない。

이전 설치본 `58bbf721-3a3a-4f82-a825-6862a3c6789b`의 글자 표시 수락과 이번 실시간 기능의 수락을 구분한다. 새 Android APK의 공식 메타데이터·공개 설정 7개·ZIP과 JS bundle 변경을 확인했고, 동결 입력 128개에 변경이 없다. 컴파일된 `AppTheme` → `TranslaCatTextView` 참조와 `useBoundsForWidth=false`의 boolean 값, 부모 스타일과 해당 이름의 모든 variant가 유지됐다. resource table은 이전 58bb와 동일하다. 이 검사로 실제 Activity의 테마 적용·화면 렌더, 서명·binary manifest 또는 클라우드 전체 소스 일치를 확인한 것은 아니다. iPhone 네이티브 확인과 이번 Android의 실제 다중 기기 이벤트 확인은 아직 완료하지 않았다.

以前のインストール版 `58bbf721-3a3a-4f82-a825-6862a3c6789b` の文字表示の受け入れと、今回のリアルタイム機能の受け入れは区別する。新しい Android APK の公式メタデータ・公開設定 7 件・ZIP・JS bundle の変更を確認し、凍結入力 128 件に変更はない。コンパイル済みの `AppTheme` → `TranslaCatTextView` 参照、`useBoundsForWidth=false` の boolean 値、親スタイルと同名の全 variant が維持された。resource table は以前の 58bb と同一。この検査で実際の Activity のテーマ適用・画面描画、署名・binary manifest、クラウド全ソースの一致を確認したわけではない。iPhone のネイティブ確認と、今回の Android の実際の複数端末イベント確認はまだ完了していない。

새 [Android APK 설치 페이지](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/633f0bb9-c02c-4925-9307-f554067beddb)의 빌드 ID는 `633f0bb9-c02c-4925-9307-f554067beddb`다. 2026-10-22 12:11:13 KST에 접수해 **12:32:44 KST FINISHED** 됐다. 검증된 APK는 **93,479,539 bytes**, SHA-256 `a693bd1d3963d08c18d4984bb8b1f3ed78f741d2d7a8d6d7776a0a4f4896d1a5`다. 빌드 요청 1회·설치 파일 다운로드 1회이며 유료 작업은 없다. root가 APK와 세 검증 영수증의 실제 파일 해시를 대조했다. 근거는 `apk/apk-completed.json`, `apk/verification/verification-manifest.json`, `apk/verification/compiled-text-style-verification.json`, `root-artifact-review.json`이다. 실제 휴대폰의 방 관리 실시간 확인은 `PENDING_NEW_APK`다.

新しい [Android APK インストールページ](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/633f0bb9-c02c-4925-9307-f554067beddb)のビルド ID は `633f0bb9-c02c-4925-9307-f554067beddb`。2026-10-22 12:11:13 KST に受付し、**12:32:44 KST FINISHED** となった。検証済み APK は **93,479,539 bytes**、SHA-256 は `a693bd1d3963d08c18d4984bb8b1f3ed78f741d2d7a8d6d7776a0a4f4896d1a5`。ビルド要求 1 回・インストールファイルのダウンロード 1 回で、有料作業はない。root が APK と 3 件の検証記録の実ファイルハッシュを照合した。根拠は `apk/apk-completed.json`、`apk/verification/verification-manifest.json`、`apk/verification/compiled-text-style-verification.json`、`root-artifact-review.json`。実際の端末でのルーム管理リアルタイム確認は `PENDING_NEW_APK` である。

## 추적과 보존

## 追跡と保存

기존 [설정·소셜 #41](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/41)에는 확인 간주 범위를, [OPEN 관리 #45](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/45)에는 이번 진행 범위를 기록했다. 기존 Project 12에서 #45를 In progress로 옮겼고 실제 기기 검증 체크를 완료로 바꾸지 않았다. 새 이슈·프로젝트를 만들지 않고 기존 항목을 재사용한다.

既存の[設定・ソーシャル #41](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/41)には確認済みとみなす範囲を、[OPEN 管理 #45](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/45)には今回の進行範囲を記録した。既存 Project 12 で #45 を In progress に移動し、実機検証のチェックは完了に変えていない。新しい issue・project を作らず既存項目を再利用する。

APK 완료 후 [#45 기존 댓글](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/45#issuecomment-6331344432)과 [#52 기존 댓글](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/52#issuecomment-6312207253)을 각각 한 번 갱신하고 정확한 본문 readback을 확인했다. 새 설치 링크와 383개 검증 결과를 반영했으며 #45는 OPEN·In progress, #41은 ASSUMED_BY_USER, 이번 실제 다중 기기 확인은 PENDING을 유지한다. issue body·checklist·기타 field와 새 댓글·새 이슈를 변경하지 않았다. 최종 근거는 `issues/final-public-update-confirmed.json`이다.

APK 完了後に [#45 の既存コメント](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/45#issuecomment-6331344432)と [#52 の既存コメント](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/52#issuecomment-6312207253)をそれぞれ 1 回更新し、正確な本文の readback を確認した。新しいインストールリンクと 383 件の検証結果を反映し、#45 は OPEN・In progress、#41 は ASSUMED_BY_USER、今回の実際の複数端末確認は PENDING を維持する。issue body・checklist・その他 field、新規コメント・新規 issue は変更していない。最終根拠は `issues/final-public-update-confirmed.json`。

중앙 근거는 `.codex-workspace/verification/shared/mobile-management-realtime-20261022/`다. 수정 전 모바일 입력 126개와 원본을 보존했으며 직전 검증본과 차이는 없었다. 제품 기존 15개 파일을 수정하고 집중 테스트 2개를 추가했다. 차이는 `product.diff`, 범위와 해시는 `source-change-summary.json`·`validation-sources.json`에 기록했다. 이전 원본·검증 근거를 삭제하지 않았다. 새 서버·DB·키 변경·의존성 설치·Git staging·commit·push는 수행하지 않았다.

中央の根拠は `.codex-workspace/verification/shared/mobile-management-realtime-20261022/`。変更前のモバイル入力 126 件と原本を保存しており、直前の検証版との差はなかった。既存の製品ファイル 15 件を変更し、集中テスト 2 件を追加した。差分は `product.diff`、範囲とハッシュは `source-change-summary.json`・`validation-sources.json` に記録した。以前の原本・検証根拠は削除していない。新たなサーバー・DB・キーの変更、依存関係のインストール、Git staging・commit・push は実行していない。
