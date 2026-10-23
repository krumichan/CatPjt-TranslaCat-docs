# 모바일 운영 문의 연결

# モバイルの運営問い合わせ接続

## 범위

## 範囲

직전 차단 관리 APK에 대해 사용자가 업데이트·차단·해제를 직접 확인하고 문제가 없다고 회신했다. 그 수락은 mobile-user-blocks-20261022/device-user-acceptance.json에 사용자 보고 PASS로 기록했다. 이번 후속 범위는 설정→문의함에서 문의 작성·내 문의 목록·상세·운영 답변·추가 답글을 제공하는 것이다. 문의 상태 변경, 채팅 메시지 신고, 계정 삭제, 약관·개인정보 정책 작성은 포함하지 않는다.

直前のブロック管理 APK について、ユーザーが更新・ブロック・解除を直接確認し問題ないと回答した。その受け入れは mobile-user-blocks-20261022/device-user-acceptance.json にユーザー報告 PASS として記録した。今回の後続範囲は、設定→問い合わせボックスでの問い合わせ作成・自分の問い合わせ一覧・詳細・運営回答・追加返信の提供である。問い合わせ状態の変更、チャットメッセージ通報、アカウント削除、利用規約・プライバシーポリシー作成は含めない。

기존 BE SupportInquiryController 계약을 사용한다. GET·POST /api/v1/support/inquiries, GET /api/v1/support/inquiries/{id}, POST /api/v1/support/inquiries/{id}/replies를 공개 인증 경계로 호출한다. 작성자는 인증 principal로 결정한다. 일반 계정은 본인 문의만 접근한다. 서버의 ADMIN 목록은 전체 문의를 반환하므로 앱은 소유자가 다른 응답을 표시하지 않고 개인 문의 전용·운영자 웹 문의함 안내를 한다. ADMIN이 본인 문의를 작성하거나 답변하는 정상 응답은 허용한다.

既存 BE SupportInquiryController 契約を使う。GET・POST /api/v1/support/inquiries、GET /api/v1/support/inquiries/{id}、POST /api/v1/support/inquiries/{id}/replies を公開認証境界経由で呼び出す。投稿者は認証 principal で決定する。一般アカウントは自分の問い合わせだけにアクセスする。サーバーの ADMIN 一覧は全問い合わせを返すため、アプリは所有者の異なる応答を表示せず、個人問い合わせ専用・運営者ウェブ問い合わせボックスへの案内を表示する。ADMIN が自分の問い合わせを作成・回答する正常な応答は許可する。

제목 최대200 UTF-16자·본문 및 답글 최대10,000자로 검증하고 종류7개, 상태3개, 사용자·운영자 답변 역할, 안전한 양수 ID, 본인 소유, 페이지 메타데이터, 답변 ID 순서·개수·같은 문의 선행 부모를 검증한다. 이메일 등 불필요한 계정 정보는 표시 모델에 남기지 않는다. 목록은20개씩 이전/다음으로 확인하며 답변 본문은 상세에서 조회한다. 서버의 LocalDateTime에 시간대를 임의로 붙이지 않고 원문 날짜·시각을 표시한다.

タイトルは最大 200 UTF-16 文字、本文・返信は最大 10,000 文字で検証し、種類 7 個、状態 3 個、ユーザー・運営者の回答役割、安全な正の ID、本人所有、ページメタデータ、回答 ID の順序・件数・同一問い合わせ内の先行親を検証する。メールなど不要なアカウント情報を表示モデルに残さない。一覧は 20 件ずつ前 / 次で確認し、回答本文は詳細で取得する。サーバーの LocalDateTime にタイムゾーンを勝手に付けず、原文の日時を表示する。

## 전송과 경계

## 送信と境界

문의·답글은 작성 내용 확인→최종 전송으로 구분하고 취소는 서버 쓰기를 하지 않는다. 문의 작성은 title/content/category만, 답글은 content만 보낸다. relatedMessageId는 support_message를 참조하므로 채팅 ID를 넣지 않는다. parentId·작성자·역할·상태도 전송하지 않는다. 상세 GET은 support 알림 읽음 처리가 있으므로 자동 재시도하지 않는다. 종료된 문의의 답글 입력·전송은 막고 서버409도 처리한다.

問い合わせ・返信は記入内容確認→最終送信に分け、キャンセルではサーバー書き込みを行わない。問い合わせ作成は title / content / category だけ、返信は content だけを送る。relatedMessageId は support_message を参照するためチャット ID を入れない。parentId・投稿者・役割・状態も送信しない。詳細 GET は support 通知の既読処理があるため自動再試行しない。終了した問い合わせは返信入力・送信を禁止し、サーバー 409 も処理する。

요청 제한 시간15초와 AbortSignal, 계정·focus·앱 활성 상태 소유권 검사를 적용한다. POST에는 멱등성 계약이 없으므로 결과 불명 요청을 자동 재전송하지 않는다. 목록/상세를 다시 확인해도 같은 본문만으로 전송 성공을 추정하지 않는다. 사용자가 조회 결과를 검토하고 중복 전송 주의를 명시 확인해야 쓰기를 재개한다. 답글 성공은 기존 상세에 없던 새 ID·증가한 개수·최상위 본문 일치까지 확인한다. 같은 세션의 화면 이탈/백그라운드/재진입 때 결과 불명 표시만 JS 메모리로 유지하며 본문·제목·토큰을 저장하지 않는다. 앱 프로세스 종료 이후 영속 전송 기록은 제공하지 않는다.

要求の制限時間 15 秒と AbortSignal、アカウント・focus・アプリ有効状態の所有権検査を適用する。POST には冪等性契約がないため結果不明要求を自動再送しない。一覧 / 詳細を再確認しても、同じ本文だけで送信成功を推定しない。ユーザーが照会結果を検討し、重複送信への注意を明示確認して初めて書き込みを再開する。返信成功は既存詳細になかった新 ID・件数増加・最上位本文一致まで確認する。同じセッションの画面離脱 / バックグラウンド / 再入場時には結果不明表示だけを JS メモリーに維持し、本文・タイトル・トークンは保存しない。アプリプロセス終了後の永続送信記録は提供しない。

작성 내용은 확인된 성공 때만 초기화한다. 최종 검토에서 성공 뒤 다시 작성하던 초안이 백그라운드 전환 시 성공 번호 초기화로 지워지는 문제를 찾아 최초 prebuild를 Gradle 전에 중단했다. 부모 실행기의 finally에서 package.json 원문 복원을 대조하고 첫 빌드 로그·소스 사본을 보존했다. 같은 화면 세션에서 성공 번호를 단조 유지하는 보완·집중 검증 후 새 빌드를 진행한다.

記入内容は確認済み成功時だけ初期化する。最終レビューで、成功後に再記入していた下書きがバックグラウンド移行時に成功番号の初期化で消える問題を発見し、最初の prebuild を Gradle 前に中断した。親実行器の finally で package.json 原文復元を照合し、最初のビルドログ・ソースコピーを保存した。同じ画面セッションで成功番号を単調に維持する補強・集中検証後、新ビルドを進める。

## 공개 경계 완료

## 公開境界完了

전담 「배포 책임과 규칙 기억하기」가 2026-10-22 23:43 KST 현재49개 경로를 보존하며 정확3패턴4메서드만 추가했다. active/renewal-template SHA-256은 bec865814e79ae2900654d9ed1c3d2310c7345311ae2464566bea3ff9b563838이며 현재52개 경로다. status/notifications 등 다른 support 경로는 추가하지 않았다. 기존 BE72d4f5a905f8d2b378ce452e6ed6694850541df2의 controller/security/실행 image를 대조했다. 새 서비스 이미지 배포·Git·DB·키·요금 변경은 없다.

専任の「배포 책임과 규칙 기억하기」が 2026-10-22 23:43 KST に現在の 49 パスを維持し、正確に 3 パターン 4 メソッドだけを追加した。active / renewal-template SHA-256 は bec865814e79ae2900654d9ed1c3d2310c7345311ae2464566bea3ff9b563838、現在は 52 パスである。status / notifications など他の support パスは追加していない。既存 BE72d4f5a905f8d2b378ce452e6ed6694850541df2 の controller / security / 実行 image を照合した。新サービスイメージのデプロイ・Git・DB・鍵・料金変更はない。

오프라인27·로컬전후7+7·외부GET17+HEAD3·익명잘못된POST2가 통과했다. 실제 인증된 문의 생성/답변/상세 읽음 동작은 검증하지 않았다. 완료 영수증 deployment/deployment-completed.json SHA-256 a0a9007d896770d411e008e9d9ff98160e78f3ba9fcc893a8476a480f891147a를 개발 채팅에서 대조했다. 복구는 적용 직전49개 경로를 담은 이번 새 백업으로만 한다.

オフライン 27・ローカル前後 7+7・外部 GET17+HEAD3・匿名の不正 POST2 が合格した。実際の認証済み問い合わせ作成 / 回答 / 詳細既読動作は検証していない。完了記録 deployment/deployment-completed.json SHA-256 a0a9007d896770d411e008e9d9ff98160e78f3ba9fcc893a8476a480f891147a を開発チャットで照合した。復旧は適用直前の 49 パスを含む今回の新バックアップだけを用いる。

## 검증과 보존

## 検証と保存

중앙 근거는 .codex-workspace/verification/shared/mobile-support-20261022/다. 새 서비스와 controller 테스트, 기존 전체 회귀, lint·TypeScript·Expo 설정, 외부 연결을 차단한 합성 loopback 화면, 실제 로컬 APK 서명·설정·compiled text resource·기능 표식과 원격 공유 파일 해시를 각각 기록한다. 검사 성공과 실제 사용자 기기 수락을 구분하고 최종 완료 상태는 후속 영수증으로 갱신한다.

中央根拠は .codex-workspace/verification/shared/mobile-support-20261022/ である。新サービスと controller テスト、既存全体回帰、lint・TypeScript・Expo 設定、外部接続を遮断した合成 loopback 画面、実ローカル APK 署名・設定・compiled text resource・機能標識とリモート共有ファイルハッシュをそれぞれ記録する。検査成功と実ユーザー端末の受け入れを区別し、最終完了状態は後続記録で更新する。

기존 APK·완료 영수증·원본 settings/README·첫 빌드 중단 로그·수정 전 소스 사본·기능 테스트·화면 fixture/export와 배포 증거를 보존한다. 모바일 저장소의 Git 초기화·staging·commit·push는 하지 않는다. 유료 스토어/개발자 등록은 사용자의 기존 선택에 따라 미룬다.

既存 APK・完了記録・原本 settings / README・最初のビルド中断ログ・修正前ソースコピー・機能テスト・画面 fixture / export とデプロイ証拠を保存する。モバイルリポジトリの Git 初期化・staging・commit・push は行わない。有料ストア / 開発者登録はユーザーの既存選択に従い後回しにする。

## 코드·합성 화면 검증 완료 후속

## コード・合成画面検証完了の追記

초안 보존 보완 후 모바일 전체511/511 PASS(신규서비스19·controller25 포함, 중복가산하지 않음), 최종 TypeScript PASS와 전체 lint 및 최종 변경/초안 보존 집중 lint PASS를 확인했다. 실제 개발용 HTTPS 환경의 Expo 설정 검사도 공개 API 설정 READY이며, 이 검사만으로 원격 소유권·실기기 푸시를 검증했다고 표시하지 않는다.

下書き保存補強後、モバイル全体 511/511 PASS（新サービス 19・controller25 を含み、重複加算しない）、最終 TypeScript PASS と全体 lint および最終変更 / 下書き保存の集中 lint PASS を確認した。実際の開発用 HTTPS 環境の Expo 設定検査も公開 API 設定 READY であり、この検査だけでリモート所有権・実機プッシュを検証したとは表示しない。

최종 웹 export를 127.0.0.1:8897 합성 계정 fixture에만 연결하고 CSP로 외부 API를 차단했다. 설정→문의함→작성→취소(POST0/초안유지)→확인(POST1/상세)→답글취소(POST0)→같은 상세 새로고침(초안유지)→확인(POST1/답글1/초안비움)→종료 문의 입력0→운영 답변 표시까지 확인했다. 390×844와320×740에서 긴 제목의 client/scroll 폭·높이가 같고 버튼도 잘리지 않았다. ui-proof.json·ui-cancel-proof.json·ui-reply-cancel-proof.json·ui-requests.jsonl 및 ui-390.png·ui-320.png를 보존했다.

最終ウェブ export を 127.0.0.1:8897 の合成アカウント fixture だけに接続し、CSP で外部 API を遮断した。設定→問い合わせボックス→作成→キャンセル（POST0 / 下書き維持）→確認（POST1 / 詳細）→返信キャンセル（POST0）→同じ詳細の再取得（下書き維持）→確認（POST1 / 返信1 / 下書き消去）→終了した問い合わせの入力0→運営回答表示まで確認した。390×844 と 320×740 で長いタイトルの client / scroll 幅・高さが一致し、ボタンも切れなかった。ui-proof.json・ui-cancel-proof.json・ui-reply-cancel-proof.json・ui-requests.jsonl および ui-390.png・ui-320.png を保存した。

이 브라우저 검증은 Android 키보드·스크린리더 실기기 수락으로 확대하지 않는다. 웹 미리보기의 radio는 시각 선택과 실제 category 전송이 반영되지만 aria-checked는 null이었다. native accessibilityState 사용 여부와 실제 Android 스크린리더는 별도 확인 범위다. PNG는 브라우저 실행기의 외장 경로 쓰기 제한 때문에 허용된 임시 폴더에 생성한 뒤 함수 실행기로 중앙 경로로 이동하고 SHA를 대조했다. 합성 서버·테스트 탭만 종료하고 viewport를 복원했으며 원본·중간 export·중단 빌드 사본과 로그를 삭제하지 않았다.

このブラウザー検証を Android キーボード・スクリーンリーダーの実機受け入れへ拡大しない。ウェブプレビューの radio は見た目の選択と実 category 送信が反映されるが、aria-checked は null だった。native accessibilityState の利用有無と実 Android スクリーンリーダーは別の確認範囲である。PNG はブラウザー実行器の外付けパスへの書き込み制限により、許可された一時フォルダーで生成後、関数実行器で中央パスへ移して SHA を照合した。合成サーバー・テストタブだけを終了し viewport を復元した。原本・途中 export・中断ビルドコピーとログは削除していない。

## 새 로컬 APK 완료

## 新ローカル APK 完成

2026-10-23 00:25 KST 같은 기존 개발 인증서로 ARM64 release APK를 완성했다. Gradle 단계23분57초·786작업 중25실행/761재사용이며 prebuild 시간은 별도다. 파일은 artifacts/translacat-dev-local-2026-10-22T14-52-55-709Z-arm64.apk, 31,415,160bytes, SHA-256 72fa31d8597e1ac1ff93a11398d12bdef9d2e911a3cdbe4c97a8bdb3e2dcbd48다. 소스164핀의 빌드 전후 해시를 보존했다.

2026-10-23 00:25 KST に同じ既存開発証明書で ARM64 release APK を完成した。Gradle 段階は 23 分 57 秒、786 タスク中 25 実行 / 761 再利用で、prebuild 時間は別である。ファイルは artifacts/translacat-dev-local-2026-10-22T14-52-55-709Z-arm64.apk、31,415,160 bytes、SHA-256 72fa31d8597e1ac1ff93a11398d12bdef9d2e911a3cdbe4c97a8bdb3e2dcbd48 である。ソース 164 ピンのビルド前後ハッシュを保存した。

실제 APK 서명과 기존 인증서600dc03d957e64a9ff9c85160f81fde10d7a9721a527a81277a86982455647d9 일치, package/scheme/versionCode1/ARM64/release, 공개 HTTPS·프로젝트 설정7개, 전체ZIP CRC·키 파일명/경로/중복 차단·16KiB 정렬, compiled TextView 자원과 문의·답글·기존 알림·미읽음·차단 bundle 표식이 모두 통과했다. bundle SHA-256 f8f1ae3fe82f6a80d386996f52f62635adac24f3055f698d9730f2bb0fc904e9. 근거는 apk-built.json·apk-verification.json·apk-verifier.log다.

実 APK 署名と既存証明書 600dc03d957e64a9ff9c85160f81fde10d7a9721a527a81277a86982455647d9 の一致、package / scheme / versionCode1 / ARM64 / release、公開 HTTPS・プロジェクト設定 7 項目、全 ZIP CRC・鍵ファイル名 / パス / 重複の阻止・16KiB 整列、compiled TextView 資源と問い合わせ・返信・既存通知・未読・ブロックの bundle 標識がすべて合格した。bundle SHA-256 は f8f1ae3fe82f6a80d386996f52f62635adac24f3055f698d9730f2bb0fc904e9。根拠は apk-built.json・apk-verification.json・apk-verifier.log である。

사용자는 정확한 새 파일 크기·해시와 기존 Expo 프로젝트를 지정한 질문에 '승인 — 문의함 APK를 같은 Expo 프로젝트에 업로드'라고 직접 답했다. upload-authorization.json에 이 승인만 기록하고, 해당 파일·기존 목적지·클라우드 빌드 및 요금 변경 제외 조건을 확인하는 공유 실행기로 공식 eas upload를 진행한다. 실제 설치 링크·다운로드 파일 해시는 별도 공유 영수증이 나온 뒤 확정한다. 새 APK의 실제 문의 작성·운영 답변 수락은 사용자 실기기 확인 대기다.

ユーザーは正確な新ファイルサイズ・ハッシュと既存 Expo プロジェクトを指定した質問に「承認 — 問い合わせボックス APK を同じ Expo プロジェクトにアップロード」と直接回答した。upload-authorization.json にこの承認だけを記録し、そのファイル・既存の宛先・クラウドビルドと料金変更を除外する条件を確認する共有実行器で公式 eas upload を進める。実インストールリンク・ダウンロードファイルハッシュは別の共有完了記録が出てから確定する。新 APK の実際の問い合わせ作成・運営回答受け入れはユーザー実機確認待ちである。

## 설치 링크·원격 APK 대조 완료

## インストールリンク・リモート APK 照合完了

[문의함 APK 설치 페이지](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/d3a10ff9-4c2b-4261-85f9-cea6a41684db). 공식 Expo 조회로 project55b79b33-07b9-4d4b-a3f8-cf24ea76b139·owner kurumichans-team·slug translacat-mobile·Android·INTERNAL·FINISHED를 확인했다. 공식 HTTPS 설치 파일을 내려받아 31,415,160bytes와 SHA-256 72fa31d8597e1ac1ff93a11398d12bdef9d2e911a3cdbe4c97a8bdb3e2dcbd48가 검증한 로컬 원본과 정확히 일치함을 확인했다. 공식 업로드 실행은1회, 클라우드 빌드·개인 키/소스 폴더 업로드·요금 변경은0이다.

[問い合わせボックス APK インストールページ](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/d3a10ff9-4c2b-4261-85f9-cea6a41684db)。公式 Expo 照会で project55b79b33-07b9-4d4b-a3f8-cf24ea76b139・owner kurumichans-team・slug translacat-mobile・Android・INTERNAL・FINISHED を確認した。公式 HTTPS インストールファイルをダウンロードし、31,415,160 bytes と SHA-256 72fa31d8597e1ac1ff93a11398d12bdef9d2e911a3cdbe4c97a8bdb3e2dcbd48 が検証済みローカル原本と正確に一致することを確認した。公式アップロード実行は 1 回、クラウドビルド・秘密鍵 / ソースフォルダーのアップロード・料金変更は 0 である。

공유 뒤에도 최종 소스164개 해시와 이전 차단 APK의 원본 해시가 유지됨을 대조했다. 근거는 apk-share.json·expo-upload-readback.json·delivery-completed.json·upload-authorization.json 및 record-completion.ps1이다. 기존 앱을 삭제하지 않고 업데이트한 뒤 설정→문의함에서 실제 본인 문의 작성·목록 반영·상세·답글과 운영 답변을 확인한다. 새 문의 흐름의 실기기 수락은 사용자 확인 대기이며 이전 차단·해제 수락과 구분한다.

共有後も最終ソース 164 件のハッシュと以前のブロック APK の原本ハッシュが維持されていることを照合した。根拠は apk-share.json・expo-upload-readback.json・delivery-completed.json・upload-authorization.json および record-completion.ps1 である。既存アプリを削除せず更新し、設定→問い合わせボックスで実際の自分の問い合わせ作成・一覧反映・詳細・返信と運営回答を確認する。新しい問い合わせフローの実機受け入れはユーザー確認待ちであり、以前のブロック・解除受け入れとは区別する。

## 사용자 확인 완료

## ユーザー確認完了

사용자가 전달된 문의함에 대해 문제가 없음을 확인했다고 직접 회신했다. 사용자 보고 PASS로 기록하며 구체적인 개별 시나리오·기기 바이너리의 독립 식별이나 전체 출시 수락으로 확대하지 않는다. device-user-acceptance.json과 delivery-completed.json에 반영하고 이전 완료 영수증과 기존 APK·코드·검증 자료는 보존했다.

ユーザーは提供された問い合わせボックスについて、問題がないことを確認したと直接回答した。ユーザー報告 PASS として記録し、具体的な個別シナリオ・端末バイナリの独立識別や全体リリース受け入れに拡大しない。device-user-acceptance.json と delivery-completed.json に反映し、以前の完了記録と既存 APK・コード・検証資料は保存した。
