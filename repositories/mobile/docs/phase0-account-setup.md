# TranslaCat 모바일 계정 준비와 실기기 빌드 절차

> 日本語：TranslaCat モバイルのアカウント準備と実機ビルド手順

## 채팅 안정성 보완 APK 준비 완료와 휴대폰 수락 대기 (2026-10-21)

> 日本語：チャット安定性改善 APK の準備完了と端末での受け入れ待ち（2026-10-21）

[새 Android APK f076c80a](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/f076c80a-ea88-4486-bcfc-aa4d05cdc357)가 `2026-10-21T04:22:33.642Z`에 **FINISHED**됐다. 현재 상태는 **설치할 APK 준비 완료·새 휴대폰 수락 대기**다. 사용자의 이번 소스 업로드·무료 빌드 직접 승인 뒤 무료 `device` Android 빌드 1회와 동일 빌드 APK 다운로드 1회로 준비했다. 재빌드·유료 전환·키·환경 설정·서버·Git 변경은 없었다.

[新しい Android APK f076c80a](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/f076c80a-ea88-4486-bcfc-aa4d05cdc357) は `2026-10-21T04:22:33.642Z` に **FINISHED** となった。現在は **インストール用 APK の準備完了・新しい端末での受け入れ待ち**である。今回のソースアップロードと無料ビルドに対するユーザーの直接承認後、無料の `device` Android ビルド 1 回と同一ビルドの APK ダウンロード 1 回で準備した。再ビルド、有料プランへの変更、キー・環境設定・サーバー・Git の変更は行っていない。

이번 보완은 채팅 재연결과 읽음 동기화, 알림 탭의 재시도와 계정 전환 정리다. 동결 입력 **111개**로 **271/271개 테스트**, lint·타입·공개 HTTPS device config와 Android/iOS/web JS export가 모두 통과했다. APK는 **93,371,851 bytes**, SHA-256 `b604a037cde406968396fbd4e9e84f5a89a183bd7d2fabb56d9a6c98fbccc0fd`다. 내장 bundle·공개 설정 7개·이전 c768과 다른 bundle·로컬 동결 입력 일치를 확인했고, 별도 독립 대조와 전체 ZIP 1,345개 항목 CRC 검사도 통과했다.

今回の改善はチャット再接続と既読同期、通知タップの再試行とアカウント切り替え時の整理である。凍結した入力 **111 件**で **271/271 件のテスト**、lint・型検査・公開 HTTPS device config、Android/iOS/web の JS export がすべて成功した。APK は **93,371,851 bytes**、SHA-256 は `b604a037cde406968396fbd4e9e84f5a89a183bd7d2fabb56d9a6c98fbccc0fd`。内蔵 bundle・公開設定 7 件・従来の c768 とは異なる bundle・ローカル凍結入力との一致を確認し、別の独立照合と ZIP 全 1,345 項目の CRC 検査も成功した。

새 설치본에서 연결 끊김 뒤 복귀·읽음·계정 전환·알림 탭은 아직 휴대폰 확인이 필요하다. 서명·binary AndroidManifest·원격 전체 소스 바이트 일치와 iPhone 빌드·APNs·실기기 수락도 미검증이다. 이전 c768의 `USER_REPORT_PASS`는 그대로 유지하며 이번 APK의 성공으로 확대하지 않는다. 세부 검증과 오프라인 정리 한계는 [Phase 1 문서 상단](phase1-session-chat-integration.md)을 따른다.

新しいインストール版での切断後の復帰・既読・アカウント切り替え・通知タップは、まだ端末での確認が必要である。署名・binary AndroidManifest・リモートソース全体のバイト一致、iPhone ビルド・APNs・実機での受け入れも未検証である。以前の c768 の `USER_REPORT_PASS` は維持するが、今回の APK の成功に拡大解釈しない。検証の詳細とオフライン時の整理の制限は [Phase 1 文書の冒頭](phase1-session-chat-integration.md) に従う。

아래 업로드 승인 대기와 실행 전 차단은 당시 이력이다. 이후 이번 범위에 대한 사용자 직접 승인으로 재개했으며 이전 원문·거절·실패 진단은 삭제하지 않았다. 아래의 「현재」·「대기」는 각 기록 시점의 상태이며 이번 최상단 절이 최신 상태다.

以下のアップロード承認待ちと実行前のブロックは、その当時の履歴である。その後、今回の範囲に対するユーザーの直接承認で再開し、以前の原文・拒否・失敗診断は削除していない。以下の「現在」「待ち」は各記録時点の状態であり、この最上部の節が最新の状態を示す。

## 채팅 안정성 로컬 검증 완료와 업로드 승인 대기 (2026-10-21)

> 日本語：チャット安定性のローカル検証完了とアップロード承認待ち（2026-10-21）

채팅 재연결·읽음 동기화와 알림의 계정 경계를 보완했다. 동결 입력 111개로 **271/271개 테스트**, lint·타입·공개 HTTPS device config와 Android/iOS/web JS export가 모두 통과했고 검사 중 소스 변경은 없었다. 이번 보완에 서버·DB 배포는 필요하지 않다.

チャット再接続・既読同期と通知のアカウント境界を改善した。凍結入力 111 件で **271/271 件のテスト**、lint・型検査・公開 HTTPS device config、Android/iOS/web の JS export がすべて成功し、検査中のソース変更はなかった。今回の改善にサーバー・DB のデプロイは不要である。

현재 상태는 **외부 소스 업로드 명시 승인 대기·새 Android 실기기 수락 대기**다. 자동 승인 검토가 소스 업로드 대상·범위에 대한 직접 승인이 부족하다는 이유로 `request-build` 명령을 실행 전에 차단했다. 준비 가드와 읽기 전용 사전검증은 통과했지만 **업로드·빌드 요청·다운로드는 모두 0회이며 새 APK ID와 설치 링크는 없다.** 조회 시점의 무료 Android 잔여는 7회, 예상 비용은 0원이었다. 유효시간이 지난 사전검증은 재개 전에 갱신한다.

現在は **外部へのソースアップロードの明示的承認待ち・新しい Android 実機での受け入れ待ち**である。自動承認審査は、ソースのアップロード先と範囲に対する直接承認が不足しているとして、`request-build` を実行前にブロックした。準備ガードと読み取り専用の事前検証は成功したが、**アップロード・ビルド要求・ダウンロードはいずれも 0 回で、新しい APK ID とインストールリンクは存在しない。** 照会時点の無料 Android 枠の残りは 7 回、予想費用は 0 ウォンだった。有効期間を過ぎた事前検証は再開前に更新する。

재개 승인 범위는 manifest `0b3372540561f1c4d512d54c48b66785f7ebc1697fcc615ad4dfc100c1e66a4c`의 **111개 입력을 기존 Expo `kurumichans-team/translacat-mobile` 프로젝트 `55b79b33-07b9-4d4b-a3f8-cf24ea76b139`로 업로드해 무료 `device` Android APK 1개를 만들고, 완료된 동일 빌드의 APK를 1회 다운로드·검증하는 작업**이다. 승인 후 정상 검토 절차로 재개하며 결제·키·환경 설정·서버·Git 변경은 포함하지 않는다.

再開承認の範囲は、manifest `0b3372540561f1c4d512d54c48b66785f7ebc1697fcc615ad4dfc100c1e66a4c` の **111 件の入力を、既存の Expo `kurumichans-team/translacat-mobile` プロジェクト `55b79b33-07b9-4d4b-a3f8-cf24ea76b139` にアップロードして無料の `device` Android APK を 1 件作成し、完了した同一ビルドの APK を 1 回ダウンロード・検証する作業**である。承認後に通常の審査手順で再開し、決済・キー・環境設定・サーバー・Git の変更は含まない。

아래 c768은 사용자가 문제없음을 확인한 이전 설치본으로 유지한다. 이번 안정성 보완의 Android 네이티브 수락과 iPhone 빌드·APNs·실기기 확인은 완료되지 않았다. 세부 구현·검증과 오프라인 정리 한계는 [Phase 1 문서 상단](phase1-session-chat-integration.md)을 따른다. 아래의 「현재」·「대기」는 각 당시 기록이며 이번 최상단 절이 최신 상태다.

以下の c768 は、ユーザーが問題ないと確認した以前のインストール版として維持する。今回の安定性改善の Android ネイティブでの受け入れ、iPhone ビルド・APNs・実機確認は完了していない。実装・検証の詳細とオフライン整理の制限は [Phase 1 文書の冒頭](phase1-session-chat-integration.md) に従う。以下の「現在」「待ち」は当時の各記録で、この最上部の節が最新の状態である。

## 현재 키보드 최신 위치 보완 사용자 문제없음 확인 (2026-10-21)

> 日本語：キーボード表示時の最新位置改善についてユーザーが問題なしと確認（2026-10-21）

[새 Android APK c7681d83](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/c7681d83-a500-41cc-a931-3db93c81a5f1)가 `2026-10-20T22:57:41.402Z`에 **FINISHED**됐다. 사용자 승인 뒤 무료 빌드 요청 1회·다운로드 1회로 준비했고, 파일은 93,352,287 bytes다. 파일 구조·내장 번들·공개 설정과 로컬 동결 입력 109개 일치 및 별도 독립 대조를 완료했다. 현재 상태는 **`USER_REPORT_PASS`**다. 사용자는 c768 안내 후 “응 확인해봤는데 문제 없어 다음 작업 진행해줘”라고 보고했다. 정확한 설치 바이너리는 독립 확인하지 않았다.

[新しい Android APK c7681d83](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/c7681d83-a500-41cc-a931-3db93c81a5f1) は `2026-10-20T22:57:41.402Z` に **FINISHED** となった。ユーザー承認後、無料ビルド要求 1 回・ダウンロード 1 回で準備し、ファイルは 93,352,287 bytes。ファイル構造・内蔵 bundle・公開設定とローカル凍結入力 109 件との一致、および別の独立照合を完了した。現在の状態は **`USER_REPORT_PASS`**。c768 の案内後、ユーザーは「確認したけど問題ないので次の作業を進めて」と報告した。実際にインストールされたバイナリそのものは独立確認していない。

이번 보완은 키보드를 열 때 최신 메시지가 가려지는 문제다. 앱 소스 4개·테스트 2개를 변경하고 **218/218개 테스트**·lint·타입·config·세 플랫폼 JS export를 통과했다. Git·서버 설정·서비스 배포는 변경하지 않았다. 자동화 UI 확인은 당시 도구 시간 초과로 미실행이었다. 이후 사용자 문제없음 보고를 확인했으며, 이를 iOS나 모든 알림 상태의 성공으로 확대하지 않는다.

今回の改善は、キーボードを開くと最新メッセージが隠れる問題に対応するもの。アプリソース 4 件・テスト 2 件を変更し、**218/218 件のテスト**・lint・型検査・config・3 プラットフォームの JS export が成功した。Git・サーバー設定・サービスのデプロイは変更していない。UI の自動確認は当時のツールのタイムアウトで未実行だった。その後のユーザーによる問題なしの報告を確認したが、iOS やすべての通知状態の成功に拡大解釈しない。

이전 db061의 떨림·키보드 간격 해결과 이번 c768 확인은 각각의 사용자 보고로 보존한다. 파일 해시·검증 범위와 보고 한계는 [Phase 1 맨 위](phase1-session-chat-integration.md)를 따른다. 이번 사용자 원문과 이전109개 원문·문서·영수증은 중앙 `shared/mobile-chat-stability-20261021/`에 보존했다. 아래 과거 원문은 그대로 보존했다.

以前の db061 における揺れ・キーボード間隔の解消と、今回の c768 の確認は、それぞれのユーザー報告として保存する。ファイルハッシュ・検証範囲・報告の限界は [Phase 1 の冒頭](phase1-session-chat-integration.md) に従う。今回のユーザー原文と以前の 109 件の原文・文書・記録は、中央の `shared/mobile-chat-stability-20261021/` に保存した。以下の過去の原文もそのまま保存している。

## 현재 스크롤·키보드 보완 APK 준비 완료·휴대폰 수락 대기 (2026-10-21)

> 日本語：スクロール・キーボード改善 APK の準備完了と端末での受け入れ待ち（2026-10-21）

현재 상태는 **`READY_APK_PENDING_NATIVE_ACCEPTANCE`**다. [새 Android APK db061a2a](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/db061a2a-54f8-4fd7-8557-6d337b9ec7be)가 `2026-10-20T16:23:37.907Z`(2026-10-21 01:23:37 KST)에 **FINISHED**됐고 공식 Succeeded·Install 표시를 확인했다. 위 페이지의 Install로 기존 앱을 업데이트할 수 있다. 무료 빌드 요청 1회·다운로드 1회이며 자동 빌드 재요청은 0회다.

現在は **`READY_APK_PENDING_NATIVE_ACCEPTANCE`**。 [新しい Android APK db061a2a](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/db061a2a-54f8-4fd7-8557-6d337b9ec7be) が `2026-10-20T16:23:37.907Z`（2026-10-21 01:23:37 KST）に **FINISHED** となり、公式画面の Succeeded・Install 表示を確認した。上記ページの Install から既存アプリを更新できる。無料ビルド要求 1 回・ダウンロード 1 回で、ビルドの自動再要求は 0 回。

이전 699 안내본 사용 맥락에서 최신 이동 뒤 위로 스크롤하면 하단으로 끌려오는 떨림과 Android 키보드·입력창 사이 간격을 사용자 보고로 받았다. 정확한 설치 바이너리는 독립 확인하지 않았다. 자동 추적의 제스처 제어권, 실측 콘텐츠 높이 − 뷰포트 높이를 사용하는 긴 마지막 행의 하단 이동, 키보드 표시 중 bottom safe-area 중복 여백을 보완했다.

以前案内した 699 版の使用という文脈で、最新位置へ移動後に上へスクロールすると下へ引き戻されて揺れることと、Android キーボードと入力欄の隙間についてユーザー報告を受けた。実際のインストール済みバイナリは独立確認していない。自動追従時のジェスチャー制御権、実測コンテンツ高 − viewport 高を用いた長い最終行の末尾移動、キーボード表示中の bottom safe-area の重複余白を改善した。

새 **107개 입력**(`7e969566…`)으로 로컬 **202/202개 테스트**·lint·타입·공개 HTTPS config·Android/iOS/web JS export가 통과했다. 합성 fixture를 사용하는 실제 export 브라우저에서 최신 하단 간격 0px, 위로 이동한 뒤 간격 675px 유지, 첫 안 읽은 ID 75·모두 읽은 방 하단·작은 뷰포트를 확인했다. 이전 105개·190개 결과와 구분하며 웹 검증을 네이티브 키보드 성공으로 재사용하지 않는다.

新しい **107 件の入力**（`7e969566…`）で、ローカル **202/202 件のテスト**・lint・型検査・公開 HTTPS config・Android/iOS/web の JS export が成功した。合成 fixture を使う実際の export ブラウザーで、最新末尾との間隔 0px、上へ移動後の間隔 675px 維持、最初の未読 ID 75・全件既読の部屋の末尾・小さい viewport を確認した。以前の 105 件・190 件の結果と区別し、web 検証をネイティブキーボードの成功として再利用しない。

APK는 **93,350,575 bytes**, SHA-256 `7a96628e91e30e7a64d2ca377c9c581aae5ca5c3f3286f1a02adef3379f3ac78`다. ZIP 구조·내장 bundle·공개 설정 7개·이전 699와 다른 bundle·사후 로컬 107개 입력 일치를 확인했다. 완료 영수증 `apk/apk-completed.json`의 SHA-256은 `1cebfa791702c161a034ed6e5a90f48a4ef03ed31a8d46dc1dce746c8eb3265a`, 검증 영수증은 `6fade9ea46e849bd6d678a6a7de0be42d3d5f4eaaf7d7a4eb0c48b0808ce54ef`이며 중앙 `shared/mobile-scroll-keyboard-regression-20261021/`에 보존했다.

APK は **93,350,575 bytes**、SHA-256 は `7a96628e91e30e7a64d2ca377c9c581aae5ca5c3f3286f1a02adef3379f3ac78`。ZIP 構造・内蔵 bundle・公開設定 7 件・以前の 699 とは異なる bundle・事後のローカル入力 107 件との一致を確認した。完了記録 `apk/apk-completed.json` の SHA-256 は `1cebfa791702c161a034ed6e5a90f48a4ef03ed31a8d46dc1dce746c8eb3265a`、検証記録は `6fade9ea46e849bd6d678a6a7de0be42d3d5f4eaaf7d7a4eb0c48b0808ce54ef` で、中央の `shared/mobile-scroll-keyboard-regression-20261021/` に保存した。

**이번 설치본의 Android 업데이트·키보드 간격·터치 스크롤·읽던 위치·헤더 수락은 아직 대기**다. 서명·binary manifest·전체 ZIP CRC·원격 전체 아카이브 바이트 일치도 미검증이며, Metro 없는 실행 준비는 device/internal·developmentClient:false 설정과 내장 bundle에 근거한다. 실제 단독 실행과 Google 복귀·채팅·AI·푸시는 이번 APK에서 별도로 확인해야 한다. 자세한 검증·한계와 휴대폰 확인 순서는 [Phase 1 맨 위](phase1-session-chat-integration.md)를 따른다.

**今回のインストール版について、Android 更新・キーボード間隔・タッチスクロール・読んでいた位置・ヘッダーの受け入れはまだ待機中**。署名・binary manifest・ZIP 全体の CRC・リモートのアーカイブ全体のバイト一致も未検証で、Metro なしの起動準備は device/internal・developmentClient:false 設定と内蔵 bundle に基づく。実際の単独起動と Google からの復帰・チャット・AI・プッシュは、この APK で個別に確認する必要がある。検証・制限の詳細と端末での確認順序は [Phase 1 の冒頭](phase1-session-chat-integration.md) に従う。

이전 AI 답변·기본 로그인·다른 폰 알림/탭의 사용자 관찰, 같은 폰 알림 실패 원인 UNKNOWN, iPhone/APNs와 Apple Developer·Google Play 유료 등록 연기를 유지한다. 새 서버·DB·키·Git 작업은 하지 않았다. 아래 699·78b 및 101개·105개 입력 설명은 각 당시 기록이며 현재는 이 맨 위 절을 따른다. 전체 기존 문서는 이번 작업의 `docs/before/`와 `docs/final-ready/before/`에 바이트 그대로 보존했다.

以前の AI 回答・基本ログイン・別の端末での通知受信とタップについてのユーザー観察、同一端末での通知失敗原因 UNKNOWN、iPhone/APNs と Apple Developer・Google Play の有料登録延期は維持する。新しいサーバー・DB・キー・Git 作業は行っていない。以下の 699・78b と 101 件・105 件の入力の説明は各当時の記録であり、現在はこの最上部の節に従う。既存文書全体は、今回の作業の `docs/before/` と `docs/final-ready/before/` にバイトを変えずに保存した。

## 현재 v3 APK 준비 완료·휴대폰 수락 대기 (2026-10-20)

> 日本語：v3 APK の準備完了と端末での受け入れ待ち（2026-10-20）

키보드·읽던 위치·헤더 보완은105개 소스의 v3 입력(`fe858990…`)으로 동결했고 전체190/190·lint·타입·device 공개 HTTPS config·Android/iOS/web export를 통과했다. 합성 loopback API를 사용하는 실제 export 웹 화면에서 첫 안 읽은 위치·최신 이동·모두 읽은 방·작은 viewport의 입력창과 헤더를 확인했고, 조회용 GET 경로1개 적용과 외부39개·WSS 검사도 완료됐다.

キーボード・読んでいた位置・ヘッダーの改善を、105 ソースの v3 入力（`fe858990…`）として凍結し、全 190/190 件・lint・型検査・device の公開 HTTPS config・Android/iOS/web export が成功した。合成 loopback API を使う実際の export web 画面で、最初の未読位置・最新位置への移動・全件既読の部屋・小さな viewport の入力欄とヘッダーを確認し、照会用 GET 経路 1 件の適用と外部 39 件・WSS の検査も完了した。

[새 v3 Android APK69922399](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/69922399-1a82-4f2e-be6d-39e66c421b2f)가 `2026-10-20T14:30:31.934Z`에 FINISHED됐다. 공식 Install 버튼도 확인했으며 무료 빌드1회·다운로드1회다. APK는93,346,527 bytes, SHA-256 `a53b718497292eac017f981ee1595b9527570f518a5dc5717aa39757b69c45cf`다. ZIP·내장 bundle·공개 HTTPS API/환경/project/package/scheme/owner/slug 검사와 사후 로컬105개 소스 일치를 확인했다. 개발 채팅의 별도 독립 대조도 같은 APK 크기·SHA와105개 소스·내장 항목 존재의 일치를 확인했으며 추가 다운로드는0회다.

[新しい v3 Android APK69922399](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/69922399-1a82-4f2e-be6d-39e66c421b2f) が `2026-10-20T14:30:31.934Z` に FINISHED となった。公式の Install ボタンも確認し、無料ビルド 1 回・ダウンロード 1 回である。APK は 93,346,527 bytes、SHA-256 は `a53b718497292eac017f981ee1595b9527570f518a5dc5717aa39757b69c45cf`。ZIP・内蔵 bundle・公開 HTTPS API/環境/project/package/scheme/owner/slug の検査と、事後のローカル 105 ソースの一致を確認した。開発チャットでの別の独立照合も同じ APK サイズ・SHA と 105 ソース・内蔵項目の存在の一致を確認し、追加ダウンロードは 0 回だった。

현재 상태는 **설치할 APK 준비 완료·네이티브 수락 대기**다. 설치 후 키보드·읽던 위치·헤더/AI 메뉴를 확인해야 하며, 서명·binary manifest·원격 전체 아카이브 byte 일치·휴대폰 업데이트와 동작 성공은 아직 증명하지 않았다. Metro 없는 실행 준비는 device/internal·developmentClient:false 설정과 내장 bundle 근거이며 실제 단독 실행은 미검증이다. 상세 한계와 완료 영수증은 [Phase 1 맨 위](phase1-session-chat-integration.md)를 따른다.

現在は **インストール用 APK の準備完了・ネイティブでの受け入れ待ち**。インストール後にキーボード・読んでいた位置・ヘッダー/AI メニューを確認する必要があり、署名・binary manifest・リモートアーカイブ全体の byte 一致・端末更新と動作成功はまだ立証していない。Metro なしの起動準備は device/internal・developmentClient:false 設定と内蔵 bundle に基づき、実際の単独起動は未検証である。詳細な制限と完了記録は [Phase 1 の冒頭](phase1-session-chat-integration.md) に従う。

이전101개 UI/AI 입력·78b와 후속 v1/v2/v3의105개 입력은 시점별로 구분한다. 이전190개 PASS나78b 사용자 AI 답변 관찰을 현재 v3 APK의 휴대폰 성공으로 재사용하지 않는다. 사용자 AI 답변 보고·같은 폰 알림 실패 원인 UNKNOWN·iPhone/APNs와 Apple Developer/Google Play 유료 등록 연기는 유지한다. 아래는 각 과거 단계의 기록이다.

以前の 101 件の UI/AI 入力・78b と、その後の v1/v2/v3 の 105 件の入力は、時点ごとに区別する。以前の 190 件 PASS や 78b でユーザーが観察した AI 回答を、現在の v3 APK の端末動作成功として再利用しない。ユーザーの AI 回答報告・同一端末の通知失敗原因 UNKNOWN・iPhone/APNs と Apple Developer/Google Play の有料登録延期は維持する。以下は各過去段階の記録である。

## 현재 후속 보완과 AI 사용자 확인 (2026-10-20)

> 日本語：後続の改善とユーザーによる AI 確認（2026-10-20）

사용자는 안내받은78b 새 앱 업데이트 뒤 AI 실제 답변 도착을 확인했다. Android 사용자 보고 PASS이며 정확한 설치 바이너리 일치·모든 AI 권한/모드·iPhone 성공은 독립 확인하지 않았다. 이어 보고한 키보드 입력창 가림, 읽던 지점 대신 과거로 이동하는 방 진입, 제목·새로고침·메뉴 헤더를 후속 소스에서 보완했다.

ユーザーは案内した新しい 78b アプリへの更新後、AI の実際の回答が届いたと確認した。Android のユーザー報告は PASS だが、実際にインストールされたバイナリの一致・すべての AI 権限/モード・iPhone の成功は独立確認していない。続いて報告されたキーボードによる入力欄の隠れ、読んでいた位置でなく過去へ移動する入室、タイトル・更新・メニューのヘッダーを後続ソースで改善した。

동결105개 입력으로 전체190개·lint·타입·Expo config·Android/iOS/web export를 통과했다. **후속 APK 준비와 첫 안 읽은 위치 조회용 GET 공개 경로의 운영 적용은 대기 중이며, 새 키보드·읽던 위치의 네이티브 해결 완료로 표현하지 않는다.** 상세 동작과 근거는 [Phase 1 문서 맨 위](phase1-session-chat-integration.md)를 따른다. 이전 Android 로그인 복귀·다른 폰 알림/탭 성공과 같은 폰 실패 원인 UNKNOWN을 보존하고, iPhone·APNs와 Apple Developer/Google Play 유료 등록 연기는 유지한다. 아래 설명은 각 과거 단계의 기록이다.

凍結入力 105 件で全 190 件・lint・型検査・Expo config・Android/iOS/web export が成功した。**後続 APK の準備と、最初の未読位置を照会する GET 公開経路の本番適用は待機中であり、新しいキーボード・読んでいた位置のネイティブ問題が解決済みとは表現しない。** 詳細な動作と根拠は [Phase 1 文書の冒頭](phase1-session-chat-integration.md) に従う。以前の Android ログイン復帰・別の端末での通知とタップ成功、同一端末での失敗原因 UNKNOWN を保存し、iPhone・APNs と Apple Developer/Google Play の有料登録延期は維持する。以下の説明は各過去段階の記録である。

## 현재 앱 구현과 계정 준비의 구분 (2026-10-20)

> 日本語：アプリの実装とアカウント準備の区別（2026-10-20）

웹과 맞춘 모바일 UI, 친구 검색·요청과 새 DIRECT/GROUP 대화, OPEN방 생성, 기존 GROUP/OPEN의 AI 관리·멘션 초안을 구현했다. AI 전용 방을 새로 만든 것은 아니며 DIRECT방 AI 추가는 현재 API 범위 밖이다. 로컬180개·lint·타입·config와 첫 세 플랫폼 export, 제어된 브라우저 fixture 확인을 실제 휴대폰 신규 기능 성공과 구분한다. AI 프로필 수명 보완 후의 세 플랫폼 export도 통과했고, 공개 경로 운영 반영과 [새 Android APK78b05373](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/78b05373-b54e-48fe-a105-3442f8144fc0) 준비·공개 설정 검사를 완료했다. 새 기능의 실제 휴대폰·AI provider 수락은 다음 단계다. 이전 APK를 이번 기능 포함본으로 안내하지 않는다.

web に合わせたモバイル UI、友達検索・リクエストと新規 DIRECT/GROUP 会話、OPEN 部屋作成、既存 GROUP/OPEN の AI 管理・メンションの下書きを実装した。AI 専用部屋を新設したものではなく、DIRECT 部屋への AI 追加は現在の API 範囲外である。ローカル 180 件・lint・型検査・config と最初の 3 プラットフォーム export、制御されたブラウザー fixture の確認は、端末での新機能の成功と区別する。AI プロファイルのライフサイクル改善後の 3 プラットフォーム export も成功し、公開経路の本番反映と [新しい Android APK78b05373](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/78b05373-b54e-48fe-a105-3442f8144fc0) の準備・公開設定検査を完了した。新機能の端末・AI provider での受け入れは次の段階であり、旧 APK を今回の機能を含む版として案内しない。

최신 기능·검증·미검증 경계는 [Phase 1 문서 상단](phase1-session-chat-integration.md#모바일-ui새-대화ai-연결-구현-2026-10-20)을 따른다. 기존 Android 기본 로그인 복귀·다른 폰 이용 알림 수신·탭 성공은 사용자 보고로 보존하고 같은 폰 시험 실패의 원인은 단정하지 않는다. iPhone 서명·APNs·실기기 확인과 Apple Developer/Google Play 유료 등록 연기는 유지한다. 아래 계정 준비와 빌드·푸시 표는 각 기록 시점의 이력이다.

最新機能・検証済みと未検証の境界は [Phase 1 文書の冒頭](phase1-session-chat-integration.md#모바일-ui새-대화ai-연결-구현-2026-10-20) に従う。既存 Android の基本ログイン復帰・別の端末での通知受信・タップ成功はユーザー報告として保存し、同一端末での試験失敗原因は断定しない。iPhone 署名・APNs・実機確認と Apple Developer/Google Play の有料登録延期は維持する。以下のアカウント準備とビルド・プッシュの表は各記録時点の履歴である。

## 최신 Android 실기기 확인 (2026-10-20)

> 日本語：最新の Android 実機確認（2026-10-20）

Phase 0 이후 기존 서버 연동과 채팅 자동 알림을 배포했고, 사용자가 Android Google 로그인 자동 앱 복귀·다른 폰을 이용한 실제 채팅 알림 수신·탭 후 해당 방 이동을 확인했다. 이는 아래 Phase 0의 수동 테스트 푸시와 구분되는 최신 사용자 보고다. 성공 당시 앱 상태와 정확한 설치 APK는 명시되지 않았고 iPhone은 여전히 미검증이다. 같은 단말의 삼성 브라우저 테스트에서 알림이 없었던 앞선 관찰도 보존하며 원인을 단정하지 않는다.

Phase 0 後に既存サーバー連携とチャットの自動通知をデプロイし、ユーザーは Android の Google ログイン後のアプリへの自動復帰・別の端末を使った実際のチャット通知受信・タップ後の該当部屋への移動を確認した。これは以下の Phase 0 の手動テストプッシュとは区別される最新のユーザー報告である。成功時のアプリ状態と実際にインストールされた APK は明示されておらず、iPhone は引き続き未検証である。同一端末の Samsung ブラウザーでのテストで通知がなかった以前の観察も保存し、原因は断定しない。

다음은 앱에서 새 OPEN방과 DIRECT/GROUP 대화를 시작하는 화면 완성 및 상태별·계정 전환 실기기 확인이다. Apple/Play 유료 등록 연기 방침은 유지한다. 최신 상세 근거는 `phase1-session-chat-integration.md` 상단과 중앙 `shared/mobile-device-feedback-20261020/physical-acceptance-user-report-01a158bb/`다. 아래의 계정 준비·수동 푸시 표는 2026-10-18 Phase 0 당시 기록이다.

次はアプリで新しい OPEN 部屋と DIRECT/GROUP 会話を開始する画面の完成、および状態別・アカウント切り替えの実機確認である。Apple/Play の有料登録延期方針は維持する。最新の詳細な根拠は `phase1-session-chat-integration.md` の冒頭と中央の `shared/mobile-device-feedback-20261020/physical-acceptance-user-report-01a158bb/` にある。以下のアカウント準備・手動プッシュの表は、2026-10-18 の Phase 0 当時の記録である。


Android와 iPhone에 같은 TranslaCat 채팅 앱을 배포하기 위한 계정 준비 절차다. 현재 **Expo·Firebase 연결, 첫 Android 개발 APK 생성·휴대폰 설치·앱 화면 실행과 백그라운드 원격 푸시 수신을 확인**했다. 푸시는 사용자가 Expo 공식 도구에서 직접 전송하고 휴대폰 알림창에 도착했다고 확인한 결과다. 다음은 **기존 서버 로그인·채팅·기기 등록과 채팅 메시지의 자동 알림 연결**이다. Apple Developer 유료 등록은 사용자의 선택에 따라 앱의 핵심 기능이 준비된 뒤 iOS 실기기 빌드·푸시 검증을 시작할 시점에 진행한다. Google Play 유료 등록도 지금은 연기하고 스토어 테스트 배포 준비 때 진행한다.

Android と iPhone に同じ TranslaCat チャットアプリを配布するためのアカウント準備手順である。現在、**Expo・Firebase 接続、最初の Android 開発 APK の作成・端末へのインストール・アプリ画面起動と、バックグラウンドでのリモートプッシュ受信を確認**した。プッシュは、ユーザーが Expo 公式ツールで直接送信し端末の通知欄に届いたと確認した結果である。次は **既存サーバーのログイン・チャット・端末登録と、チャットメッセージの自動通知連携**。Apple Developer の有料登録はユーザーの選択により、アプリの主要機能が準備できてから iOS 実機ビルド・プッシュ検証を始める時点で行う。Google Play の有料登録も今は延期し、ストアのテスト配布を準備する時点で行う。

기준일은 2026년 10월 18일이다. 가입 유형과 결제는 계정 소유자가 직접 결정하고 진행한다. 계정 준비 상태는 아래 확인표로 관리하며, 코드 스캐폴드 준비와 실제 계정 연결 완료를 구분한다.

基準日は 2026 年 10 月 18 日。登録種別と決済はアカウント所有者が直接判断して実施する。アカウントの準備状況は以下のチェック表で管理し、コードのひな型の準備と実際のアカウント連携完了を区別する。

## 현재 상태와 준비할 정보

> 日本語：現在の状態と準備する情報

| 항목 | Phase 0 현재 상태 | 다음 확인 사항 |
| --- | --- | --- |
| Expo 계정과 소유자 | 로컬 CLI `Logged in`, `whoami`의 `kurumichan` 확인 완료 | 계정 접근 유지 |
| EAS 프로젝트 | `project:info`의 기존 owner·slug·UUID 일치 확인 완료 | 플랫폼별 자격증명과 빌드는 별도 준비 |
| Apple 개발자 계정 | 사용자 선택으로 유료 등록 연기 | 기능 준비 후 iOS 실기기 빌드·푸시 검증 전에 등록 |
| Google Play Console | 사용자 선택으로 유료 등록 연기 | 스토어 테스트 배포 전 등록·본인·기기 인증과 공개 출시 테스트 준비 |
| Firebase Android 설정 | `translacat-mobile-dev`의 개발 앱 등록, 설정 파일 일치 검사와 EAS development File 변수 생성·재조회 완료 | preview·production은 별도 앱/파일 준비 |
| FCM V1 발송 자격증명 | 개발 전용 계정·키 1개와 EAS 연결, `validate_only` HTTP 200 및 실제 Android 원격 수신 확인 | 채팅 서버의 자동 알림 발송 연결 |
| Android 개발 빌드 | development / INTERNAL APK 생성·휴대폰 설치·Metro 연결·앱 화면 실행 사용자 확인 | 실제 로그인·메시지와 후속 상태별 기기 검증 |
| iPhone 개발 빌드 | 유료 등록 연기에 따라 대기 | 개발자 팀·APNs·기기 등록 후 빌드·설치 |
| 실제 푸시 수신 | Android 백그라운드 원격 알림 도착을 사용자 확인 | 잠금·foreground·cold-start 별도 검증, 알림 클릭 방 이동과 iPhone |

日本語：

| 項目 | Phase 0 の現在の状態 | 次の確認事項 |
| --- | --- | --- |
| Expo アカウントと所有者 | ローカル CLI の `Logged in`、`whoami` の `kurumichan` を確認済み | アカウントへのアクセスを維持 |
| EAS プロジェクト | `project:info` の既存 owner・slug・UUID の一致を確認済み | プラットフォーム別の認証情報とビルドは別途準備 |
| Apple 開発者アカウント | ユーザーの選択で有料登録を延期 | 機能準備後、iOS 実機ビルド・プッシュ検証前に登録 |
| Google Play Console | ユーザーの選択で有料登録を延期 | ストアのテスト配布前に登録・本人/端末確認と公開リリースのテストを準備 |
| Firebase Android 設定 | `translacat-mobile-dev` の開発アプリ登録、設定ファイル一致検査、EAS development File 変数の作成・再照会を完了 | preview・production は別のアプリ/ファイルを準備 |
| FCM V1 送信認証情報 | 開発専用アカウント・キー 1 件と EAS 接続、`validate_only` HTTP 200 および実際の Android リモート受信を確認 | チャットサーバーの自動通知送信を接続 |
| Android 開発ビルド | development / INTERNAL APK 作成・端末インストール・Metro 接続・アプリ画面起動をユーザーが確認 | 実際のログイン・メッセージと後続の状態別端末検証 |
| iPhone 開発ビルド | 有料登録延期により待機 | 開発者チーム・APNs・端末登録後にビルド・インストール |
| 実際のプッシュ受信 | Android のバックグラウンドのリモート通知到着をユーザーが確認 | ロック・foreground・cold-start を個別検証、通知タップによる部屋移動と iPhone |

확인된 Expo 프로젝트는 다음과 같다. 2026년 10월 18일 로컬 CLI 로그인과 기존 프로젝트 접근을 확인했다. Android 개발 빌드·서명과 백그라운드 푸시 수신까지 확인했으며 iOS와 운영 배포 검증은 별도 단계다.

確認した Expo プロジェクトは以下のとおり。2026 年 10 月 18 日にローカル CLI ログインと既存プロジェクトへのアクセスを確認した。Android 開発ビルド・署名とバックグラウンドのプッシュ受信まで確認しており、iOS と本番デプロイの検証は別段階である。

| 항목 | 확인된 값 |
| --- | --- |
| Expo username | `kurumichan` |
| 프로젝트 owner | `kurumichans-team` |
| 프로젝트 slug | `translacat-mobile` |
| EAS projectId | `55b79b33-07b9-4d4b-a3f8-cf24ea76b139` |

日本語：

| 項目 | 確認した値 |
| --- | --- |
| Expo username | `kurumichan` |
| プロジェクト owner | `kurumichans-team` |
| プロジェクト slug | `translacat-mobile` |
| EAS projectId | `55b79b33-07b9-4d4b-a3f8-cf24ea76b139` |

[확인된 Expo 프로젝트](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile)를 계속 사용한다. 지금 직접 진행할 순서는 다음과 같다.

[確認済み Expo プロジェクト](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile) を引き続き使用する。現在、直接進める順序は次のとおり。

1. 로컬 EAS CLI 로그인과 기존 프로젝트 접근 확인은 완료했다. 아래 앱 식별자와 EAS 프로젝트 연결 절차는 다시 준비할 때 참고한다.
   <br>日本語：ローカル EAS CLI のログインと既存プロジェクトへのアクセス確認は完了した。以下のアプリ識別子と EAS プロジェクト接続の手順は、再準備時に参照する。
2. Firebase 개발 프로젝트 `TranslaCat Mobile Dev` / `translacat-mobile-dev`는 사용자가 생성했고 Spark 무료 플랜 화면을 확인했다. 소유자의 Google 권한 승인 후, 중앙 작업 폴더에만 설치한 공식 Firebase CLI `15.34.0`으로 인증·Android 앱 등록·설정 다운로드를 완료했다. 앱 내 브라우저의 조회 오류와 별개로 공식 CLI 등록 결과를 재조회했다. CLI 명령은 이 프로젝트와 확인한 계정을 명시한다. 로그인 명령에는 토큰을 포함할 수 있는 `--json` 출력을 사용하지 않는다. 별도 사용자 승인 후 개발 전용 FCM 계정·키와 EAS 연결, 전달 없는 검증 요청까지 완료했다.
   <br>日本語：Firebase 開発プロジェクト `TranslaCat Mobile Dev` / `translacat-mobile-dev` はユーザーが作成し、Spark 無料プラン画面を確認した。所有者の Google 権限承認後、中央作業フォルダーにのみ導入した公式 Firebase CLI `15.34.0` で認証・Android アプリ登録・設定ダウンロードを完了した。アプリ内ブラウザーの照会エラーとは別に、公式 CLI の登録結果を再照会した。CLI コマンドではこのプロジェクトと確認済みアカウントを明示する。ログインコマンドでは、トークンを含む可能性がある `--json` 出力を使わない。別途のユーザー承認後、開発専用 FCM アカウント・キーと EAS 接続、配信を伴わない検証要求まで完了した。
3. 앱 기능을 준비하는 동안 무료 Apple Account의 본인 명의·이중 인증을 미리 점검할 수 있다. 유료 Developer 등록은 iOS 실기기 검증 전까지 연기한다.
   <br>日本語：アプリ機能の準備中に、無料 Apple Account の本人名義・2 ファクタ認証をあらかじめ確認できる。有料 Developer 登録は iOS 実機検証の前まで延期する。
4. Play Console 결제를 진행할 시점을 정하고 공개 출시 전에 필요한 비공개 테스터를 준비한다. 가입만으로 테스트 기간이 시작되는 것은 아니며 테스트용 앱이 준비된 뒤 참여 조건을 충족해야 한다.
   <br>日本語：Play Console の支払い時期を決め、公開リリース前に必要な非公開テスターを準備する。登録だけではテスト期間は始まらず、テスト用アプリが準備できてから参加条件を満たす必要がある。

계정 가입·계약·결제는 모두 소유자가 공식 화면에서 직접 진행한다. 자동 가입·자동 결제는 수행하지 않는다.

アカウント登録・契約・決済はすべて所有者が公式画面で直接行う。自動登録・自動決済は実施しない。

개인 계정 로그인에 쓰는 이메일은 동일한 Gmail로 맞춰도 된다. 다만 Expo 계정, Google 계정, Apple Account는 각각 별도의 계정이며 가입·복구·이중 인증도 별도로 관리한다. iPhone 사용자의 개인 Apple Account를 개발자 소유자 계정으로 사용할 필요는 없다. iPhone 사용자는 자신의 계정을 유지하고 테스트 기기로 참여한다.

個人アカウントのログイン用メールを同じ Gmail にそろえてもよい。ただし Expo アカウント、Google アカウント、Apple Account はそれぞれ別のアカウントで、登録・復旧・2 ファクタ認証も個別に管理する。iPhone 利用者個人の Apple Account を開発者所有者のアカウントにする必要はない。iPhone 利用者は自分のアカウントを維持してテスト端末として参加する。

비밀번호·이중 인증 코드·복구 코드·결제 정보·신분증·Apple 개인 키·Google 서비스 계정 JSON·Expo access token을 채팅이나 GitHub 이슈에 붙이지 않는다. 가입 완료 여부와 Expo username 같은 식별 정보만 공유한다. 인증이 필요하면 소유자가 자신의 브라우저나 로컬 터미널에서 입력한다.

パスワード・2 ファクタ認証コード・復旧コード・決済情報・身分証・Apple 秘密鍵・Google サービスアカウント JSON・Expo access token をチャットや GitHub issue に貼り付けない。登録完了の有無や Expo username などの識別情報のみ共有する。認証が必要な場合は所有者が自分のブラウザーやローカル端末に入力する。

## 1 Expo 계정 만들기

> 日本語：1 Expo アカウントの作成

1. [Expo 가입](https://expo.dev/signup)에서 본인이 관리할 이메일과 username으로 계정을 만든다. 이미 있다면 새로 만들지 않고 [로그인](https://expo.dev/login)한다.
   <br>日本語：[Expo 登録](https://expo.dev/signup) で本人が管理するメールと username を使ってアカウントを作成する。すでにある場合は新規作成せず [ログイン](https://expo.dev/login) する。
2. dashboard의 계정 메뉴에서 username을 확인한다. 개인 프로젝트면 이 계정이 owner가 된다. 팀 공동 소유가 필요하면 Organization을 만들고 그 slug를 owner로 정한다. 개인 계정 비밀번호를 공동 사용하지 않는다. [Expo 계정 유형](https://docs.expo.dev/accounts/account-types/)
   <br>日本語：dashboard のアカウントメニューで username を確認する。個人プロジェクトではこのアカウントが owner になる。チームの共同所有が必要なら Organization を作成してその slug を owner とする。個人アカウントのパスワードは共同利用しない。[Expo アカウント種別](https://docs.expo.dev/accounts/account-types/)
3. 개인 계정 설정에서 인증 앱으로 이중 인증을 설정하고 복구 코드는 개인 보관소에 저장한다. [Expo 이중 인증](https://docs.expo.dev/accounts/two-factor/)
   <br>日本語：個人アカウント設定で認証アプリによる 2 ファクタ認証を設定し、復旧コードを個人保管庫に保存する。[Expo 2 ファクタ認証](https://docs.expo.dev/accounts/two-factor/)
4. 개발자에게 **Expo username과 프로젝트를 둘 owner 이름**을 알려준다. 계정 이메일 전체나 비밀번호는 필요하지 않다.
   <br>日本語：開発者には **Expo username とプロジェクトを置く owner 名**を伝える。アカウントの完全なメールアドレスやパスワードは不要である。

EAS Build는 무료 플랜에서도 사용할 수 있다. 빌드 큐·사용량·한도를 보고 유료 플랜 필요 여부를 나중에 결정한다. 스토어 개발자 등록 비용은 Expo 요금과 별개다. [EAS Build 준비](https://docs.expo.dev/build/setup/)

EAS Build は無料プランでも利用できる。ビルド待ち行列・使用量・上限を確認して有料プランが必要か後で判断する。ストア開発者登録の費用は Expo 料金とは別である。[EAS Build の準備](https://docs.expo.dev/build/setup/)

완료 기준: 본인이 Expo dashboard에 로그인할 수 있고, 사용할 owner 이름이 확정되어 있다.

完了基準：本人が Expo dashboard にログインでき、使用する owner 名が確定している。

## 2 Apple Account와 개발자 등록

> 日本語：2 Apple Account と開発者登録

현재 선택은 **유료 등록 연기**다. 이 절차는 핵심 기능 준비 후 iOS 실기기 빌드와 푸시 검증에 들어갈 때 진행한다. 그 전까지 코드 개발과 Android 준비를 계속할 수 있으며, iPhone의 실제 설치·원격 푸시 성공은 미검증 상태로 남긴다.

現在の選択は **有料登録の延期**。この手順は主要機能の準備後に iOS 実機ビルドとプッシュ検証を始める時点で実施する。それまではコード開発と Android 準備を継続でき、iPhone の実際のインストール・リモートプッシュ成功は未検証として残す。

### Apple Account 준비

> 日本語：Apple Account の準備

1. 이미 본인 명의 Apple Account가 있다면 우선 사용 여부를 확인한다. 없는 경우 [Apple Account](https://account.apple.com/)의 계정 생성에서 본인이 쓰는 Gmail을 기본 이메일로 사용할 수 있다. Apple은 웹 계정 생성 시 이메일과 전화번호를 확인한다. [Apple Account 생성](https://support.apple.com/en-us/108647)
   <br>日本語：本人名義の Apple Account がすでにあれば、まず使用するか確認する。ない場合は [Apple Account](https://account.apple.com/) のアカウント作成で、本人の Gmail を主メールにできる。Apple は web アカウント作成時にメールと電話番号を確認する。[Apple Account の作成](https://support.apple.com/en-us/108647)
2. 법적 이름·생년월일·지속해서 접근할 수 있는 전화번호를 입력하고 이중 인증을 켠다. 개발자 등록에는 이중 인증과 본명 사용이 필요하다. [Apple 개발자 등록](https://developer.apple.com/programs/enroll/)
   <br>日本語：法的な氏名・生年月日・継続して利用できる電話番号を入力し、2 ファクタ認証を有効にする。開発者登録には 2 ファクタ認証と本名の使用が必要である。[Apple 開発者登録](https://developer.apple.com/programs/enroll/)

### 개인과 조직 중 선택

> 日本語：個人と組織の選択

| 가입 유형 | 선택 기준과 공개 판매자 이름 |
| --- | --- |
| Individual | 개인 또는 1인 사업자로 등록한다. App Store 판매자 이름에 본인의 법적 이름이 표시된다. 앱 표시 이름 TranslaCat과 판매자 이름은 별개다. |
| Organization | 법적 실체가 있는 조직으로 등록한다. 조직 이름이 판매자 이름이 되며, 계약 권한·D-U-N-S 번호·조직 도메인 이메일·작동하는 공개 웹사이트 등이 필요하다. |

日本語：

| 登録種別 | 選択基準と公開される販売者名 |
| --- | --- |
| Individual | 個人または個人事業主として登録する。App Store の販売者名に本人の法的氏名が表示される。アプリ表示名 TranslaCat と販売者名は別である。 |
| Organization | 法的実体を持つ組織として登録する。組織名が販売者名になり、契約権限・D-U-N-S 番号・組織ドメインのメール・稼働する公開 web サイトなどが必要である。 |

상표명만 보여주기 위해 개인 이름 칸에 TranslaCat을 입력하지 않는다. 개인 본명 공개를 원하지 않으면 결제 전에 실제 조직으로 등록할 조건을 확인한다. Apple Developer Program은 **연간 99 USD**이며 지역별 표시 통화와 금액은 가입 화면을 따른다. [Apple 가입 조건과 비용](https://developer.apple.com/programs/enroll/)

商標名のみを表示する目的で個人名欄に TranslaCat を入力しない。本名の公開を避けたい場合は、支払い前に実際の組織として登録する条件を確認する。Apple Developer Program は **年額 99 USD** で、地域別の表示通貨と金額は登録画面に従う。[Apple 登録条件と費用](https://developer.apple.com/programs/enroll/)

### 등록 진행

> 日本語：登録の進め方

1. [Apple Developer 등록 시작](https://developer.apple.com/enroll/)에서 소유자 본인의 Apple Account로 진행한다.
   <br>日本語：[Apple Developer 登録開始](https://developer.apple.com/enroll/) から所有者本人の Apple Account で進める。
2. 유형을 선택하고 법적 정보·연락처·신원 확인 절차를 완료한다. Apple이 요청하는 신분 증빙은 Apple의 공식 화면으로만 제출한다. [신원 확인](https://developer.apple.com/help/account/membership/identity-verification/)
   <br>日本語：種別を選び、法的情報・連絡先・本人確認の手順を完了する。Apple が要求する身分証明は Apple の公式画面にのみ提出する。[本人確認](https://developer.apple.com/help/account/membership/identity-verification/)
3. 본인이 계약을 확인하고 결제한다. 등록 후 Developer 계정과 [App Store Connect](https://appstoreconnect.apple.com/)에 접근 가능한지 확인한다.
   <br>日本語：本人が契約内容を確認して支払う。登録後に Developer アカウントと [App Store Connect](https://appstoreconnect.apple.com/) にアクセスできるか確認する。
4. 개발자에게 **개인 또는 조직 유형, 승인 여부, 해당 팀 사용 가능 여부**만 알려준다. 이중 인증은 필요할 때 로컬에서 본인이 진행한다.
   <br>日本語：開発者には **個人/組織の種別、承認の有無、そのチームを利用できるか**のみ伝える。2 ファクタ認証は必要時に本人がローカルで行う。

완료 기준: 유료 멤버십이 활성화되고 올바른 팀으로 Developer 계정에 접근할 수 있다. iPhone 푸시 개발 빌드 자격증명을 생성하려면 유료 Apple Developer 계정이 필요하다. 이후 테스트할 iPhone을 등록하고 APNs 키·서명·프로비저닝을 연결한다. [Expo iOS 푸시 빌드 준비](https://docs.expo.dev/push-notifications/push-notifications-setup/)

完了基準：有料メンバーシップが有効で、正しいチームとして Developer アカウントにアクセスできる。iPhone のプッシュ開発ビルドの認証情報を作成するには有料 Apple Developer アカウントが必要である。その後、テスト対象の iPhone を登録し、APNs キー・署名・プロビジョニングを接続する。[Expo iOS プッシュビルドの準備](https://docs.expo.dev/push-notifications/push-notifications-setup/)

## 3 Google Play Console 등록

> 日本語：3 Google Play Console 登録

등록비 결제는 사용자가 진행할 시점을 선택한다. 지금 결제하지 않아도 개발 APK로 Android 기기 검증을 시작할 수 있다. 공개 출시 일정에는 계정 인증과 비공개 테스트·승인 기간을 포함한다.

登録費を支払う時期はユーザーが選ぶ。今支払わなくても開発 APK で Android 端末検証を開始できる。公開リリースの予定には、アカウント確認と非公開テスト・承認期間を含める。

1. [Google Play Console 가입](https://play.google.com/console/signup)에 본인이 관리하는 Google 계정으로 들어간다.
   <br>日本語：[Google Play Console 登録](https://play.google.com/console/signup) に本人が管理する Google アカウントでアクセスする。
2. 개인 또는 조직 계정을 실제 신분에 맞춰 선택한다. 개인으로 진행한다면 필요한 법적 이름·주소·연락처·신원 확인 정보를 입력하고 개발자 계약을 확인한다.
   <br>日本語：実際の身分に合わせて個人または組織アカウントを選ぶ。個人の場合、必要な法的氏名・住所・連絡先・本人確認情報を入力し、開発者契約を確認する。
3. 본인이 **최초 1회 25 USD** 등록비를 결제한다. Google이 요구하는 신원·이메일·전화번호 등 계정 확인을 완료한다. 공개 개발자 프로필에 표시될 항목은 저장 전에 화면에서 확인한다. [Play Console 시작](https://support.google.com/googleplay/android-developer/answer/6112435?hl=en)
   <br>日本語：本人が **初回 1 回限り 25 USD** の登録費を支払う。Google が求める本人・メール・電話番号などのアカウント確認を完了する。公開開発者プロフィールに表示される項目は保存前に画面で確認する。[Play Console の開始](https://support.google.com/googleplay/android-developer/answer/6112435?hl=en)
4. 신규 개인 계정의 홈에서 Android 기기 접근 확인 작업을 연다. QR 코드로 공식 Play Console 모바일 앱을 열고, **가입 소유자와 같은 Google 계정**으로 로그인해 확인한다. 비루팅 실물 Android 10 이상 기기를 사용할 수 있다. [기기 인증 절차](https://support.google.com/googleplay/android-developer/answer/14316361?hl=en)
   <br>日本語：新しい個人アカウントのホームから Android 端末へのアクセス確認タスクを開く。QR コードで公式 Play Console モバイルアプリを開き、**登録した所有者と同じ Google アカウント**でログインして確認する。root 化されていない実物の Android 10 以上の端末を利用できる。[端末認証手順](https://support.google.com/googleplay/android-developer/answer/14316361?hl=en)
5. 비공개 테스트에 참여할 사람을 미리 확보한다. 2023년 11월 13일 이후 생성된 개인 계정은 **최소 12명이 14일 연속 참여 상태를 유지하는 비공개 테스트**를 거친 다음 production access를 신청해야 한다. 테스트 피드백과 수정 기록도 남긴다. 조건 충족 후에도 승인은 별도 심사다. [신규 개인 계정 테스트 조건](https://support.google.com/googleplay/android-developer/answer/14151465?hl=en)
   <br>日本語：非公開テストに参加する人を事前に確保する。2023 年 11 月 13 日以降に作成された個人アカウントは、**最低 12 人が 14 日間連続で参加状態を保つ非公開テスト**を経てから production access を申請する必要がある。テストのフィードバックと修正記録も残す。条件を満たした後も承認は別途審査される。[新規個人アカウントのテスト条件](https://support.google.com/googleplay/android-developer/answer/14151465?hl=en)

개발 초기에는 소수 인원으로 직접 설치하거나 내부 테스트를 할 수 있지만 이것이 12명 비공개 테스트 조건을 대체하지는 않는다. 두 사람의 Android와 iPhone만으로 기능 개발은 시작할 수 있어도, 신규 개인 Play 계정의 공개 출시에 필요한 테스터는 별도로 준비해야 한다.

開発初期には少人数による直接インストールや内部テストができるが、12 人の非公開テスト条件の代わりにはならない。2 人の Android と iPhone だけでも機能開発は開始できるが、新しい個人 Play アカウントの公開リリースに必要なテスターは別途準備する必要がある。

완료 기준: Play Console에 소유자로 접근 가능하고 본인·기기 인증이 완료되어 있다. 공개 출시를 위해서는 이후 앱 설정·비공개 테스트·production access 신청·스토어 심사를 추가로 완료한다.

完了基準：Play Console に所有者としてアクセスでき、本人・端末の確認が完了している。公開リリースには、その後アプリ設定・非公開テスト・production access 申請・ストア審査を追加で完了する。

## 4 앱 식별자와 EAS 프로젝트 연결

> 日本語：4 アプリ識別子と EAS プロジェクトの接続

작업 폴더는 `C:\workspace\project-cat\CatPjt-TranslaCat-mobile`이다. **개발 잠정 Android 및 iOS 식별자는 `com.krumichan.translacat.dev`**이며 정식 출시 식별자가 아니다. 실제 소유자와 이름을 확정하기 전에는 production 앱을 등록하거나 임의의 예시 ID로 빌드하지 않는다.

作業フォルダーは `C:\workspace\project-cat\CatPjt-TranslaCat-mobile`。**開発用の暫定 Android/iOS 識別子は `com.krumichan.translacat.dev`** で、正式リリースの識別子ではない。実際の所有者と名称が確定する前に production アプリを登録したり、任意のサンプル ID でビルドしたりしない。

| 설정 | 의미와 입력 기준 |
| --- | --- |
| `EXPO_OWNER` | 앞에서 확정한 Expo 개인 username 또는 Organization slug |
| `EXPO_PROJECT_ID` | 해당 owner 아래 EAS가 생성한 실제 프로젝트 UUID |
| `EXPO_PUBLIC_APP_ENV` | 개발·검증·운영 환경을 구분하는 프로젝트 설정 값 |
| `EXPO_PUBLIC_API_URL` | 앱 실기기에서 도달 가능한 공개 HTTPS API 주소 |
| `TRANSLACAT_ANDROID_PACKAGE` | 확정한 Android application ID |
| `TRANSLACAT_IOS_BUNDLE_IDENTIFIER` | 확정한 Apple bundle identifier |

日本語：

| 設定 | 意味と入力基準 |
| --- | --- |
| `EXPO_OWNER` | 前述で確定した Expo 個人 username または Organization slug |
| `EXPO_PROJECT_ID` | その owner の下で EAS が作成した実際のプロジェクト UUID |
| `EXPO_PUBLIC_APP_ENV` | 開発・検証・本番環境を区別するプロジェクト設定値 |
| `EXPO_PUBLIC_API_URL` | アプリの実機から到達できる公開 HTTPS API アドレス |
| `TRANSLACAT_ANDROID_PACKAGE` | 確定した Android application ID |
| `TRANSLACAT_IOS_BUNDLE_IDENTIFIER` | 確定した Apple bundle identifier |

`EXPO_PUBLIC_` 값은 앱에 포함되어 사용자에게 공개될 수 있는 값으로 취급한다. API 비밀 키나 서비스 계정 키를 넣지 않는다. 실기기에서 `localhost`는 PC가 아니라 그 휴대폰 자신을 가리키므로 서버 주소로 사용하지 않는다. [Expo 환경 변수](https://docs.expo.dev/guides/environment-variables/)

`EXPO_PUBLIC_` の値はアプリに含まれ、ユーザーに公開され得る値として扱う。API 秘密キーやサービスアカウントキーを入れない。実機の `localhost` は PC ではなくその端末自身を指すため、サーバーアドレスに使用しない。[Expo 環境変数](https://docs.expo.dev/guides/environment-variables/)

CLI는 EAS CLI `25.1.1`을 프로젝트 개발 의존성으로 고정해 전역 설치 없이 사용하도록 준비한다. 설치 후 소유자가 로컬 터미널에서 `eas login`을 진행하고 `eas whoami`로 `kurumichan` 계정인지 확인한다. 이 문서에서 `eas`는 프로젝트의 로컬 CLI를 뜻한다. [EAS 로그인](https://docs.expo.dev/build/setup/)

CLI は EAS CLI `25.1.1` をプロジェクトの開発依存関係として固定し、グローバルインストールなしで使えるようにする。インストール後、所有者がローカル端末で `eas login` を実行し、`eas whoami` で `kurumichan` アカウントであることを確認する。本書の `eas` はプロジェクトのローカル CLI を指す。[EAS ログイン](https://docs.expo.dev/build/setup/)

이번 EAS 프로젝트는 이미 웹에서 생성했으므로 새로 만들지 않는다. 로컬 설정에는 owner `kurumichans-team`과 위 projectId를 연결하고, 로그인 후 `eas project:info`로 해당 프로젝트 접근과 설정을 확인한다. 기존 프로젝트 연결이 필요한 경우에만 `eas init --id 55b79b33-07b9-4d4b-a3f8-cf24ea76b139`를 사용한다. [프로젝트 생성과 연결](https://docs.expo.dev/eas/cli/#eas-init)

今回の EAS プロジェクトはすでに web で作成済みなので新規作成しない。ローカル設定で owner `kurumichans-team` と上記 projectId を接続し、ログイン後に `eas project:info` でアクセスと設定を確認する。既存プロジェクトの接続が必要な場合のみ `eas init --id 55b79b33-07b9-4d4b-a3f8-cf24ea76b139` を使う。[プロジェクト作成と接続](https://docs.expo.dev/eas/cli/#eas-init)

이 초기 빌드 스크립트는 공개 API 주소와 production 앱 식별자를 로컬에서 먼저 검사한다. 이 공개 값들은 로컬 환경과 해당 EAS environment에 동일하게 설정한다. Firebase 파일은 다음 절차의 EAS File 변수로 공급하며 로컬 `GOOGLE_SERVICES_JSON`은 비워 둔다.

この初期ビルドスクリプトは、公開 API アドレスと production アプリ識別子を先にローカルで検査する。これらの公開値はローカル環境と該当 EAS environment に同じ値を設定する。Firebase ファイルは次の手順の EAS File 変数から供給し、ローカルの `GOOGLE_SERVICES_JSON` は空のままにする。

완료 기준: `whoami`가 의도한 소유자이고 EAS dashboard의 프로젝트와 로컬 owner·projectId가 일치한다. production 식별자는 확정 전까지 비워 두고, 설정 검증이 누락을 오류로 처리하도록 유지한다.

完了基準：`whoami` が意図した所有者を示し、EAS dashboard のプロジェクトとローカルの owner・projectId が一致する。production 識別子は確定前まで空のままとし、設定検証が欠落をエラーとして扱う状態を保つ。

## 5 Firebase와 Android FCM V1 준비

> 日本語：5 Firebase と Android FCM V1 の準備

2026년 10월 18일 확인된 개발 설정은 다음과 같다. 전달 없는 권한 검사와 실제 기기 수신 결과를 구분해 기록한다.

2026 年 10 月 18 日に確認した開発設定は以下のとおり。配信しない権限検査と実際の端末受信結果は分けて記録する。

| 항목 | 확인 결과 |
| --- | --- |
| Firebase 프로젝트 | `translacat-mobile-dev`, Spark 무료 플랜 |
| Android 개발 package | `com.krumichan.translacat.dev` |
| Firebase Android appId | `1:267436430050:android:9b86b3569d6791a46ac4bb` |
| 앱 설정 파일 | 프로젝트와 package 일치 검사 통과, 중앙 검증 폴더에 보관 |
| EAS 파일 변수 | 프로젝트 범위 `development` 환경의 `GOOGLE_SERVICES_JSON`, File / secret 생성 후 존재 재조회 성공 |
| FCM V1 개인 키 | `expo-fcm-dev` 계정용 JSON 키 1개 발급·EAS Android FCM V1 연결과 메타데이터 재조회 완료 |
| 발송 권한 검사 | 개발 계정으로 FCM `validate_only: true` 요청 HTTP 200, 실제 메시지 전달 없음 |
| 실제 Android 수신 | Expo 공식 도구에서 보낸 `TranslaCat 푸시 테스트`가 백그라운드 휴대폰 알림창에 도착했다고 사용자 확인 |

日本語：

| 項目 | 確認結果 |
| --- | --- |
| Firebase プロジェクト | `translacat-mobile-dev`、Spark 無料プラン |
| Android 開発 package | `com.krumichan.translacat.dev` |
| Firebase Android appId | `1:267436430050:android:9b86b3569d6791a46ac4bb` |
| アプリ設定ファイル | プロジェクトと package の一致検査に成功し、中央検証フォルダーに保存 |
| EAS ファイル変数 | プロジェクト範囲の `development` 環境の `GOOGLE_SERVICES_JSON`、File / secret 作成後に存在の再照会に成功 |
| FCM V1 秘密鍵 | `expo-fcm-dev` アカウント用 JSON キー 1 件の発行・EAS Android FCM V1 接続とメタデータ再照会を完了 |
| 送信権限検査 | 開発アカウントで FCM `validate_only: true` を要求し HTTP 200、実際のメッセージ配信なし |
| 実際の Android 受信 | Expo 公式ツールから送った `TranslaCat 푸시 테스트` がバックグラウンドの端末通知欄に届いたとユーザーが確認 |

`preview` package는 `com.krumichan.translacat.preview`이므로 개발용 설정 파일을 재사용하지 않는다. 운영 package는 미확정이며 운영 Firebase 앱도 등록하지 않았다.

`preview` package は `com.krumichan.translacat.preview` なので、開発用設定ファイルは再利用しない。本番 package は未確定で、本番 Firebase アプリも登録していない。

1. 같은 소유자의 Google 계정으로 [Firebase Console](https://console.firebase.google.com/)에 들어가 개발용 프로젝트를 만든다. 관리할 프로젝트 이름과 ID를 확인한다. 이번 단계에는 푸시 전달에 필요한 Firebase 설정만 준비한다.
   <br>日本語：同じ所有者の Google アカウントで [Firebase Console](https://console.firebase.google.com/) に入り、開発用プロジェクトを作成する。管理するプロジェクト名と ID を確認する。今回の段階ではプッシュ配信に必要な Firebase 設定のみ準備する。
2. Android 앱 추가 시 개발 빌드가 실제 사용할 package를 정확히 입력한다. Firebase Android 앱에 등록한 package는 나중에 바꿀 수 없으므로 잠정 ID 변경 가능성이 있으면 앱 등록은 식별자 확정 뒤 진행한다. 운영 앱은 확정된 운영 package로 별도 등록한다. [Firebase Android 등록](https://firebase.google.com/docs/android/setup)
   <br>日本語：Android アプリ追加時に、開発ビルドが実際に使用する package を正確に入力する。Firebase Android アプリに登録した package は後で変更できないため、暫定 ID を変更する可能性があるなら識別子確定後に登録する。本番アプリは確定した本番 package で別途登録する。[Firebase Android 登録](https://firebase.google.com/docs/android/setup)
3. Android 앱 설정 파일 `google-services.json`을 내려받는다. Expo 프로젝트의 사용할 EAS environment에 **File 변수 `GOOGLE_SERVICES_JSON`**으로 이 파일을 등록한다. 원격 빌드에서 실제 경로로 주입하므로 로컬 환경의 같은 변수는 비워 둔다. 이 파일은 Firebase 앱 식별용 설정이며 서비스 계정 개인 키 JSON과 다르다. [EAS 파일 환경 변수](https://docs.expo.dev/eas/environment-variables/)
   <br>日本語：Android アプリ設定ファイル `google-services.json` をダウンロードする。使用する Expo プロジェクトの EAS environment に、**File 変数 `GOOGLE_SERVICES_JSON`** として登録する。リモートビルドで実際のパスとして注入されるので、ローカル環境の同名変数は空にする。これは Firebase アプリ識別用の設定であり、サービスアカウント秘密鍵 JSON とは異なる。[EAS ファイル環境変数](https://docs.expo.dev/eas/environment-variables/)
4. FCM V1 전달용 Google 서비스 계정 자격증명은 EAS Credentials의 Android push credentials에 연결한다. 기존 키와 권한을 먼저 확인하고, 필요한 경우 공식 절차로 키를 생성한다. 개인 키 JSON은 저장소·채팅에 넣지 않고 소유자가 EAS의 자격증명 화면 또는 로컬 CLI에서 업로드한다. [Expo FCM V1 자격증명](https://docs.expo.dev/push-notifications/fcm-credentials/)
   <br>日本語：FCM V1 配信用 Google サービスアカウント認証情報は、EAS Credentials の Android push credentials に接続する。既存キーと権限を先に確認し、必要なら公式手順でキーを作成する。秘密鍵 JSON はリポジトリやチャットに入れず、所有者が EAS の認証情報画面またはローカル CLI からアップロードする。[Expo FCM V1 認証情報](https://docs.expo.dev/push-notifications/fcm-credentials/)

개발용 발송 계정에는 해당 프로젝트의 `roles/firebasecloudmessaging.admin`(Firebase Cloud Messaging API Admin)을 사용한다. FCM 발송 권한 `cloudmessaging.messages.create`를 포함하는 역할이다. 이름이 비슷한 `roles/firebasenotifications.admin`과 혼동하지 않으며 Owner·Editor를 발송 계정에 부여하지 않는다. [FCM IAM 역할](https://docs.cloud.google.com/iam/docs/roles-permissions/firebasecloudmessaging)

開発用送信アカウントには、そのプロジェクトの `roles/firebasecloudmessaging.admin`（Firebase Cloud Messaging API Admin）を使う。FCM 送信権限 `cloudmessaging.messages.create` を含むロールである。似た名前の `roles/firebasenotifications.admin` と混同せず、送信アカウントに Owner・Editor を付与しない。[FCM IAM ロール](https://docs.cloud.google.com/iam/docs/roles-permissions/firebasecloudmessaging)

승인한 계정은 `expo-fcm-dev`이며 이 개발 프로젝트의 위 역할을 조건 없는 binding으로 추가했다. 기존 IAM binding을 보존했고 키는 한 개만 발급했다. 로컬 원본은 중앙 검증 폴더의 접근 제한 `private` 하위에 보관하며 현재 Windows 사용자·SYSTEM·관리자만 허용한다. 앱 소스·앱 환경 변수·GitHub에는 개인 키를 넣지 않는다. 기존에 꺼져 있던 IAM API만 활성화했으며 IAM API는 무료이고 결제 계정·유료 플랜은 연결하지 않았다. [IAM 요금](https://cloud.google.com/iam/pricing)

承認したアカウントは `expo-fcm-dev` で、この開発プロジェクトの上記ロールを条件なし binding として追加した。既存 IAM binding は維持し、キーは 1 件のみ発行した。ローカル原本は中央検証フォルダーのアクセス制限された `private` 配下に保存し、現在の Windows ユーザー・SYSTEM・管理者だけに許可する。アプリソース・アプリ環境変数・GitHub に秘密鍵を入れない。以前無効だった IAM API のみ有効にし、IAM API は無料で、請求先アカウントや有料プランは接続していない。[IAM 料金](https://cloud.google.com/iam/pricing)

FCM 권한 검사는 테스트 topic에 `validate_only: true`를 사용해 전달 없이 확인했다. 이후 개발 앱에서 알림 권한 허용과 Expo 푸시 토큰 표시를 사용자가 확인했고, 휴대폰 브라우저의 [Expo 공식 알림 도구](https://expo.dev/notifications)에 직접 토큰을 붙여 테스트를 전송했다. `chat-messages` 채널의 실제 원격 알림이 Android 알림창에 도착했다고 사용자 확인을 받았다. 토큰 값은 채팅·작업 기록에 수집하지 않았고 ticket·receipt도 별도 조회하지 않았다. 잠금·foreground·cold-start 상태별 수신, 알림 클릭 방 이동과 채팅 서버 자동 발송은 아직 검증하지 않았다.

FCM 権限検査はテスト topic に `validate_only: true` を使い、配信せずに確認した。その後、開発アプリで通知許可と Expo プッシュトークンの表示をユーザーが確認し、端末ブラウザーの [Expo 公式通知ツール](https://expo.dev/notifications) に直接トークンを貼り付けてテスト送信した。`chat-messages` チャンネルの実際のリモート通知が Android 通知欄に届いたとユーザーから確認を得た。トークン値はチャット・作業記録に収集せず、ticket・receipt も別途照会していない。ロック・foreground・cold-start 別の受信、通知タップによる部屋移動、チャットサーバーからの自動送信はまだ検証していない。

Firebase Authentication으로 사용자 DB를 새로 만들지 않는다. Google·Apple 앱 로그인은 기존 BE 계정을 유지하는 후속 인증 구현 단계에서 다룬다. 이번 Firebase 준비는 Android 알림 전달용이다.

Firebase Authentication でユーザー DB を新設しない。Google・Apple のアプリログインは既存 BE アカウントを維持する後続の認証実装段階で扱う。今回の Firebase 準備は Android 通知配信用である。

완료 기준: Android 앱 package, EAS File 변수의 `google-services.json`에 해당하는 Firebase 프로젝트, EAS에 연결한 FCM V1 자격증명의 프로젝트가 일치한다. 원격 Android 빌드는 주입된 파일의 존재를 검사한다. 실제 알림 수신 완료 여부는 실기기 검증으로 판정한다.

完了基準：Android アプリ package、EAS File 変数の `google-services.json` に対応する Firebase プロジェクト、EAS 接続済み FCM V1 認証情報のプロジェクトが一致する。リモート Android ビルドは注入されたファイルの存在を検査する。実際の通知受信完了は実機検証で判定する。

## 6 두 실기기 개발 빌드 준비

> 日本語：6 2 台の実機向け開発ビルドの準備

EAS 프로젝트와 해당 플랫폼의 서명·푸시 자격증명을 연결한 뒤 프로젝트의 `build:development:android`와 `build:development:ios` 스크립트로 개발 빌드를 만든다. 소스만 준비되었다고 기기 설치나 푸시 수신이 완료되는 것은 아니다. EAS CLI 인증·클라우드 빌드 실행·서명 연결은 소유자 인증이 가능한 세션에서 진행한다.

EAS プロジェクトと対象プラットフォームの署名・プッシュ認証情報を接続した後、プロジェクトの `build:development:android` と `build:development:ios` スクリプトで開発ビルドを作成する。ソースだけを準備しても端末インストールやプッシュ受信が完了したことにはならない。EAS CLI 認証・クラウドビルド実行・署名接続は、所有者の認証が可能なセッションで行う。

2026년 10월 18일 Expo `Free` 플랜과 Android 무료 한도 15회 중 사용 0회를 확인하고 첫 Android 개발 빌드를 요청했다. 개발 package의 원격 keystore를 생성했으며 [빌드 결과](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/a13c44e4-d91c-4da4-80bc-315ad7b4a271)에서 APK archive 업로드와 Install 버튼·설치 QR을 확인했다. 사용자가 Android에 설치하고 Metro에 연결해 채팅 기본 화면이 뜬 것을 확인했다. 유료 플랜 변경·스토어 제출은 수행하지 않았다.

2026 年 10 月 18 日に Expo `Free` プランと Android 無料枠 15 回中使用 0 回を確認し、最初の Android 開発ビルドを要求した。開発 package のリモート keystore を作成し、[ビルド結果](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/a13c44e4-d91c-4da4-80bc-315ad7b4a271) で APK archive のアップロードと Install ボタン・インストール QR を確認した。ユーザーが Android にインストールして Metro に接続し、基本チャット画面が表示されたことを確認した。有料プラン変更・ストア提出は実施していない。

첫 클라우드 Expo Doctor 결과는 **20개 중 19개 통과**다. 나머지는 개발 의존성에 고정 설치한 `eas-cli`를 프로젝트 밖에서 실행하도록 권하는 검사이며, 네이티브 컴파일은 계속 진행한다. 모든 검사 통과로 표기하지 않고 개발 도구 배치 정리를 후속 검증 항목으로 남긴다.

最初のクラウド Expo Doctor は **20 件中 19 件成功**。残りは開発依存関係に固定した `eas-cli` をプロジェクト外で実行するよう推奨する検査で、ネイティブコンパイルは継続する。全検査成功とは記載せず、開発ツール配置の整理を後続の検証項目として残す。

이 `development` APK는 Expo 개발 클라이언트다. 설치 후 앱 화면을 실행하려면 PC의 모바일 폴더에서 `npm.cmd start`로 Metro를 실행하고 개발 서버에 연결해야 한다. 독립 실행 배포용 앱과 구분한다. 현재 공개 API 주소와 서버 로그인·채팅 연동은 미완료이므로 이 빌드는 기본 화면과 알림 진단용이다.

この `development` APK は Expo 開発クライアントである。インストール後にアプリ画面を起動するには、PC のモバイルフォルダーで `npm.cmd start` により Metro を起動し、開発サーバーに接続する必要がある。単独起動の配布用アプリとは区別する。現在、公開 API アドレスとサーバーログイン・チャット連携は未完了なので、このビルドは基本画面と通知診断用である。

이번 작업에서는 PC의 LAN 모드 Metro 서버를 8081 포트에 실행해 두었고, Android manifest와 개발 JS bundle의 HTTP 200 응답을 확인했다. 이후 사용자가 실제 휴대폰에서 서버에 접속해 앱의 채팅 기본 화면이 뜬 것을 확인했다. `설정 → 개발 점검`에서 권한·Expo 푸시 토큰을 확인한 뒤, 기기 내 예약 알림과 구분되는 실제 원격 푸시 수신도 사용자 관찰로 확인했다.

今回の作業では PC の LAN モード Metro サーバーを 8081 ポートで起動し、Android manifest と開発 JS bundle の HTTP 200 応答を確認した。その後、ユーザーが実際の端末からサーバーに接続し、アプリの基本チャット画面が表示されたことを確認した。`설정 → 개발 점검` で権限・Expo プッシュトークンを確認後、端末内の予約通知とは区別される実際のリモートプッシュ受信もユーザー観察で確認した。

### Android 첫 원격 알림 검증 결과

> 日本語：Android の最初のリモート通知検証結果

- 개발 APK 설치·Metro 연결·앱 화면 실행·알림 권한·Expo 푸시 토큰 표시를 사용자가 확인했다.
  <br>日本語：開発 APK のインストール・Metro 接続・アプリ画面起動・通知許可・Expo プッシュトークン表示をユーザーが確認した。
- 사용자가 휴대폰 브라우저에서 Expo 공식 도구로 `TranslaCat 푸시 테스트`를 전송하고, 백그라운드 상태의 실제 Android 알림창에 도착했다고 확인했다. 이 결과는 실제 원격 전달 성공이며 앞선 `validate_only` 검사와 별개다.
  <br>日本語：ユーザーが端末ブラウザーの Expo 公式ツールから `TranslaCat 푸시 테스트` を送信し、バックグラウンド状態の実際の Android 通知欄に届いたと確認した。これは実際のリモート配信成功であり、先行の `validate_only` 検査とは別である。
- 기기 토큰은 수집하지 않았으며 전송 ticket와 공급자 receipt도 독립 조회하지 않았다. 확인 근거는 사용자가 보고한 기기 표시 결과다.
  <br>日本語：端末トークンは収集せず、送信 ticket と provider receipt も独立照会していない。確認根拠は、ユーザーが報告した端末表示結果である。
- 잠금·foreground·cold-start 별도 동작, 로그인과 소속 확인 후 방 이동, 서버의 채팅 자동 알림 발송과 iPhone 수신은 후속 검증이다. Phase 0 전체나 양 플랫폼 출시 완료로 처리하지 않는다.
  <br>日本語：ロック・foreground・cold-start の個別動作、ログインと所属確認後の部屋移動、サーバーのチャット自動通知送信と iPhone 受信は後続検証とする。Phase 0 全体や両プラットフォームのリリース完了として扱わない。

- **Android:** 소유자 휴대폰에서 개발 APK를 설치하고 실행한다. 앱의 알림 권한을 허용하며 개발 API에 연결되는지 확인한다. Play Console 등록과 별개로, Android 개발 빌드 자체에 Play 공개 출시 승인이 필요한 것은 아니다.
  <br>日本語：**Android：**所有者の端末に開発 APK をインストールして起動する。アプリの通知を許可し、開発 API に接続できるか確認する。Play Console 登録とは別に、Android 開発ビルド自体に Play の公開リリース承認は不要である。
- **iPhone:** 기기 사용자가 등록 링크를 자신의 iPhone에서 열어 테스트 기기 등록을 완료한다. 유료 개발자 팀의 해당 기기용 서명 설정으로 개발 빌드를 만들고 설치한다. Apple 개발자 비밀번호를 기기 사용자에게 공유하지 않는다. [iOS 기기 개발 빌드](https://docs.expo.dev/develop/development-builds/introduction/?buildenv=build-with-eas)
  <br>日本語：**iPhone：**端末利用者が自分の iPhone で登録リンクを開き、テスト端末の登録を完了する。有料開発者チームのその端末向け署名設定で開発ビルドを作成してインストールする。Apple 開発者パスワードを端末利用者と共有しない。[iOS 端末の開発ビルド](https://docs.expo.dev/develop/development-builds/introduction/?buildenv=build-with-eas)

푸시는 **Expo Go 대신 development build**에서 검증한다. SDK 58 개발은 처음부터 이 경로를 기준으로 한다. 이후 TestFlight와 Play 내부 테스트에서 배포용 빌드를 별도로 확인한다. [푸시 개발 빌드 안내](https://docs.expo.dev/push-notifications/faq/)

プッシュは **Expo Go ではなく development build** で検証する。SDK 58 開発は最初からこの経路を基準とする。その後 TestFlight と Play 内部テストで配布用ビルドを別途確認する。[プッシュ開発ビルド案内](https://docs.expo.dev/push-notifications/faq/)

계정과 빌드 연결 후의 첫 기기 검증은 Android와 iPhone 각각에서 앱 실행·권한 허용·토큰 등록·테스트 알림 수신이다. 채팅 서버 연동이 끝나면 **웹 또는 Android 전송 → 잠긴 iPhone 알림 → 알림을 눌러 해당 방 진입 → 답장 → 웹과 Android 반영**, 그리고 반대 방향을 확인한다.

アカウントとビルド接続後の最初の端末検証は、Android と iPhone それぞれでのアプリ起動・通知許可・トークン登録・テスト通知受信である。チャットサーバー連携後は、**web または Android から送信 → ロック中の iPhone に通知 → 通知をタップして該当部屋へ入室 → 返信 → web と Android へ反映**、および逆方向を確認する。

## Phase 0 인계 확인표

> 日本語：Phase 0 引き継ぎチェック表

- [x] Expo username `kurumichan`과 owner `kurumichans-team`을 확정했다.
  <br>日本語：Expo username `kurumichan` と owner `kurumichans-team` を確定した。
- [x] 웹에서 `translacat-mobile` 프로젝트를 만들고 실제 EAS 프로젝트 UUID를 받았다.
  <br>日本語：web で `translacat-mobile` プロジェクトを作成し、実際の EAS プロジェクト UUID を取得した。
- [x] 로컬 EAS CLI에 로그인하고 `whoami`와 프로젝트 접근을 확인했다.
  <br>日本語：ローカル EAS CLI にログインし、`whoami` とプロジェクトへのアクセスを確認した。
- [ ] Apple 가입 유형과 판매자 이름 공개 범위를 확인하고 등록 상태를 기록했다.
  <br>日本語：Apple 登録種別と販売者名の公開範囲を確認し、登録状況を記録した。
- [ ] Google Play 가입·본인·Android 기기 인증 상태를 기록했다.
  <br>日本語：Google Play の登録・本人確認・Android 端末認証状況を記録した。
- [ ] 정식 Android package와 Apple bundle identifier의 확정 여부를 기록했다.
  <br>日本語：正式 Android package と Apple bundle identifier が確定したかを記録した。
- [x] Firebase 개발 프로젝트·Android 개발 앱·EAS development 설정 파일 연결을 확인했다.
  <br>日本語：Firebase 開発プロジェクト・Android 開発アプリ・EAS development 設定ファイルの接続を確認した。
- [x] FCM V1 발송 자격증명을 연결하고 해당 프로젝트 일치와 전달 없는 권한 검증을 확인했다.
  <br>日本語：FCM V1 送信認証情報を接続し、対象プロジェクトの一致と配信を伴わない権限検証を確認した。
- [x] Android 개발 APK를 생성하고 설치 링크·QR을 확인했다.
  <br>日本語：Android 開発 APK を作成し、インストールリンク・QR を確認した。
- [x] Android 실기기 설치·Metro 연결·앱 기본 화면 실행을 사용자 확인했다.
  <br>日本語：Android 実機インストール・Metro 接続・アプリ基本画面起動をユーザーが確認した。
- [x] Android 알림 권한·Expo 토큰 발급과 백그라운드 실제 원격 푸시 수신을 사용자 확인했다.
  <br>日本語：Android の通知許可・Expo トークン発行と、バックグラウンドでの実際のリモートプッシュ受信をユーザーが確認した。
- [ ] iPhone 개발 빌드 설치와 실제 원격 푸시 수신을 확인했다.
  <br>日本語：iPhone 開発ビルドのインストールと実際のリモートプッシュ受信を確認した。
- [ ] 12명 비공개 테스터 준비와 추후 14일 연속 참여 일정을 잡았다.
  <br>日本語：12 人の非公開テスター準備と、その後の 14 日間連続参加の日程を設定した。

공유할 인계 내용은 `Expo username / owner / EAS projectId / Apple 가입 유형과 승인 여부 / Play 인증 여부 / Firebase projectId / 식별자 확정 여부 / 두 기기 설치 여부`다. 아직 가입·키 연결·실기기 설치를 진행하지 않았다면 해당 항목을 대기로 남긴다. 이 확인표를 채우기 전의 Phase 0 완료는 **계정 절차와 코드 기반 준비까지**를 뜻하며, 자격증명 연결·푸시 성공·스토어 출시 완료로 표시하지 않는다.

共有する引き継ぎ内容は `Expo username / owner / EAS projectId / Apple 가입 유형과 승인 여부 / Play 인증 여부 / Firebase projectId / 식별자 확정 여부 / 두 기기 설치 여부`（Apple の登録種別と承認状況、Play の認証状況、識別子の確定状況、2 台の端末へのインストール状況を含む）。まだ登録・キー接続・実機インストールを実施していない場合は、その項目を待機状態のままにする。このチェック表を埋める前の Phase 0 完了は **アカウント手順とコード基盤の準備まで**を意味し、認証情報接続・プッシュ成功・ストア公開完了とは記載しない。