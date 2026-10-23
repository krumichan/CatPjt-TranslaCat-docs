# 모바일 사용자 차단 관리

# モバイルのユーザーブロック管理

기존 알림·방별 무음·배지 기능은 사용자 실기기 보고로 확인했다. 이 결과를 보존하고 모바일 개발 계획의 사용자 보호 단계 중 기존 웹·BE 차단 관리 연결을 구현한다. 신고·계정 삭제와 OPEN의 익명 멤버 차단은 이번 기존 계정 차단과 별도다.

既存の通知・ルーム別ミュート・バッジ機能はユーザーの実機報告で確認した。この結果を保存し、モバイル開発計画のユーザー保護段階として既存ウェブ・BE のブロック管理を接続する。通報・アカウント削除と OPEN の匿名メンバーブロックは、今回の既存アカウントブロックとは別である。

## 구현

## 実装

설정과 친구 화면에서 사용자 차단 관리로 이동한다. TranslaCat ID 검색으로 닉네임·ID를 확인하고 명시적인 확인 화면 뒤 차단한다. 본인의 차단 목록과 해제도 제공한다. 자신의 계정은 선택하지 못하고, 해제는 현재 목록의 대상 userId를 사용한다. 차단 행 id를 대상 사용자로 보내지 않는다. OPEN 방별 별칭을 실제 계정 ID로 해석하거나 표시하지 않는다.

設定と友達画面からユーザーブロック管理へ移動する。TranslaCat ID 検索でニックネーム・ID を確認し、明示的な確認画面を経てブロックする。自分のブロック一覧と解除も提供する。自分自身のアカウントは選べず、解除には現在の一覧の対象 userId を使用する。ブロック行の id を対象ユーザーとして送らない。OPEN のルーム別別名を実際のアカウント ID と解釈・表示しない。

기존 BE 계약 GET/POST /api/v1/blocks, DELETE /api/v1/blocks/{blockedUserId}를 재사용한다. POST 본문은 blockedPublicId 하나이며 본인은 인증 principal에서 결정된다. 기존 친구 관계 해제·친구 연결 제한이 적용되고 해제해도 친구는 자동 복원되지 않는다. 기존 대화 삭제·공통 방 메시지 숨김·모든 푸시 억제의 보장으로 표시하지 않는다. 방별 무음과 계정 차단의 차이를 화면에서 설명한다.

既存 BE 契約 GET/POST /api/v1/blocks、DELETE /api/v1/blocks/{blockedUserId} を再利用する。POST 本文は blockedPublicId 1 個で、本人は認証 principal から決まる。既存の友達関係解除・友達接続制限が適用され、ブロックを解除しても友達は自動復元しない。既存会話の削除・共通ルームのメッセージ非表示・全プッシュ抑制を保証する表示はしない。ルーム別ミュートとアカウントブロックの違いを画面で説明する。

확인·연타·취소를 구분하고, 쓰기 응답을 잃어도 쓰기를 자동 재전송하지 않는다. 결과는 GET 차단 목록으로 확인한다. GET도 실패하면 추가 변경을 막고 새로고침을 요구한다. 통신 제한 시간은 15초, 계정·화면·활성 상태가 바뀌면 요청 취소와 owner 검사를 함께 적용해 늦은 응답의 UI·cache 효과를 막는다. 친구·친구 요청·방 목록 cache는 같은 세션에서만 무효화한다.

確認・連打・キャンセルを区別し、書き込み応答を失っても書き込みを自動再送しない。結果は GET のブロック一覧で確認する。GET も失敗したら追加変更を止め、再取得を求める。通信制限時間は 15 秒、アカウント・画面・有効状態が変わったら要求キャンセルと owner 検査を併用し、遅延応答の UI・cache 効果を防ぐ。友達・友達要求・ルーム一覧 cache は同じセッションでのみ無効化する。

## 검증과 진행 상태

## 検証と進行状態

신규 집중15개 포함 모바일 회귀467/467 PASS, 공식 Expo Router 생성 타입 갱신 후 TypeScript PASS, 앱 전체 lint PASS. 이후 이전 APK 완료 기록을 보존할 수 있도록 로컬 빌드 출력 폴더를 프로세스 환경으로 지정하는 한 줄 보완을 했고 변경 파일 집중 lint는 별도 실행 중이다. 합성 loopback 화면·native APK·기기 수락은 이 기록 시점 진행/대기이며 완료 영수증으로 추가한다. 기존 운영 사용자로 차단/해제를 시험하지 않았다.

新規集中 15 件を含むモバイル回帰 467/467 PASS、公式 Expo Router 生成型の更新後に TypeScript PASS、アプリ全体 lint PASS。その後、以前の APK 完了記録を保存できるようローカルビルド出力フォルダーをプロセス環境で指定する 1 行の補強を行い、変更ファイルの集中 lint は別途実行中である。合成 loopback 画面・native APK・端末受け入れは、この記録時点で進行中 / 待機で、完了記録に追記する。既存本番ユーザーでブロック / 解除は試験していない。

현재 공개 HTTPS GET /api/v1/blocks는404다. 기존 BE controller가 존재하는 것과 공개 경계 제공을 구분하여 배포 전담 「배포 책임과 규칙 기억하기」에 구체적인 최소 경계 연결을 인계했다. 기존 경로47개를 보존하면서 위2패턴3동사만 연결하도록 요청했으며 BE/CHAT/FE 이미지·DB·키·요금제 변경은 요청하지 않았다. 전담의 실제 완료 영수증 전에는 연결 완료로 표시하지 않는다.

現在の公開 HTTPS GET /api/v1/blocks は 404 である。既存 BE controller の存在と公開境界の提供を区別し、デプロイ専任の「배포 책임과 규칙 기억하기」へ具体的な最小境界接続を引き継いだ。既存 47 パスを維持して上記 2 パターン 3 動詞だけを接続するよう依頼し、BE / CHAT / FE イメージ・DB・鍵・料金プラン変更は要求していない。専任担当の実際の完了記録前に接続完了とは表示しない。

## 보존과 범위

## 保存と範囲

원본 설정·친구·빌드 스크립트·README와 검증·로그·합성 fixture는 현재 workspace의 .codex-workspace/verification/shared/mobile-user-blocks-20261022/에 보존한다. 기존 APK와 별도 완료 기록을 덮어쓰지 않는다. 이번 개발 채팅에서 Git 작업·서버 변경·실제 계정 차단·유료 등록을 하지 않는다.

原本の設定・友達・ビルドスクリプト・README と検証・ログ・合成 fixture は、現在の workspace の .codex-workspace/verification/shared/mobile-user-blocks-20261022/ に保存する。既存 APK と別の完了記録を上書きしない。今回の開発チャットで Git 作業・サーバー変更・実アカウントのブロック・有料登録を行わない。

## 코드·합성 화면·공개 경계 완료 후속

## コード・合成画面・公開境界完了の追記

전체 회귀467/467(신규15 포함)·TypeScript·전체 lint가 통과했고, 빌드 출력 분리 보완 뒤 최종 변경 파일 집중 lint와 Node 구문 검사도 통과했다. 기존 APK151핀과 비교하여 README·빌드 명령·설정·친구 화면 네 파일 변경과 차단 화면·route·service·controller·hook·test 여섯 파일 추가 외 기존 소스 drift는0이다. 집중15개를 전체467개에 다시 더하지 않는다.

全体回帰 467/467（新規 15 を含む）・TypeScript・全体 lint が合格し、ビルド出力分離の補強後、最終変更ファイルの集中 lint と Node 構文検査も合格した。既存 APK151 ピンと比べ、README・ビルドコマンド・設定・友達画面の 4 ファイル変更と、ブロック画面・route・service・controller・hook・test の 6 ファイル追加以外の既存ソース drift は 0 である。集中 15 件を全体 467 件に重ねて加算しない。

실제 export를 합성 loopback API에만 연결한 브라우저에서 로그인→설정→차단 관리→ID 확인→차단 취소(POST0)→확인 후 차단(POST1/목록1)→확인 후 해제(DELETE1/목록0)를 확인했다. 390×844와320×740에서 긴 닉네임이 줄바꿈되고 해제 버튼의 client/scroll 크기240×52가 일치했다. native 키보드나 Android 실기기 수락으로 확대하지 않는다. 첫 export의 Metro 캐시에 이전 fixture 주소가 남아 있어 원본을 보존하고 공식 --clear로 두 번째 export를 생성·검증했다. 근거는 ui-proof.json·ui-cancel-proof.json·ui-requests.jsonl이다.

実 export を合成 loopback API だけに接続したブラウザーで、ログイン→設定→ブロック管理→ID 確認→ブロックキャンセル（POST0）→確認後ブロック（POST1 / 一覧1）→確認後解除（DELETE1 / 一覧0）を確認した。390×844 と 320×740 で長いニックネームが折り返され、解除ボタンの client / scroll サイズ 240×52 が一致した。native キーボードや Android 実機受け入れに拡大しない。最初の export の Metro キャッシュに以前の fixture アドレスが残っていたため原本を保存し、公式 --clear で 2 回目の export を生成・検証した。根拠は ui-proof.json・ui-cancel-proof.json・ui-requests.jsonl である。

배포 전담은 2026-10-22 22:48 KST 기존47경로 전체 바이트를 보존해 위2패턴3동사만 추가했다. 두 설정 SHA-256은90e5ff82cc9e1479febbdab14bc6a8d7501c7f9233bedc2d198c378d9fc5fb72이며 기존 운영 BE72d4f5a의 exact controller/security/image를 확인했다. offline27·local전후7+7·외부GET/HEAD20·익명잘못된POST/DELETE2가 통과했고 실제 사용자 차단/해제는 실행하지 않았다. BE/CHAT/FE 이미지·Git·DB·키 변경은0이다. 완료 영수증 deployment/deployment-completed.json SHA-256 c4e79768e25caa5c5849be6d92a1065fb747e00afc3a805149f2f3f2df4f1c47를 개발 채팅에서도 대조했다.

デプロイ専任担当は 2026-10-22 22:48 KST に既存 47 パスの全バイトを維持し、上記 2 パターン 3 動詞だけを追加した。2 設定の SHA-256 は 90e5ff82cc9e1479febbdab14bc6a8d7501c7f9233bedc2d198c378d9fc5fb72 で、既存本番 BE72d4f5a の exact controller / security / image を確認した。offline27・local 前後 7+7・外部 GET / HEAD20・匿名不正 POST / DELETE2 が合格し、実ユーザーのブロック / 解除は実行していない。BE / CHAT / FE イメージ・Git・DB・鍵の変更は 0 である。完了記録 deployment/deployment-completed.json SHA-256 c4e79768e25caa5c5849be6d92a1065fb747e00afc3a805149f2f3f2df4f1c47 を開発チャットでも照合した。

같은 기존 서명의 ARM64 로컬 APK는 새 중앙 작업 폴더로 출력 분리하여 빌드 중이다. 기존 APK·완료 영수증을 덮어쓰지 않는다. 새 APK 완성·공유·기기 수락은 후속 영수증으로 구분한다.

同じ既存署名の ARM64 ローカル APK は新しい中央作業フォルダーへ出力を分けてビルド中である。既存 APK・完了記録は上書きしない。新 APK 完成・共有・端末受け入れは後続記録で区別する。

## 새 로컬 APK 빌드·검증 완료 후속

## 新ローカル APK ビルド・検証完了の追記

2026-10-22 23:07 KST 새 ARM64 release APK의 Gradle 단계가11분7초에 완료됐다(786작업 중25실행·761재사용). source157핀의 빌드 전후 해시가 일치하며 이전 APK151핀 중 이번 변경 외 drift0을 유지한다. 파일은 artifacts/translacat-dev-local-2026-10-22T13-52-41-308Z-arm64.apk, 31,379,900bytes, SHA-256 3174403e888f4d1cd0784190ce2a397761ddcb6860aeb8367df25ecdaa43ab1b다.

2026-10-22 23:07 KST に、新 ARM64 release APK の Gradle 段階が 11 分 7 秒で完了した（786 タスク中 25 実行・761 再利用）。source157 ピンのビルド前後ハッシュは一致し、以前の APK151 ピンのうち今回変更以外の drift0 を維持する。ファイルは artifacts/translacat-dev-local-2026-10-22T13-52-41-308Z-arm64.apk、31,379,900 bytes、SHA-256 3174403e888f4d1cd0784190ce2a397761ddcb6860aeb8367df25ecdaa43ab1b である。

실제 apksigner 서명·기존 인증서600dc03d957e64a9ff9c85160f81fde10d7a9721a527a81277a86982455647d9 일치, package/scheme/versionCode1/ARM64/release manifest, 공개 HTTPS 설정7개, ZIP CRC·경로·키 파일명 차단·16KiB 정렬, compiled TextView useBoundsForWidth=false, 차단·기존 알림/미읽음/badge bundle 표식을 확인했다. 새 bundle SHA-256은7262284c67b5f5bbb20ca8ff512f6c019dac82cfccaa7c507aecb154a946c881다. 근거는 apk-built.json·apk-verification.json·apk-verifier.log·apk-zipalign.log·local-apk-text-resources.json 및 native-build.log다. 이전 APK와 완료 영수증은 그대로 보존했다.

実際の apksigner 署名・既存証明書 600dc03d957e64a9ff9c85160f81fde10d7a9721a527a81277a86982455647d9 との一致、package / scheme / versionCode1 / ARM64 / release manifest、公開 HTTPS 設定 7 項目、ZIP CRC・パス・鍵ファイル名の阻止・16KiB 整列、compiled TextView useBoundsForWidth=false、ブロック・既存通知 / 未読 / badge の bundle 標識を確認した。新 bundle SHA-256 は 7262284c67b5f5bbb20ca8ff512f6c019dac82cfccaa7c507aecb154a946c881 である。根拠は apk-built.json・apk-verification.json・apk-verifier.log・apk-zipalign.log・local-apk-text-resources.json と native-build.log である。以前の APK と完了記録はそのまま保存した。

이번 새 해시의 Expo 업로드는 자동 검토가 별도 구체적 승인을 요구해 실행 전 거절했으며, 사용자가 정확한 파일 크기/해시와 기존 프로젝트를 지정한 질문에 “승인 — 새 APK를 같은 Expo 프로젝트에 업로드”라고 직접 답해 승인 부족 사유를 해결했다. upload-authorization.json에 이력·정확한 승인을 기록하고 동일한 공식 eas upload로 한 파일 공유를 진행한다. 클라우드 빌드·개인 키·소스 폴더 업로드·요금제 변경은 요청하지 않는다. 실제 링크/다운로드 해시는 후속 영수증으로 구분한다. 새 APK에서 인증된 차단·해제를 사용자 폰으로 확인하는 단계는 아직 대기다.

今回の新ハッシュの Expo アップロードは、自動レビューが別の具体的承認を要求して実行前に拒否した。ユーザーは正確なファイルサイズ / ハッシュと既存プロジェクトを指定した質問に「承認 — 新 APK を同じ Expo プロジェクトへアップロード」と直接回答し、承認不足を解消した。upload-authorization.json に履歴・正確な承認を記録し、同じ公式 eas upload で 1 ファイルの共有を進める。クラウドビルド・秘密鍵・ソースフォルダーのアップロード・料金プラン変更は要求しない。実リンク / ダウンロードハッシュは後続記録で区別する。新 APK で認証済みブロック・解除をユーザー端末で確認する段階はまだ待機である。

## 설치 링크·원격 파일 검증 완료

## インストールリンク・リモートファイル検証完了

[새 차단 관리 APK 설치 페이지](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/a7b81344-5e21-4d21-a9f4-0b60816f1b73)

[新ブロック管理 APK インストールページ](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/a7b81344-5e21-4d21-a9f4-0b60816f1b73)

공식 Expo 조회로 프로젝트55b79b33-07b9-4d4b-a3f8-cf24ea76b139·owner kurumichans-team·slug translacat-mobile·Android·INTERNAL·FINISHED를 확인했다. 공식 HTTPS 파일을 논리적으로 한 번 내려받아 31,379,900bytes 및 SHA-256 3174403e888f4d1cd0784190ce2a397761ddcb6860aeb8367df25ecdaa43ab1b가 로컬에서 실제 검증한 APK와 정확히 일치함을 확인했다. 공식 APK 업로드 실행은1회이며 실행 전 자동 검토 거절은 전송 횟수에 더하지 않는다. 클라우드 빌드·요금제 변경·개인 키·소스 폴더 업로드는0이다.

公式 Expo 照会でプロジェクト 55b79b33-07b9-4d4b-a3f8-cf24ea76b139・owner kurumichans-team・slug translacat-mobile・Android・INTERNAL・FINISHED を確認した。公式 HTTPS ファイルを論理的に 1 回ダウンロードし、31,379,900 bytes と SHA-256 3174403e888f4d1cd0784190ce2a397761ddcb6860aeb8367df25ecdaa43ab1b がローカルで実検証した APK と正確に一致することを確認した。公式 APK アップロード実行は 1 回で、実行前の自動レビュー拒否は送信回数に加算しない。クラウドビルド・料金プラン変更・秘密鍵・ソースフォルダーのアップロードは 0 である。

이전 앱을 삭제하지 않고 업데이트 설치한 뒤 설정/친구→사용자 차단 관리에서 테스트 계정의 ID 확인·취소·차단·목록 반영·해제를 확인한다. 차단하면 기존 친구 관계가 해제되고 해제 시 자동 복원되지 않는다. 기존 메시지는 남으며 방 알림은 방별 무음에서 관리한다. 새 APK의 실제 인증된 차단·해제는 아직 사용자 실기기 수락 대기다.

以前のアプリを削除せず更新インストールし、設定 / 友達→ユーザーブロック管理でテストアカウントの ID 確認・キャンセル・ブロック・一覧反映・解除を確認する。ブロックすると既存の友達関係は解除され、ブロック解除時も自動復元しない。既存メッセージは残り、ルーム通知はルーム別ミュートで管理する。新 APK の実際の認証済みブロック・解除は、まだユーザー実機受け入れ待ちである。

최종 근거는 delivery-completed.json·apk-share.json·expo-upload-readback.json·upload-authorization.json이다. 원본·소스 사본·이전 APK·첫 캐시 export·실패/중간 및 최종 로그를 보존했다. 검증용 loopback 프로세스와 임시 브라우저 탭만 정리하고 viewport를 복원했다. Git 변경과 서버 앱 이미지/DB 변경은 하지 않았다.

最終根拠は delivery-completed.json・apk-share.json・expo-upload-readback.json・upload-authorization.json である。原本・ソースコピー・以前の APK・最初のキャッシュ export・失敗 / 途中および最終ログを保存した。検証用 loopback プロセスと一時ブラウザータブだけを整理して viewport を復元した。Git 変更とサーバーアプリのイメージ / DB 変更は行っていない。

## 사용자 실기기 수락 완료

## ユーザー実機受け入れ完了

사용자가 새 APK로 업데이트한 뒤 차단과 해제를 확인했고 문제가 없다고 직접 회신했다. 이 범위는 사용자 보고 PASS로 기록하며 에이전트의 직접 기기 조작이나 설치 바이너리의 독립 식별로 확대하지 않는다. 기존 코드·화면·공개 경계·APK·다운로드 해시 검증 근거를 보존한다. device-user-acceptance.json과 갱신된 delivery-completed.json에 반영했다.

ユーザーは新 APK に更新してからブロックと解除を確認し、問題ないと直接回答した。この範囲をユーザー報告 PASS として記録し、エージェントによる直接端末操作やインストール済みバイナリの独立識別へ拡大しない。既存コード・画面・公開境界・APK・ダウンロードハッシュ検証の根拠を保存する。device-user-acceptance.json と更新した delivery-completed.json に反映した。
