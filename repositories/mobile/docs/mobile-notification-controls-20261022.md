# 모바일 전역 알림·방별 무음·안읽음과 앱 배지

# モバイル全体通知・ルーム別ミュート・未読とアプリバッジ

2026-10-22 사용자 승인에 따라 전역 알림 ON/OFF, 방별 무음, 메시지를 읽은 뒤 안읽음 수와 앱 아이콘 배지를 정리하는 후속 작업을 시작한다. 초기 기록 시각은 `2026-10-22T07:00:31Z`(16:00 KST)다. 이전 ID 초대 기능의 `USER_REPORTED_CONFIRMED`는 그 안내 범위로 유지하며 이번 신규 기능의 검증·수락으로 확대하지 않는다.

2026-10-22 のユーザー承認に従い、全体通知 ON/OFF、ルーム別ミュート、メッセージ既読後に未読数とアプリアイコンのバッジを整理する後続作業を開始する。初期記録時刻は `2026-10-22T07:00:31Z`（16:00 KST）である。以前の ID 招待機能の `USER_REPORTED_CONFIRMED` はその案内範囲に保ち、今回の新機能の検証・受け入れに拡大しない。

## 분담과 추적

## 分担と追跡

기존 [#46 기기 푸시 등록·권한과 전역/방 설정](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/46)에서 전역/방별 설정을, [#47 푸시 진입·badge와 복구](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/47)에서 안읽음·배지를 추적한다. 상위 계획은 [모바일 EPIC #52](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/52)이며 새 이슈를 중복 생성하지 않는다. 기존 본문·수동 체크·OPEN 상태·담당·라벨·프로젝트 상태를 보존한다.

既存の [#46 端末プッシュ登録・権限と全体 / ルーム設定](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/46) で全体 / ルーム別設定を、[#47 プッシュからの遷移・badge と復旧](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/47) で未読・バッジを追跡する。上位計画は [モバイル EPIC #52](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/52) であり、新規 issue を重複作成しない。既存本文・手動チェック・OPEN 状態・担当・ラベル・プロジェクト状態を保存する。

Mobile은 OS 권한과 서비스 설정을 구분하는 화면, 현재 계정의 설정 저장·복원, 서버 집계와 앱 표시·native badge의 동기화를 맡는다. CHAT는 수신 정책과 현재 읽음·안읽음의 권위 있는 상태를, BE는 기존 기기/세션 소유권 경계를 유지한다. 구체 API·집계·배지 계약은 담당의 현재 소스 조사로 확정하고 존재하지 않는 경로나 완료된 서버 기능을 가정하지 않는다.

Mobile は OS 権限とサービス設定を区別する画面、現在のアカウントの設定保存・復元、サーバー集計とアプリ表示・native badge の同期を担当する。CHAT は受信方針と現在の既読・未読の正とする状態を、BE は既存の端末 / セッション所有権境界を維持する。具体的な API・集計・バッジ契約は担当による現在のソース調査で確定し、存在しないパスやサーバー機能の完成を仮定しない。

## 초기 조사 인계와 이번 범위

## 初期調査の引き継ぎと今回の範囲

root의 착수 전 조사에서는 앱 설정에 기기 권한 허용·복구와 연결 상태가 표시되지만 전역 push ON/OFF·방별 mute의 화면/서비스는 미구현이었다. 알림 표시의 `shouldSetBadge`는 false이고 native `setBadgeCountAsync` 호출도 없다는 인계다. 이 문서 작성 과정에서 제품 소스·단말을 별도로 실행한 결과는 아니다.

root の着手前調査では、アプリ設定に端末権限の許可・復旧と接続状態が表示される一方、全体 push ON/OFF・ルーム別 mute の画面 / サービスは未実装だった。通知表示の `shouldSetBadge` は false で、native `setBadgeCountAsync` の呼び出しもないという引き継ぎである。これは本書作成中に製品ソースや端末を別途実行した結果ではない。

| 범위 | 구현·검수할 내용 | 초기 상태 |
| --- | --- | --- |
| 전역 설정 | OS permission과 별도인 서비스 ON/OFF, 현재 계정에 저장·재조회·재진입 복원, 서버 수신 정책과 일치 | 계약 조사·구현 진행, 검증 대기 |
| 방별 무음 | 현재 접근 가능한 방의 설정 저장·복원, 역할/참여·종료 변경과 계정 경계, 기존 기본값·OPEN opt-in 정책 유지 | 계약 조사·구현 진행, 검증 대기 |
| 안읽음 | 실제 읽음이 서버에 반영된 뒤 권위 있는 집계를 재조회·표시에 반영, 앱 복귀·재연결·동시 변경에서 보정 | 구현·검증 대기 |
| 아이콘 배지 | 정의한 집계와 native 표시 동기화, logout/계정 변경의 이전 수치 정리, OS·런처 지원 여부 처리 | 구현·검증 대기 |

| 範囲 | 実装・検査する内容 | 初期状態 |
| --- | --- | --- |
| 全体設定 | OS permission とは別のサービス ON/OFF、現在のアカウントへの保存・再取得・再入場時の復元、サーバー受信方針との一致 | 契約調査・実装中、検証待ち |
| ルーム別ミュート | 現在アクセスできるルームの設定保存・復元、役割 / 参加・終了変更とアカウント境界、既存デフォルト・OPEN opt-in 方針の維持 | 契約調査・実装中、検証待ち |
| 未読 | 実際の既読がサーバーに反映された後、正とする集計を再取得して表示に反映し、アプリ復帰・再接続・同時変更時に補正 | 実装・検証待ち |
| アイコンバッジ | 定義した集計と native 表示の同期、logout / アカウント変更時の以前の値の整理、OS・ランチャー対応の処理 | 実装・検証待ち |

전역·방별 알림 설정과 안읽음/배지 집계의 의미는 분리해 정한다. 무음 방을 집계에서 제외할지 등의 상세 정책을 화면 구현만으로 임의 변경하지 않는다. foreground의 REST 보정과 background 공급자 payload의 배지 책임 범위를 구분한다. 푸시 수신·알림 탭·background fetch만으로 메시지를 읽음 처리하지 않는다.

全体・ルーム別通知設定と未読 / バッジ集計の意味は分けて定める。ミュートしたルームを集計から除外するかなどの詳細方針を画面実装だけで勝手に変えない。foreground の REST 補正と background のプロバイダー payload のバッジ責任範囲を区別する。プッシュ受信・通知タップ・background fetch だけでメッセージを既読にしない。

## 검수 계획

## 検査計画

다음은 예정 조건이며 통과 체크가 아니다.

以下は予定条件であり、合格チェックではない。

- 전역 ON/OFF와 방별 무음의 저장·복원·웹/서버 일치, 기본값·미리보기 정책 보존을 확인한다. OS 권한 거절·재허용과 서비스 OFF를 구별한다.
- 저장 실패·결과 불명·늦은 응답·연속 조작·동시 기기 변경을 구분하며 오래된 응답이 최신 화면을 덮지 않게 한다.
- 실제 읽음 뒤 집계를 재조회해 방·전체 안읽음과 배지를 맞춘다. 재연결·복귀·중복/늦은 이벤트에서 이중 가감하지 않는다.
- 계정 전환·로그아웃·세션 만료 후 이전 계정의 설정·수치·응답·OS 효과가 새 계정에 적용되지 않는지 확인한다.
- OS와 런처가 배지를 지원하지 않는 경우를 실패·성공 주장과 구분한다. Android·iPhone의 실제 표시와 잠금/foreground/background 조건은 각각 별도 수락으로 남긴다.
- 기존 채팅 저장·읽음·안전한 알림 진입과 OPEN 초대 수신·미리보기·직접 참여 흐름의 회귀를 확인한다.

- 全体 ON/OFF とルーム別ミュートの保存・復元・ウェブ / サーバーとの一致、デフォルト・プレビュー方針の維持を確認する。OS 権限の拒否・再許可とサービス OFF を区別する。
- 保存失敗・結果不明・遅延応答・連続操作・別端末での同時変更を区別し、古い応答が最新画面を上書きしないようにする。
- 実際の既読後に集計を再取得し、ルーム・全体の未読とバッジを合わせる。再接続・復帰・重複 / 遅延イベントで二重加減算しない。
- アカウント切り替え・ログアウト・セッション期限切れ後、前のアカウントの設定・値・応答・OS 効果が新しいアカウントに適用されないことを確認する。
- OS とランチャーがバッジ非対応の場合を失敗・成功の主張から区別する。Android・iPhone の実表示とロック / foreground / background 条件は、それぞれ別の受け入れとして残す。
- 既存チャットの保存・既読・安全な通知遷移と OPEN 招待の受信・プレビュー・直接参加の回帰を確認する。

## 현재 검증·배포 상태와 보존

## 現在の検証・デプロイ状態と保存

착수 단계이며 이번 기능의 최종 자동 검사·실제 서버 정책·운영 배포·새 APK·실기기 결과는 아직 없다. 이전 기능의 시험 수·배포 완료·사용자 보고를 이번 변경의 결과로 복사하지 않는다. iPhone 실행과 Apple/Google 유료 등록은 진행하지 않는다.

着手段階であり、今回の機能の最終自動検査・実際のサーバー方針・本番デプロイ・新 APK・実機結果はまだない。以前の機能のテスト数・デプロイ完了・ユーザー報告を今回の変更の結果としてコピーしない。iPhone 実行と Apple / Google の有料登録は行わない。

중앙 근거는 `.codex-workspace/verification/shared/mobile-notification-controls-20261022/`이며 추적 근거는 `tracking/`에 둔다. 구현·검증 완료 후 root의 동결 소스·실행 로그·독립 검토·배포 담당 완료 영수증·APK 파일 검증을 기준으로 후속 상태를 추가한다. 배포는 지정된 「배포 책임과 규칙 기억하기」에서 승인 범위의 인계를 확인해 담당한다.

中央根拠は `.codex-workspace/verification/shared/mobile-notification-controls-20261022/`、追跡根拠は `tracking/` に置く。実装・検証完了後、root の凍結ソース・実行ログ・独立レビュー・デプロイ担当の完了記録・APK ファイル検証に基づいて後続状態を追記する。デプロイは指定されたチャット「배포 책임과 규칙 기억하기」で承認範囲の引き継ぎを確認して担当する。

이 문서·추적 작업은 제품 소스, Git staging/commit/push, 서버·DB·키·의존성·유료 서비스나 APK 빌드를 변경하지 않는다. 기존 원본·완료 영수증·사용자 확인 기록과 이번 초기 근거를 삭제하지 않는다.

この文書・追跡作業では製品ソース、Git staging / commit / push、サーバー・DB・鍵・依存関係・有料サービスや APK ビルドを変更しない。既存原本・完了記録・ユーザー確認記録と今回の初期根拠を削除しない。

## 구현·코드 검증 완료 · 2026-10-22 16:31 KST

## 実装・コード検証完了 · 2026-10-22 16:31 KST

앞 절의 착수·미구현·미검증 문구는 16:00 KST 초기 기록이다. 이후 전역/방별 설정과 안읽음·배지 연결을 구현했고 아래 코드 검증을 마쳤다. **운영 배포·새 APK 완료·Android/iPhone 물리 수락은 아직 확인되지 않았다.** 이번 문서 갱신에서 시험·빌드·배포를 새로 실행하지 않았다.

前節の着手・未実装・未検証という記述は 16:00 KST の初期記録である。その後、全体 / ルーム別設定と未読・バッジ接続を実装し、以下のコード検証を終えた。**本番デプロイ・新 APK 完成・Android / iPhone の物理端末での受け入れはまだ確認していない。** 今回の文書更新でテスト・ビルド・デプロイを新たに実行したわけではない。

### 확정한 계약과 동작

### 確定した契約と動作

| API | 동작 |
| --- | --- |
| GET/PUT `/chat/notifications/preferences` | 현재 계정의 `{pushEnabled:boolean}` 조회·설정. 계정의 모든 installation에 적용 |
| GET/PUT `/chat/rooms/{roomId}/notification-preferences` | 현재 계정이 접근 가능한 방의 `{roomId, muted:boolean}` 조회·설정 |

| API | 動作 |
| --- | --- |
| GET/PUT `/chat/notifications/preferences` | 現在のアカウントの `{pushEnabled:boolean}` を取得・設定。アカウントの全 installation に適用 |
| GET/PUT `/chat/rooms/{roomId}/notification-preferences` | 現在のアカウントがアクセスできるルームの `{roomId, muted:boolean}` を取得・設定 |

actor는 인증 principal에서만 얻고 요청 body에 정확한 boolean 하나만 허용한다. 방 설정은 현재 활성·미삭제 멤버/방을 요구하고 OPEN의 ACTIVE·미해제 ban 경계도 확인한다. 앱은 종료된 OPEN의 설정 card를 표시하지 않는다. 공개 prefix는 기존 `/api/v1` 경계를 따른다.

actor は認証 principal からのみ取得し、リクエスト body には厳密な boolean 1 個だけを許可する。ルーム設定には現在有効で未削除のメンバー / ルームを要求し、OPEN の ACTIVE・未解除 ban の境界も確認する。アプリは終了した OPEN の設定 card を表示しない。公開 prefix は既存の `/api/v1` 境界に従う。

전역 OFF는 모든 기기의 새 메시지와 OPEN 초대 푸시를 막는다. 방별 mute는 해당 방의 메시지 push만 막고 OPEN 초대는 전역 정책을 따른다. 대화·멤버십·초대함·읽음 cursor·안읽음 집계는 변경하지 않는다. row 없는 계정은 기존처럼 ON이고 기존/신규 member의 muted 기본값은 false다. 초기 백로그의 OPEN opt-in 계획을 이번 변경에서 새 기본 정책으로 적용한 것은 아니다.

全体 OFF は全端末の新着メッセージと OPEN 招待のプッシュを止める。ルーム別 mute は該当ルームのメッセージ push だけを止め、OPEN 招待は全体方針に従う。会話・メンバーシップ・招待ボックス・既読 cursor・未読集計は変更しない。row のないアカウントは従来どおり ON、既存 / 新規 member の muted デフォルトは false である。初期バックログの OPEN opt-in 計画を今回の変更で新しいデフォルト方針として適用したものではない。

전역 OFF→ON과 mute→unmute 때 DB UTC 시각을 cutoff로 저장해 그 이전에 만들어진 대기 job을 다시 보내지 않는다. 같은 값 PUT은 cutoff를 바꾸지 않는다. provider에 이미 넘긴 알림을 회수하거나 exactly-once 전달을 보장하지 않는다.

全体 OFF→ON と mute→unmute の際は DB の UTC 時刻を cutoff として保存し、それ以前に作られた待機 job を再送しない。同じ値の PUT は cutoff を変更しない。既に provider に渡した通知の取り消しや exactly-once 配信は保証しない。

안읽음·badge는 기존 `TotalAttentionCount = unreadChatMessageCount + unreadActivityCount` 의미를 그대로 쓴다. 방 수나 OPEN 초대 수를 다시 더하지 않고 mute/OFF로 집계를 지우지 않는다. 실제 읽음 성공 뒤 REST를 재조회하고 private read/notification 신호·복귀·재연결·30초 polling으로 보정한다. native 쓰기는 직렬화해 계정 전환·logout 뒤 이전 count가 남으면 0을 후속 적용한다.

未読・badge は既存の `TotalAttentionCount = unreadChatMessageCount + unreadActivityCount` の意味をそのまま用いる。ルーム数や OPEN 招待数を再加算せず、mute / OFF で集計を消さない。実際の既読成功後に REST を再取得し、private read / notification 信号・復帰・再接続・30 秒 polling で補正する。native 書き込みは直列化し、アカウント切り替え・logout 後に以前の count が残れば 0 を後続適用する。

0 집계로 알림창을 정리할 때는 **REST보다 먼저 캡처한 native 알림 snapshot**과 현재 identifier·eventId·binding이 모두 일치하는 항목만 삭제한다. 그 이후 새 알림과 identifier 재사용은 보존한다. 개별 메시지는 읽은 큰 문자열 ID cursor, 초대는 성공한 개별 read ID로 제한한다. foreground의 로컬 OFF/mute 보호는 global·방별로 독립적인 5초 동안만 적용해 다른 기기의 ON/unmute를 오래된 캐시로 계속 숨기지 않는다. 알림 handler에 새 네트워크 대기를 추가하지 않는다.

集計 0 で通知欄を整理する際は、**REST より前に取得した native 通知 snapshot** と現在の identifier・eventId・binding がすべて一致する項目だけを削除する。その後の新着通知と identifier の再利用は保護する。個別メッセージは既読にした大きな文字列 ID cursor、招待は成功した個別 read ID に限定する。foreground のローカル OFF / mute 保護は global・ルーム別で独立した 5 秒間だけ適用し、別端末の ON / unmute を古いキャッシュで隠し続けない。通知 handler に新たなネットワーク待機を追加しない。

서버는 provider 전송 전에 현재 정책·등록·binding·lease를 다시 확인하고 기존 요약으로 nullable badge를 추가한다. 요약 조회 실패는 0으로 위장하지 않고 badge만 생략한다. provider 도착·다른 기기의 읽음과 badge 전송이 원자적으로 맞는 보장은 없으므로 foreground REST 보정은 계속 필요하다.

サーバーは provider 送信前に現在の方針・登録・binding・lease を再確認し、既存要約から nullable badge を追加する。要約取得失敗を 0 に見せかけず、badge だけを省略する。provider 到着・別端末の既読と badge 送信が原子的に一致する保証はないため、foreground REST 補正は引き続き必要である。

### 동결 입력·실제 검사 범위

### 凍結入力・実際の検査範囲

| 범위 | 저장 결과와 제한 |
| --- | --- |
| 모바일 입력 | 최종 150핀·22변경. `mobile/validation-sources-final.json` SHA-256 `575c2646660e9cd905d18ce79f9e7ff856887dd7a825df6e08fb5004c095e02d` |
| 모바일 회귀 | 452/452 PASS, fail·skip 0. 마지막 관리 화면의 active/미종료 조건은 이 회귀 뒤 타입·집중 lint로 추가 확인했고 그 뒤 전체 452개를 재실행했다고 표시하지 않음 |
| 모바일 도구 | `typecheck-completed.log`, `lint-final-fixed.log`, `lint-closed-room.log`, 공개 HTTPS `config-device.log`, Android JS `export-android.log` PASS. JS export는 native 수락 증거가 아님 |
| CHAT 입력 | baseline `18f9283`에서 26파일 변경, 최종 350 source pin. `server/server-handoff.json`은 코드 검증·동결 인계이며 배포 완료 영수증 아님 |
| CHAT 회귀 | Unit 242/242·API 627/627·Integration 353/353 PASS, skip 0. 그 뒤 테스트만 추가한 이전 schema upgrade 1/1 별도 PASS. 전체 354개 한 번의 실행이나 추가 후 전체 재실행으로 합산하지 않음 |
| CHAT 도구 | build warnings/errors 0, pending model drift 0. 집중 API57·DB11은 전체와 겹치므로 더하지 않음 |
| 독립 검토 | 초기 두 P2(0 집계 뒤 새 알림 삭제·오래된 deny 캐시)를 수정. 현재 검토 범위의 새 P1/P2 0. 독립 순수 집중7개 PASS는 전체452와 중복 합산하지 않으며 전체 서버26파일 전수/native 검토로 확대하지 않음 |

| 範囲 | 保存結果と制限 |
| --- | --- |
| モバイル入力 | 最終 150 ピン・22 変更。`mobile/validation-sources-final.json` SHA-256 `575c2646660e9cd905d18ce79f9e7ff856887dd7a825df6e08fb5004c095e02d` |
| モバイル回帰 | 452/452 PASS、fail・skip 0。最後の管理画面の active / 未終了条件はこの回帰後に型・集中 lint で追加確認し、その後に全 452 件を再実行したとは記録しない |
| モバイルツール | `typecheck-completed.log`、`lint-final-fixed.log`、`lint-closed-room.log`、公開 HTTPS `config-device.log`、Android JS `export-android.log` PASS。JS export は native 受け入れの証拠ではない |
| CHAT 入力 | baseline `18f9283` から 26 ファイル変更、最終 350 source pin。`server/server-handoff.json` はコード検証・凍結引き継ぎで、デプロイ完了記録ではない |
| CHAT 回帰 | Unit 242/242・API 627/627・Integration 353/353 PASS、skip 0。その後テストだけを追加した旧 schema upgrade 1/1 は別途 PASS。全 354 件を 1 回実行した結果や、追加後の全件再実行として合算しない |
| CHAT ツール | build warnings / errors 0、pending model drift 0。集中 API57・DB11 は全体と重複するので加算しない |
| 独立レビュー | 初期の P2 2 件（集計 0 の後の新着通知削除・古い deny キャッシュ）を修正。現在のレビュー範囲の新規 P1 / P2 0。独立した純粋な集中 7 件 PASS は全体 452 と重複加算せず、サーバー全 26 ファイルの全数 / native レビューに拡大しない |

서버 migration `20261022071017_ChatNotificationPreferences`는 새 `chat_notification_preference` 테이블과 기존 `chat_room_member`의 `notification_muted`·`notification_enabled_since_utc` 두 컬럼을 추가한다. 기존 데이터·read cursor·notification payload·이전 migration을 삭제하지 않는다. MySQL의 ALTER는 metadata lock을 기다리거나 잡을 수 있으며 multi-DDL transaction의 완전 원자성·무중단을 가정하지 않는다. 배포 담당이 실제 schema·기존 transaction·권한·제한된 migration 결과를 확인해야 한다.

サーバー migration `20261022071017_ChatNotificationPreferences` は、新しい `chat_notification_preference` テーブルと既存 `chat_room_member` の `notification_muted`・`notification_enabled_since_utc` の 2 カラムを追加する。既存データ・read cursor・notification payload・以前の migration は削除しない。MySQL の ALTER は metadata lock を待機または取得し得るため、multi-DDL transaction の完全な原子性・無停止を仮定しない。デプロイ担当が実際の schema・既存 transaction・権限・限定的な migration 結果を確認する必要がある。

### 인계·APK와 남은 수락

### 引き継ぎ・APK と残る受け入れ

동결된 CHAT 인계를 「배포 책임과 규칙 기억하기」에 전달했다. 이 단계는 운영 결과 미확인이며 정확한 신규 commit·CI/runtime·migration·기존 공개 경로를 보존한 두 GET/PUT 경계 확인 영수증을 기다린다. 이번에는 BE·FE 소스 배포가 필요하지 않다.

凍結した CHAT の引き継ぎを「배포 책임과 규칙 기억하기」に送った。この段階では本番結果は未確認で、正確な新規 commit・CI / runtime・migration・既存公開パスを保存した 2 つの GET / PUT 境界確認記録を待つ。今回は BE・FE ソースのデプロイは必要ない。

무료 Android device build **1회**의 GO는 `apk/root-go.json`에 기록됐다. 기존 95ab7e46은 앞선 초대 기능 APK이며 이번 알림 제어 결과물로 바꾸어 부르지 않는다. 빌드 담당이 진행 중인 새 APK의 FINISHED·파일 검증·링크는 아직 기록하지 않는다. 유료 credits·중복 빌드·유료 개발자 등록을 요청하지 않았다.

無料 Android device build **1 回**の GO は `apk/root-go.json` に記録した。既存の 95ab7e46 は前の招待機能 APK であり、今回の通知制御成果物と呼び替えない。ビルド担当が進めている新 APK の FINISHED・ファイル検証・リンクはまだ記録しない。有料 credits・重複ビルド・有料開発者登録は要求していない。

새 APK와 운영 반영 뒤 실제 전역 OFF/ON, 방별 mute/unmute, 다른 계정 전환·logout, 읽음 후 탭/native badge 및 OS 알림창 정리를 사용자 기기에서 확인해야 한다. Android launcher별 숫자 배지 지원과 iPhone의 실제 실행은 별도 미검증 범위다. 이전 초대 기능의 사용자 총괄 확인과 과거 ASSUMED 기록은 그대로 보존한다.

新 APK と本番反映後に、実際の全体 OFF / ON、ルーム別 mute / unmute、別アカウントへの切り替え・logout、既読後のタブ / native badge と OS 通知欄の整理をユーザー端末で確認する必要がある。Android launcher ごとの数字バッジ対応と iPhone の実行は別途未検証範囲である。以前の招待機能に関するユーザーの総括確認と過去の ASSUMED 記録はそのまま保存する。

근거: `mobile/validation-sources-final.json`, `mobile/checks-manifest.json`, `server/server-handoff.json`, `server/checks-final.json`, `review-root.md`, `review-root-final.md`, `review/root-final-review.json`, `apk/root-go.json`, `tracking/`의 갱신 전후 이슈·문서 영수증. 초기 실패·중간 로그·원본·이전 완료 기록은 삭제하지 않는다. 문서·추적 담당의 제품 소스·Git·서비스·운영 DB·키·의존성 변경과 삭제는 없다.

根拠：`mobile/validation-sources-final.json`、`mobile/checks-manifest.json`、`server/server-handoff.json`、`server/checks-final.json`、`review-root.md`、`review-root-final.md`、`review/root-final-review.json`、`apk/root-go.json`、`tracking/` 内の更新前後の issue・文書記録。初期失敗・途中ログ・原本・以前の完了記録は削除しない。文書・追跡担当による製品ソース・Git・サービス・本番 DB・鍵・依存関係の変更と削除はない。

## 운영 반영·새 Android APK 완료 · 2026-10-22 17:04 KST

## 本番反映・新 Android APK 完了 · 2026-10-22 17:04 KST

앞선 착수·16:31 KST 절의 배포/APK 대기는 당시 이력으로 보존한다. 배포 담당 완료 영수증과 APK 담당 완료 영수증이 도착했고 root가 실제 APK 파일·동결 소스·배포 영수증을 직접 대조했다. **현재 상태는 운영/APK 완료, 이번 새 기능의 사용자 실기기 수락 대기**다. 아래 기록에서 앞선 452·242·627·353 및 별도 upgrade 1개의 실행 범위와 마지막 UI 조건의 별도 타입/lint 검사를 합치거나 다시 실행했다고 표시하지 않는다.

先の着手・16:31 KST 節のデプロイ / APK 待ちは当時の履歴として保存する。デプロイ担当の完了記録と APK 担当の完了記録が届き、root が実 APK ファイル・凍結ソース・デプロイ記録を直接照合した。**現在は本番 / APK 完了、今回の新機能に関するユーザー実機受け入れ待ち**である。以下の記録では、先の 452・242・627・353 と別途 upgrade 1 件の実行範囲、最後の UI 条件の別途型 / lint 検査を合算したり再実行したと記録したりしない。

### CHAT 운영 반영과 공개 경계

### CHAT 本番反映と公開境界

- CHAT `f17162a362692b8d8608d851402ea07d0fda5990`는 baseline `18f9283`에서 동결한 정확한 26파일만 커밋·푸시했다. [공식 run 39743172659](https://github.com/krumichan/CatPjt-TranslaCat-chat/actions/runs/39743172659)의 verify/release가 성공했고 배포 완료 기록은 **2026-10-22 16:56:11 KST**다. CHAT 작업 트리는 clean, 기존 FE 5파일의 hash·unstaged 상태는 보존했으며 이번 BE·FE 재배포와 비관련 변경은 없다.
- 운영 이미지는 `ghcr.io/krumichan/catpjt-translacat-chat@sha256:84e6ac3f5e1cd0fe1cff2fd11adda3c69d6efdf8b172942f047611e6d15cee3d`다. 같은 이미지의 migrator 뒤 앱을 교체했고 migration history **4→5**, 새 테이블 1개·기존 멤버 컬럼 2개와 기본값/키를 확인했다. 기존 데이터·schema·권한을 보존했고 pending migration 0, health **UP**, readiness **READY**, restart 0이다. 이전 컨테이너는 중지한 채 보존했다.
- 기존 공개 경로 45개를 보존한 채 설정 경계 2개만 추가해 47개가 됐다. GET/PUT `/api/v1/chat/notifications/preferences`와 GET/PUT `/api/v1/chat/rooms/{roomId}/notification-preferences`다. 외부 **20개 PASS**는 인증 없는 401·잘못된 경로 404·HEAD 405 등의 접근 경계 확인이다. 인증된 설정 PUT, provider 발송과 실제 기기 수신은 실행하지 않았다.
- 공개 경계를 열기 전에 새 runtime/schema/history/pending 0과 저장된 전역 설정 row 0·muted member 0을 두 집계로 확인했다. 개별 사용자·방 데이터를 조회한 검사가 아니며 해당 시점의 도입 조건일 뿐 이후에도 항상 0이라는 의미는 아니다. 제한된 완료 관찰 구간의 worker fixed marker는 메시지 0·초대 0으로, worker 실행이나 provider 전달 성공 증거로 확대하지 않는다.

- CHAT `f17162a362692b8d8608d851402ea07d0fda5990` は baseline `18f9283` から凍結した正確な 26 ファイルだけを commit・push した。[公式 run 39743172659](https://github.com/krumichan/CatPjt-TranslaCat-chat/actions/runs/39743172659) の verify / release が成功し、デプロイ完了記録は **2026-10-22 16:56:11 KST** である。CHAT 作業ツリーは clean、既存 FE 5 ファイルの hash・unstaged 状態は保存し、今回の BE・FE 再デプロイや無関係な変更はない。
- 本番イメージは `ghcr.io/krumichan/catpjt-translacat-chat@sha256:84e6ac3f5e1cd0fe1cff2fd11adda3c69d6efdf8b172942f047611e6d15cee3d` である。同じイメージの migrator 後にアプリを置き換え、migration history **4→5**、新規テーブル 1 個・既存メンバーカラム 2 個とデフォルト / キーを確認した。既存データ・schema・権限を維持し、pending migration 0、health **UP**、readiness **READY**、restart 0 である。以前のコンテナーは停止した状態で保存した。
- 既存公開パス 45 個を維持し、設定境界 2 個だけを追加して 47 個になった。GET / PUT `/api/v1/chat/notifications/preferences` と GET / PUT `/api/v1/chat/rooms/{roomId}/notification-preferences` である。外部 **20 件 PASS** は未認証の 401・誤パス 404・HEAD 405 などのアクセス境界確認である。認証済み設定 PUT、provider 送信と実端末受信は実行していない。
- 公開境界を開く前に、新 runtime / schema / history / pending 0 と保存済み全体設定 row 0・muted member 0 を 2 集計で確認した。個別ユーザー・ルームデータを取得した検査ではなく、その時点の導入条件にすぎず、その後も常に 0 という意味ではない。限定した完了観測区間の worker fixed marker はメッセージ 0・招待 0 であり、worker 実行や provider 配信成功の証拠に拡大しない。

실제 자동 app rollback flag는 **UNKNOWN** 상태를 유지했고 읽거나 변경한 것으로 기록하지 않는다. 첫 도입에서 새 공개 설정 경계를 닫아 둔 migration/swap 단계의 복구 조건을 평가한 것이며, 설정 경계를 연 뒤에는 저장된 설정·추가 schema를 보존하는 fix-forward를 우선한다. 부득이하게 예전 binary로 복구하려면 두 push worker 계열을 먼저 정지해야 한다. Down/Drop이나 비밀 변경은 하지 않았다. 사전 MySQL 권한·metadata lock 전부를 직접 본 것으로 확대하지 않으며 실제 같은 이미지 migration과 사후 schema/history 검증을 구분한다.

実際の自動 app rollback flag は **UNKNOWN** を維持し、読んだり変更したりしたと記録しない。初回導入で新しい公開設定境界を閉じた migration / swap 段階の復旧条件を評価したもので、設定境界を開いた後は保存設定・追加 schema を維持する fix-forward を優先する。やむを得ず以前の binary へ戻すなら、2 系統の push worker を先に停止する必要がある。Down / Drop や秘密情報の変更は行っていない。事前の MySQL 権限・metadata lock をすべて直接確認したという意味に拡大せず、実際の同イメージ migration と事後の schema / history 検証を区別する。

### APK 파일·소스와 root 대조

### APK ファイル・ソースと root の照合

[새 Android APK 2c0940d4-e9d2-45c5-88d2-bf90eed8cdd6](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/2c0940d4-e9d2-45c5-88d2-bf90eed8cdd6)는 device profile의 **FINISHED**, 완료 시각 **2026-10-22 16:53:58 KST**, 패키지 `com.krumichan.translacat.dev`다.

[新 Android APK 2c0940d4-e9d2-45c5-88d2-bf90eed8cdd6](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/2c0940d4-e9d2-45c5-88d2-bf90eed8cdd6) は device profile の **FINISHED**、完了時刻 **2026-10-22 16:53:58 KST**、package `com.krumichan.translacat.dev` である。

| 확인 범위 | 저장된 실제 결과 |
| --- | --- |
| 공식 APK 파일 | **93,545,175 bytes**, SHA-256 `e77bdf7fa4c384e16d1df14880fbc41e2de195b02ca8a7237f1cbda79cc94b71` |
| 동결 소스 | 최종 150핀 manifest `575c2646660e9cd905d18ce79f9e7ff856887dd7a825df6e08fb5004c095e02d`, 전후 drift 0 |
| 공개 설정·bundle | 공개 설정 7개 PASS, 기존 95ab와 다른 JS bundle 및 설정 API/native badge 문자열 확인. root 직접 읽은 bundle 2,691,384 bytes, SHA-256 `4554a165bc982ad03bb35df111acf64193eae3e4e6a392d49da4cf0cc0166533` |
| 기존 글자 잘림 보완 | compiled AppTheme/TextStyle·TranslaCatTextView/useBoundsForWidth=false 연결과 관련 resource table이 기존 95ab와 같은지 확인. 실제 화면 렌더 확인으로 확대하지 않음 |
| 실행 비용·횟수 | 무료 build 요청 1회, retry 0, 공식 파일 논리 다운로드 1회. 추가 유료 credits/개발자 등록·키/환경·의존성 변경 없음 |
| 미검증 범위 | native 실제 표시, 설치본과 cloud 전체 source archive의 완전 일치, 서명·binary manifest/서명 인증서 비교, 전체 archive CRC는 독립 검증하지 않음 |

| 確認範囲 | 保存した実結果 |
| --- | --- |
| 公式 APK ファイル | **93,545,175 bytes**、SHA-256 `e77bdf7fa4c384e16d1df14880fbc41e2de195b02ca8a7237f1cbda79cc94b71` |
| 凍結ソース | 最終 150 ピン manifest `575c2646660e9cd905d18ce79f9e7ff856887dd7a825df6e08fb5004c095e02d`、前後 drift 0 |
| 公開設定・bundle | 公開設定 7 項目 PASS、既存 95ab と異なる JS bundle および設定 API / native badge 文字列を確認。root が直接読んだ bundle は 2,691,384 bytes、SHA-256 `4554a165bc982ad03bb35df111acf64193eae3e4e6a392d49da4cf0cc0166533` |
| 既存文字切れの補正 | compiled AppTheme / TextStyle・TranslaCatTextView / useBoundsForWidth=false 接続と関連 resource table が既存 95ab と同じかを確認。実画面描画確認に拡大しない |
| 実行費用・回数 | 無料 build 要求 1 回、retry 0、公式ファイルの論理ダウンロード 1 回。追加有料 credits / 開発者登録・鍵 / 環境・依存関係の変更なし |
| 未検証範囲 | native 実表示、インストール済みアプリと cloud 全 source archive の完全一致、署名・binary manifest / 署名証明書比較、全 archive CRC は独立検証していない |

root 대조 시각은 **2026-10-22 16:59:28 KST**이며 상태는 `ROOT_APK_AND_DEPLOYMENT_VERIFIED_DEVICE_ACCEPTANCE_PENDING`다. 실제 파일 hash/크기·embedded bundle·text resource·소스 150핀 drift 0과 배포 완료 영수증 hash를 직접 확인했다. 이전 APK·실패/중간 로그·원본·테스트용 중지 volume/network·완료 영수증은 삭제하지 않는다.

root の照合時刻は **2026-10-22 16:59:28 KST**、状態は `ROOT_APK_AND_DEPLOYMENT_VERIFIED_DEVICE_ACCEPTANCE_PENDING` である。実ファイルの hash / サイズ・embedded bundle・text resource・ソース 150 ピン drift 0 とデプロイ完了記録の hash を直接確認した。以前の APK・失敗 / 途中ログ・原本・テスト用停止 volume / network・完了記録は削除しない。

### 새 APK에서 남은 사용자 수락

### 新 APK に残るユーザー受け入れ

다음은 아직 **PENDING_NEW_APK**이며 과거의 초대/푸시/탭 사용자 보고나 ASSUMED 기록을 대신 적용하지 않는다.

以下はまだ **PENDING_NEW_APK** であり、過去の招待 / プッシュ / タップに関するユーザー報告や ASSUMED 記録を代用しない。

1. 새 APK 업데이트 후 설정을 다시 열어 전역 OFF/ON 저장과 재진입 복원을 확인한다. 다른 계정에서 **새** 메시지·OPEN 초대를 보내 OFF에서는 push 억제, ON에서는 새 알림 수신을 확인한다. OFF 동안의 초대함·안읽음 집계 보존과 다시 켤 때 이전 대기 알림을 몰아 보내지 않는지 구분한다.
2. 한 방을 mute하고 다른 방과 대조해 해당 방의 새 메시지 push만 억제되는지 확인한다. unmute 이후 새 메시지를 사용하고 OPEN 초대의 전역 정책도 구분한다.
3. 실제 방/초대를 읽고 방·전체 안읽음 수와 앱 아이콘 badge·기존 알림창이 정리되는지 확인한다. 새 알림이 겹쳐 도착할 때 새 항목이 사라지지 않는지, 앱 복귀 뒤 집계가 맞는지도 확인한다.
4. 로그아웃/계정 전환 후 이전 계정의 count가 남지 않고 0 정리 뒤 현재 계정 집계로 수렴하는지 확인한다. Android 런처별 숫자 배지 지원은 별도 제약이며 iPhone 물리 실행·유료 등록은 하지 않았다.

1. 新 APK 更新後に設定を再度開き、全体 OFF / ON の保存と再入場時の復元を確認する。別アカウントから**新しい**メッセージ・OPEN 招待を送り、OFF では push 抑制、ON では新着通知受信を確認する。OFF 中の招待ボックス・未読集計の維持と、再 ON 時に以前の待機通知をまとめて送らないことを区別する。
2. 1 ルームを mute して別ルームと比較し、そのルームの新着メッセージ push だけが抑制されるか確認する。unmute 後は新しいメッセージを使い、OPEN 招待の全体方針も区別する。
3. 実際にルーム / 招待を読み、ルーム・全体の未読数とアプリアイコン badge・既存通知欄が整理されるか確認する。新着通知が重なって届いたとき新しい項目が消えないか、アプリ復帰後に集計が一致するかも確認する。
4. ログアウト / アカウント切り替え後に前アカウントの count が残らず、0 への整理後に現アカウント集計へ収束するか確認する。Android ランチャーごとの数字バッジ対応は別の制約で、iPhone の物理実行・有料登録は行っていない。

근거 파일 SHA-256:

根拠ファイルの SHA-256：

- `deployment/deployment-completed.json`: `f1c5bdd1fea4c3b4793ce8df2d7ed1aefad6dd046d1979e5f781caa1d28349e7`
- `apk/apk-completed.json`: `a3190ee644f936f1ed6b1fb203f38e428d8783075d437e1c5272a3bdd9c64d37`
- `root-delivery-verified.json`: `383c3e0baf1c5d01feda12598db3c25a8ed3244cfc5f57116194368fe06e311f`
- `first-cutover-recovery-assessment.json`, `deployment/rollback-stage-assessment.json`, `tracking/release-prepared.json` 및 최종 readback 영수증도 보존한다.

- `deployment/deployment-completed.json`: `f1c5bdd1fea4c3b4793ce8df2d7ed1aefad6dd046d1979e5f781caa1d28349e7`
- `apk/apk-completed.json`: `a3190ee644f936f1ed6b1fb203f38e428d8783075d437e1c5272a3bdd9c64d37`
- `root-delivery-verified.json`: `383c3e0baf1c5d01feda12598db3c25a8ed3244cfc5f57116194368fe06e311f`
- `first-cutover-recovery-assessment.json`、`deployment/rollback-stage-assessment.json`、`tracking/release-prepared.json` と最終 readback 記録も保存する。

이번 문서·추적 갱신은 기존 #46/#47 본문·수동 체크·OPEN·Project 상태를 변경하지 않는다. 제품 소스·Git·서버·DB·키·의존성·빌드/배포를 새로 실행하지 않았다.

今回の文書・追跡更新は既存 #46 / #47 の本文・手動チェック・OPEN・Project 状態を変更しない。製品ソース・Git・サーバー・DB・鍵・依存関係・ビルド / デプロイを新たに実行していない。

## 다음 APK 후속 · 메시지별 미읽음 참여자 표시 · 2026-10-22 17:48 KST

## 次 APK の追記 · メッセージ別の未読参加者表示 · 2026-10-22 17:48 KST

새 사진 피드백에서 앱의 받은 메시지 옆 미읽음 참여자 수가 숨겨지는 차이를 확인했다. 모바일 ChatRoomScreen의 내 메시지 한정 조건을 제거해 웹처럼 SYSTEM 이외의 발신·수신·AI 메시지에서 서버의 `unreadMemberCount > 0`일 때 표시한다. 0/null은 숨기고 숫자 1을 하드코딩하지 않는다. CHAT의 해당 값은 그 메시지를 아직 읽지 않은 회원 수이며 계정 전체 메시지/활동의 안읽음·앱 아이콘 badge와 다른 집계다. 사진별 관점·시점이 달라 사진 당시 정확한 값이 1이었다고 독립 검증한 것은 아니다.

新しい写真のフィードバックで、アプリの受信メッセージ横の未読参加者数が隠れる差を確認した。モバイル ChatRoomScreen の自分のメッセージ限定条件を外し、ウェブ同様 SYSTEM 以外の送信・受信・AI メッセージでサーバーの `unreadMemberCount > 0` の場合に表示する。0 / null は隠し、数字 1 をハードコードしない。CHAT のこの値は、そのメッセージをまだ読んでいない会員数であり、アカウント全体のメッセージ / 活動の未読・アプリアイコン badge とは異なる集計である。写真ごとに観点・時点が異なり、撮影時の正確な値が 1 だったと独立検証したわけではない。

이번 저장 회귀 `mobile-message-unread-display-20261022/tests.log`는 452/452 PASS다. 타입·lint 최종 실행 완료는 별도 근거로 구분한다. **앞 절에서 완료된 APK 2c0940d4에는 이 수정이 없으며 새 APK는 아직 없다.** 서버·FE·DB·운영 배포 변경은 필요 없고 유료 작업을 하지 않는다. 새 APK 반영 뒤 발신·수신·AI의 양수 표시와 0/null 숨김을 실제 기기에서 확인해야 한다. 이전 운영/APK 완료·실기기 수락 대기·사용자 확인과 ASSUMED 기록은 그대로 보존한다. 이번 기록 정리에서 제품 소스·Git·빌드·배포·검사를 새로 실행하지 않았다.

今回保存した回帰 `mobile-message-unread-display-20261022/tests.log` は 452/452 PASS である。型・lint 最終実行完了は別の根拠として区別する。**前節で完成した APK 2c0940d4 にはこの修正はなく、新 APK はまだない。** サーバー・FE・DB・本番デプロイ変更は不要で、有料作業は行わない。新 APK 反映後、送信・受信・AI の正の値の表示と 0 / null 非表示を実機で確認する必要がある。以前の本番 / APK 完了・実機受け入れ待ち・ユーザー確認と ASSUMED 記録はそのまま保存する。今回の記録整理で製品ソース・Git・ビルド・デプロイ・検査を新たに実行していない。

후속 최종 검사(2026-10-22 17:50 KST)는 같은 `ChatRoomScreen.tsx` 한 파일 수정에서 **452/452·typecheck·해당 화면 파일 lint 모두 exit 0**, 웹의 양수·비SYSTEM 공통 표시 정책에 대한 독립 정적 검토 PASS다. 변경 파일 SHA-256은 `a8599df59423ba4bcda6e069e172b5d4911593c1350eb68729d971f364d2cab5`이며 이전 150핀 중 나머지 drift는 0이다. 이는 정적/자동 검사이며 실제 폰 수락은 아니다. `validation-completed.json` SHA-256 `28ab6353a6d09c721e21613147868f2844e95faa2c894dc9eb287832034bc61c`와 `review/unread-final-parity-review.json`을 근거로 앞 문단의 타입/lint 대기를 갱신한다.

後続最終検査（2026-10-22 17:50 KST）は同じ `ChatRoomScreen.tsx` 1 ファイル修正に対し、**452/452・typecheck・当該画面ファイル lint がすべて exit 0**、ウェブの正の値・非 SYSTEM 共通表示方針に対する独立静的レビュー PASS である。変更ファイル SHA-256 は `a8599df59423ba4bcda6e069e172b5d4911593c1350eb68729d971f364d2cab5`、以前の 150 ピンの残りの drift は 0 である。静的 / 自動検査であり、実端末受け入れではない。`validation-completed.json` SHA-256 `28ab6353a6d09c721e21613147868f2844e95faa2c894dc9eb287832034bc61c` と `review/unread-final-parity-review.json` に基づいて前段の型 / lint 待ちを更新する。

새 조회 `apk/read-only-result.json`에서 Free Android 잔량 **0/15(사용 15/15)**를 확인했다. 이번 새 build 요청·source upload·artifact download·유료 작업은 0회이며 2c094 APK는 이 수정 미포함으로 유지한다. 다음 무료 APK에 묶기 또는 무료 로컬 Android 빌드 환경 준비의 사용자 선택을 기다린다. 서버·웹·DB·Git 변경은 없다. `tracking/confirmed.json`과 이전 문서를 보존하고 최종 추적 readback은 별도 `tracking/final-confirmed.json`에 둔다.

新たな照会 `apk/read-only-result.json` で Free Android 残量 **0/15（使用 15/15）**を確認した。今回の新規 build 要求・source upload・artifact download・有料作業は 0 回で、2c094 APK はこの修正を含まない状態を維持する。次の無料 APK にまとめるか、無料のローカル Android ビルド環境を準備するか、ユーザーの選択を待つ。サーバー・ウェブ・DB・Git の変更はない。`tracking/confirmed.json` と以前の文書を保存し、最終追跡 readback は別の `tracking/final-confirmed.json` に置く。

## 새 로컬 APK 사용자 확인 후속

## 新しいローカル APK のユーザー確認追記

사용자가 새 APK 업데이트 후 “받은 메시지에도 안 읽음 숫자 잘 보여”라고 직접 보고했다. Expo 설치 페이지 `b965582f-5822-4b59-8154-9ced4bd7f997` 안내 이후의 **업데이트 설치 및 받은 메시지의 양수 미읽음 표시 USER_REPORTED_PASS**로 기록한다. 근거는 `.codex-workspace/verification/shared/android-local-build-20261022/device-user-acceptance.json`이다. 기존 APK·로그·자동 검사 및 업로드 파일 해시 근거는 보존한다.

ユーザーは新しい APK 更新後に「受信メッセージにも未読数がよく見える」と直接報告した。Expo インストールページ `b965582f-5822-4b59-8154-9ced4bd7f997` 案内後の、**更新インストールおよび受信メッセージの正の未読数表示 USER_REPORTED_PASS** として記録する。根拠は `.codex-workspace/verification/shared/android-local-build-20261022/device-user-acceptance.json` である。既存 APK・ログ・自動検査およびアップロードファイルのハッシュ根拠は保存する。

사용자 폰의 설치 바이너리를 독립 식별하거나 모든 시나리오를 직접 조작한 것은 아니다. 이번 확인으로 전역 알림 OFF/ON, 방별 무음, 읽음 후 배지·알림창 정리, 계정 전환 및 iPhone까지 통과했다고 확대하지 않는다. 이전 절의 새 APK 미제공·수신 메시지 표시 수락 대기는 이 후속으로 갱신하며 나머지 검증 범위는 그대로 둔다.

ユーザー端末のインストール済みバイナリを独立識別したり、全シナリオを直接操作したりしたものではない。この確認を、全体通知 OFF / ON、ルーム別ミュート、既読後のバッジ・通知欄の整理、アカウント切り替えおよび iPhone まで合格したという意味に拡大しない。前節の新 APK 未提供・受信メッセージ表示の受け入れ待ちはこの追記で更新し、他の検証範囲はそのままにする。

## 알림 설정·방별 무음·배지 사용자 확인 후속

## 通知設定・ルーム別ミュート・バッジのユーザー確認追記

사용자는 새 로컬 APK 설치·수신 메시지 표시 확인에 이어 “전체 알림이랑 방별 무음, 배지까지 확인했어 문제 없어!”라고 보고했다. 이 세 항목은 USER_REPORTED_PASS로 갱신한다. 근거는 `android-local-build-20261022/notification-device-user-acceptance.json`이다. 이전 초기·배포·APK·자동 검사 기록은 보존한다. 계정 전환·iPhone·모든 경합 시나리오를 독립 검증한 것으로 확대하지 않는다.


ユーザーは新しいローカル APK のインストール・受信メッセージ表示確認に続けて「全体通知もルーム別ミュートもバッジも確認した。問題ない」と報告した。この 3 項目を USER_REPORTED_PASS に更新する。根拠は `android-local-build-20261022/notification-device-user-acceptance.json` である。以前の初期・デプロイ・APK・自動検査記録は保存する。アカウント切り替え・iPhone・すべての競合シナリオを独立検証したという意味に拡大しない。
