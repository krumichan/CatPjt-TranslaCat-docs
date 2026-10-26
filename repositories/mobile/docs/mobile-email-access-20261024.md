# 모바일 이메일 가입·계정 복구 / モバイルのメール登録・アカウント復旧

## 구현 상태 / 実装状態

2026-10-24 개발 소스다. 데스크톱에서 lint·typecheck·전체 단위 테스트·설정·Android JS export를 통과했다. 공개 서버 경로 개방, 네이티브 APK 및 실제 메일 수신 검증은 별도 완료 근거가 필요하다. 이 문서만으로 운영 활성화를 의미하지 않는다.

2026-10-24の開発ソース。デスクトップでlint・typecheck・全単体テスト・設定・Android JS exportに通過した。公開サーバー経路の開放、ネイティブAPK、実際のメール受信には、それぞれ別の完了根拠が必要。この文書だけで本番有効化を意味しない。

## 화면과 계약 / 画面と契約

- 로그인에서 `/register`, `/password-reset` 화면으로 이동한다. 가입은 이메일·인증번호·닉네임·생년월일·비밀번호 확인을 사용한다. 생년월일은 `YYYY-MM-DD`다.

  ログインから`/register`、`/password-reset`へ移動する。登録にはメール・認証コード・ニックネーム・生年月日・パスワード確認を使用する。生年月日は`YYYY-MM-DD`。
- 가입: `POST /auth/email/challenges`, `/auth/email/verify`, `/auth/register`. 복구: `POST /auth/password-reset/challenges`, `/auth/password-reset/verify`, `/auth/password-reset/complete`. 모두 공개 BE HTTPS의 `/api/v1` 아래이며 Bearer를 보내지 않는다. 웹 BFF를 우회해 호출하지 않는다.

  登録：`POST /auth/email/challenges`、`/auth/email/verify`、`/auth/register`。復旧：`POST /auth/password-reset/challenges`、`/auth/password-reset/verify`、`/auth/password-reset/complete`。すべて公開BE HTTPSの`/api/v1`配下でBearerを送らない。Web BFFを迂回して呼び出さない。
- 인증번호·증명은 최대 10분, 재발송은 최소 60초이며 서버가 더 긴 대기 시간을 주면 유지한다. 이메일이 바뀌면 인증을 무효화한다. 비밀번호는 공백을 보존하고 8자 이상·UTF-8 72바이트 이하를 검사한다.

  コード・証明は最大10分、再送信は最低60秒とし、サーバーがより長い待機時間を返した場合も保持する。メール変更時に認証を無効化する。パスワードの空白を保持し、8文字以上・UTF-8で72バイト以下を検査する。
- 증명은 메모리에만 두고, 화면을 벗어나면 폐기한다. 메일 앱으로 이동할 때 진행 중 요청과 비밀번호는 지우되 이미 받은 challenge·미사용 증명의 만료 시각은 유지한다. 쓰기 자동 재시도·자동 로그인은 없다. 결과 불명확 시 로그인으로 결과를 먼저 확인한다.

  証明はメモリ内だけに保持し、画面を離れると破棄する。メールアプリへ移る際は実行中リクエストとパスワードを消去するが、取得済みchallenge・未使用証明の有効期限は維持する。書き込みの自動再試行・自動ログインは行わない。結果不明の場合は先にログインで結果を確認する。
- 비밀번호 변경은 기존 sid와 legacy refresh를 폐기한다. 재설정 이전 비밀번호 확인 및 이전 모바일 승인 요청으로 새 세션을 만들지 못하게 한다. Google 전용 계정은 복구로 비밀번호를 추가하지 않고 Google 로그인을 안내한다.

  パスワード変更は既存sidとlegacy refreshを失効させる。変更前のパスワード確認やモバイル承認リクエストから新セッションを作成させない。Google専用アカウントは復旧でパスワードを追加せず、Googleログインを案内する。

## 입력 화면 UX / 入力画面のUX

- 생년월일은 키보드 입력 대신 연도·월·날짜를 고르는 달력을 사용한다. 선택기는 임시 선택을 별도로 유지하고, 확인을 눌렀을 때만 `YYYY-MM-DD`로 반영한다. 취소·Android 뒤로 가기·로그인 세대 전환은 미적용 선택을 폐기한다. 기존 0001년부터 UTC 오늘까지의 날짜 계약을 유지하며 새 연령 제한을 만들지 않는다.

  生年月日はキーボード入力ではなく年・月・日を選ぶカレンダーを使用する。仮選択は確認するまでフォームに反映しない。キャンセル・Androidの戻る操作・ログイン世代の変更時は未確定の選択を破棄する。既存の西暦0001年からUTCの当日までの契約を維持し、新たな年齢制限は設けない。
- 입력 오류는 해당 입력 바로 아래에 표시하고 오류 입력으로 이동한다. 통신 오류는 해당 요청 버튼과 연결한다. 하단 완료 영역에도 오류를 표시해 화면 위로 돌아가지 않아도 결과를 알 수 있게 한다. 같은 오류를 다시 제출해도 안내하고, 무관한 입력을 수정했다는 이유로 오류를 지우지 않는다.

  入力エラーは対象入力の直下に表示して移動する。通信エラーは対象のリクエストボタンに関連付ける。画面下部にもエラーを示し、上部へ戻らなくても結果が分かるようにする。同じエラーを再送信した場合も案内し、無関係な入力を編集しても未解決のエラーを消さない。
- 가입·비밀번호 변경 버튼은 폼 스크롤과 분리한 하단 영역에 두고 키보드 위에서 사용할 수 있게 한다. 포커스된 입력은 키보드 전환 후에도 보이는 위치로 이동한다. 긴 오류는 하단 안내 안에서 따로 스크롤하며, 입력과 완료 버튼의 공간을 모두 차지하지 않게 한다.

  登録・パスワード変更ボタンはフォームのスクロールと分離した下部領域に置き、キーボードの上から操作できるようにする。フォーカス中の入力はキーボードの切り替え後も見える位置へ移動する。長いエラーは案内領域内でスクロールし、入力欄と完了ボタンの領域を奪わないようにする。

2026-10-24 공용 Mac 검증 job `20261024T103037277Z-mac-0f84ace2`에서 개선 화면을 포함한 고정 소스 206개의 lint·typecheck·전체 598개 테스트·설정·Android/iOS JS export·iOS 프로젝트 생성(`prebuild --no-install`)이 모두 종료 코드 0으로 통과했다. 입력·lockfile은 변경되지 않았고 소스 차이는 0개였다. 소스 manifest SHA-256은 `95ebd0f961adfe7db49ac3a841447c5bbad3015158478401583bd09e162c8a0d`다.

2026-10-24の共通Mac検証job `20261024T103037277Z-mac-0f84ace2`で、画面改善を含む固定ソース206ファイルのlint・typecheck・全598テスト・設定・Android/iOS JS export・iOSプロジェクト生成（`prebuild --no-install`）がすべて終了コード0で通過した。入力とlockfileに変更はなく、ソース差分は0件。ソースmanifestのSHA-256は`95ebd0f961adfe7db49ac3a841447c5bbad3015158478401583bd09e162c8a0d`。

이 UI의 개발일은 2026-10-24이며, 같은 고정 소스의 iOS native 검증은 2026-10-25 00:32~00:42 KST에 실행했다. Mac job `20261024T150125788Z-mac-00c8ef20`은 코드 서명 없는 Release arm64 Simulator 앱을 빌드·설치·기동하고 24개 단계를 모두 종료 코드 0으로 마쳤다. 실제 첫 화면에서 게스트 채팅 홈·로그인 버튼·하단 탭을 확인했고 오류 overlay는 보이지 않았다. 작업 전용 Simulator의 종료도 확인했다. 공용 중계의 읽기 파일 경합으로 한때 큐 상태가 `Unconfirmed`에 머물렀으나, 같은 JobId·token·결과 SHA를 대조한 복구로 2026-10-25 02:19:33 KST에 `Released`·종료 코드 0을 확인했다. 02:21 KST에는 같은 job의 정상 결과 회수도 완료했다. 앱 실행 성공 시각과 이후 공용 큐 해제 시각을 구분한다.

このUIの開発日は2026-10-24で、同じ固定ソースのiOS native検証は2026-10-25 00:32~00:42 KSTに実行した。Mac job `20261024T150125788Z-mac-00c8ef20`でコード署名なしのRelease arm64 Simulatorアプリをビルド・インストール・起動し、全24段階が終了コード0で完了した。実際の初期画面ではゲストのチャットホーム・ログインボタン・下部タブを確認し、エラーoverlayは見られなかった。専用Simulatorの終了も確認済み。共通中継の読み取りファイル競合でキュー状態が一時`Unconfirmed`に留まったが、同じJobId・token・結果SHAを照合した復旧により2026-10-25 02:19:33 KSTに`Released`・終了コード0を確認した。02:21 KSTには同じjobの通常結果回収も完了した。アプリ起動成功の時刻と、その後の共通キュー解放時刻を区別する。

598テストを通過したJS検証はAPI未指定、Simulator nativeビルドは公開API `https://217.142.230.186/api/v1`を実行設定へ明示した。APIは生成設定とアプリ内のExpo設定から確認しており、環境の異なるbundleを同一成果物とは扱わない。詳細な入力・成果物の根拠は[iOS無料準備](mobile-ios-free-preparation-20261024.md)に記録する。

598개 테스트를 통과한 JS 검증은 API 미지정, Simulator native 빌드는 공개 API `https://217.142.230.186/api/v1`을 실행 설정에 명시했다. API는 생성 설정과 앱 내부 Expo 설정에서 확인했으며, 환경이 다른 bundle을 동일 산출물로 취급하지 않는다. 입력·산출물의 상세 근거는 [iOS 무료 준비](mobile-ios-free-preparation-20261024.md)에 기록한다.

사용자는 이번 UI 변경의 새 APK 실기기 테스트를 생략했다(`WAIVED_BY_USER_NOT_TESTED`). 이 개선을 담은 새 Android APK는 만들지 않았다. Simulator 홈 화면 확인을 생년월일 선택·오류 표시·키보드 동작의 실제 기기 수락으로 확대하지 않는다. Apple 로그인은 미구현이고 운영 APNs·실제 계정 가입·메일 발송·비밀번호 변경도 이번 검증에서 확인하지 않았다.

ユーザーは今回のUI変更について新APKの実機テストを省略した（`WAIVED_BY_USER_NOT_TESTED`）。この改善を含む新Android APKは作成していない。Simulatorのホーム画面確認を生年月日選択・エラー表示・キーボード動作の実機受け入れへ拡大しない。Appleログインは未実装で、本番APNs・実際のアカウント登録・メール送信・パスワード変更も今回の検証対象には含めていない。

## 문서 연결 / 文書へのリンク

로그인·가입·복구·설정은 공통 `LegalLinks`를 사용한다. `policy-links.ts`에는 확인한 공개 HTTPS 문서만 등록한다. 현재 운영 FE에는 실제 공개 약관·개인정보처리방침 페이지가 없어 두 주소는 `null`이고 버튼도 비활성이다. 가짜 URL이나 미검토 문안으로 대체하지 않는다. 이용약관·개인정보처리방침 초안은 로컬 검토용으로 보관하며 공개 저장소의 반영 대상에서 제외한다. 정식 게시·시행·동의는 완료되지 않았다. 공개 문서 확정은 테스트 APK 준비와 구분한다.

ログイン・登録・復旧・設定は共通の`LegalLinks`を使用する。`policy-links.ts`には確認済みの公開HTTPS文書のみ登録する。現在の本番FEには実際の公開利用規約・プライバシーポリシーページがないため、両URLは`null`でボタンも無効。利用規約・プライバシーポリシー案はローカル確認用として保持し、公開リポジトリへの反映対象から除外する。正式公開・施行・同意は未完了。公開文書の確定とテストAPKの準備は区別する。

実行手順・DB・公開境界は[BE復旧契約](../../be/docs/password-reset-20261024.md)を参照。

실행 순서·DB·공개 경계는 [BE 복구 계약](../../be/docs/password-reset-20261024.md)을 참고한다.


## 2026-10-25 큰 글씨·키보드·날짜 선택기 확인 완료 / 大きい文字・キーボード・日付選択の確認完了

24일 이메일 폼 UI와 25일 후속 접근성 보완을 구분한다. 25일 변경은 `EmailAccessScreen.tsx`의 긴 완료 안내를 스크롤 본문으로 옮기고 고정 완료 버튼에만 2배 글자 확대 한도를 지정한 것, 공통 `Button`에 선택적 `maxFontSizeMultiplier` prop을 추가한 것 두 파일이다. 다른 호출부의 기본 글자 확대와 인증·메일·API 로직은 바꾸지 않았다. 원본과 25일 두 파일 전용 patch를 별도 보존했다.

24日のメールフォームUIと25日の追加アクセシビリティ修正を分けて記録する。25日の変更は2ファイルのみ。長い完了案内をスクロール本文へ移し、固定完了ボタンだけ文字拡大を2倍までにし、共通Buttonに任意の `maxFontSizeMultiplier` propを追加した。他の呼出元の既定拡大と認証・メール・APIロジックは変えていない。原本と25日分の2ファイル専用patchを保全した。

固定source206のmanifest SHA256は `a26d6c4d28d6765067a7d8b2f62d62c31c0583b799fa84ea38e24c1dfc22bd7c`。source check229 `20261025T092126995Z-mac-80f59473` はlint/typecheck/598tests（失敗・skip0）/config/Android+iOS JS export/prebuild-no-installを新たに通過した。旧sourceの598件結果の流用ではない。任意のglobal Xcode probeのexit1は保持し、明示DEVELOPER_DIRのnative成功と区別する。

새 고정 소스 206개의 manifest SHA256은 `a26d6c4d28d6765067a7d8b2f62d62c31c0583b799fa84ea38e24c1dfc22bd7c`이다. source check229 `20261025T092126995Z-mac-80f59473`에서 lint/typecheck/598 tests(실패·생략0)/config/Android+iOS JS export/prebuild-no-install을 새로 통과했다. 이전 소스의 결과를 재사용한 것이 아니다. 선택적 전역 Xcode probe exit1을 보존하고 명시 DEVELOPER_DIR로 실행한 native 성공과 구분한다.

새 native230 `20261025T092330629Z-mac-8862fce0`은 Release arm64 Simulator 빌드·기동·전용 UUID 종료와 전체 24단계 exit0/소스 차이0을 확인했다. 원본 결과 SHA256은 `d5f892b4fbddfe55a2bf4c9a5f4f0feb77cb8b82875bb93fe99db64c8dcacdfd`, 앱 ZIP은 `b1a353d70945e40af99c8a57c5241666f8fdf05ce5121a9e5318d9a33b152645`이다. Mach-O arm64/Simulator platform7·내장 공개 API·새 초기 게스트 화면을 독립 대조했다. 공용 proxy의 Unconfirmed는 제품 성공과 별도로 보존했으며 담당의 원소유자 전용 종료 복구 후 18:51:51 KST에 Released, 18:54:15 KST에 같은 job의 정상 결과 회수를 확인했다. 원인은 NonTransientProxyObservationFailure까지만 관측했고 공유 위반 등으로 단정하지 않는다.

native230 `20261025T092330629Z-mac-8862fce0` は新sourceのRelease arm64 Simulatorビルド・起動・専用UUID終了と全24段階exit0/source差分0を確認した。原結果SHA256は `d5f892b4fbddfe55a2bf4c9a5f4f0feb77cb8b82875bb93fe99db64c8dcacdfd`、app ZIPは `b1a353d70945e40af99c8a57c5241666f8fdf05ce5121a9e5318d9a33b152645`。Mach-O arm64/Simulator platform7・埋込公開API・新しい初期guest画面を独立照合した。共通proxyのUnconfirmedは製品成功と分けて保全し、担当の元所有者限定の終了復旧後18:51:51 KSTにReleased、18:54:15 KSTに同jobの通常回収を確認した。原因はNonTransientProxyObservationFailureまでの観測で、共有違反等と断定しない。

UI231 `20261025T095615976Z-mac-42167a49` はログイン・登録・パスワード復旧の3画面×3設定の経路assertion9件と通常文字のキーボード2件を通過した。その後、最大文字でWDAのscroll-to-visible helperがmax scroll countに達してFailed1となった原結果を保持する。UI233 `20261025T100656602Z-mac-4b7c5653` は同じ新appの残り4件だけを、元231のhash付き部分結果を照合して実行しPassed0。自分のScrollView内の限定touch panに替え、最大文字でのキーボード2件と日付pickerのopen/cancel2件を確認した。598tests/native/経路9件/通常キーボード2件を再実行していない。

UI231 `20261025T095615976Z-mac-42167a49`에서 로그인·가입·비밀번호 복구의 3화면×3설정 경로 assertion 9건과 기본 글자 크기 키보드 2건을 통과했다. 이후 최대 글자 크기에서 WDA scroll-to-visible helper가 max scroll count에 도달한 원래 Failed1 결과를 보존한다. UI233 `20261025T100656602Z-mac-4b7c5653`은 같은 새 앱의 남은 4건만, 원231의 해시가 고정된 부분 결과를 대조해 실행하여 Passed0이었다. 자기 ScrollView 안의 제한된 touch pan으로 바꿔 최대 글자 크기 키보드 2건과 날짜 선택기 열기·취소 2건을 확인했다. 소스 검사/native/기존 9화면/기본 키보드를 반복하지 않았다.

기본·최대 글자 크기에서 가입/복구 모두 이메일 입력란과 완료 버튼이 소프트웨어 키보드 위에 있고 서로 겹치지 않았다. 최대 글자 크기 측정은 입력란 하단247pt ≤ 버튼 상단408pt, 버튼 하단483pt < 키보드 상단512pt였다. 날짜 선택기는 정상적으로 열렸고 취소 후 날짜 미선택 placeholder로 돌아왔다. 실제 PNG 15장(초기9+키보드4+picker2)을 직접 검토했다. OS dark 설정에서도 앱 표면은 밝게 표시되어 앱의 어두운 테마 수락을 의미하지 않는다. 큰 제목·본문은 확대·스크롤하며 고정 버튼만 크기를 제한한다.

通常・最大文字の登録/復旧の両方でメール入力と完了ボタンがsoftware keyboardの上にあり、互いに重ならないことを確認した。最大文字では入力下端247pt ≤ ボタン上端408pt、ボタン下端483pt < keyboard上端512pt。pickerは開き、cancel後は未選択placeholderに戻った。実PNG15枚（初期9+keyboard4+picker2）を直接確認した。OS dark設定でもアプリ表面はlightであり、アプリdark themeの受入を意味しない。大きい見出しと本文は拡大・スクロールし、固定ボタンのみ拡大を制限する。

233のSSH timeoutは新jobやworker失敗の根拠にせず同じJobIdのCollectで回収した。原結果SHA256は `cde183cbb8adf45f7a2b5895994e5b3d1cdc4da87c82e249234396b1516c47a2`。終了・共通Released・通常回収・専用Simulator Shutdown・自分のAppium停止・proxy定義削除を確認した。原因を電源/睡眠/IP変更と断定せず、他job・順番・予算・queue/lockの手編集や他processの停止は行っていない。

233의 SSH timeout을 새 job이나 worker 실패의 근거로 삼지 않고 같은 JobId의 Collect로 회수했다. 원본 결과 SHA256은 `cde183cbb8adf45f7a2b5895994e5b3d1cdc4da87c82e249234396b1516c47a2`이다. 종료·공용 Released·정상 회수·전용 Simulator Shutdown·자기 Appium 종료·proxy 정의 삭제를 확인했다. 원인을 전원/수면/IP 변경으로 단정하지 않으며 다른 job·순번·예산·큐/잠금 수동 편집이나 다른 프로세스 종료는 하지 않았다.

이번 확인은 이메일 입력 포커스·스크롤·picker 열기/취소까지다. 실제 메일·가입·비밀번호 변경·탈퇴, 날짜의 연/월/일 전체 선택·적용, 실제 오류 메시지 UI, Google/Apple 로그인 복귀, 운영 APNs, iPhone 실기기/IPA/TestFlight 수락으로 확대하지 않는다. Apple 로그인은 미구현이다. 새 Android APK는 생성하지 않았으며 실기기 검증은 사용자의 `WAIVED_BY_USER_NOT_TESTED`를 유지한다. 유료 Apple/EAS·운영/Git 변경·실제 자격/서비스계정 전송은 0건이고 정책 URL null·법적 초안 local/public 제외를 유지한다.

範囲はメール入力focus/scrollとpicker open/cancelまで。実メール・登録・パスワード変更・退会、日付の年月日全選択/適用、実エラー表示UI、Google/Appleログイン復帰、本番APNs、iPhone実機/IPA/TestFlight受入とは区別する。Appleログインは未実装。新Android APKは作成せず、実機はユーザーの `WAIVED_BY_USER_NOT_TESTED` を維持する。有料Apple/EAS・本番/Git変更・実資格/サービスaccount転送は0件、policy URL nullと法的draftのlocal/public除外も維持する。

검증 원본·실패 이력·날짜별 patch·결합 확인 근거는 `.codex-workspace/verification/shared/mobile-ios-interactive-smoke-20261025/`에 보존한다. 6GiB 점검 예산은 새 범위의 추정치이며 RSS 표본은 공용 CoreSimulator를 제외하므로 실제 총 메모리 피크로 주장하지 않는다. 이 무료 Simulator 단계에는 추가 사용자 직접 조치가 없다.

原本・失敗履歴・日付別patch・結合確認は `.codex-workspace/verification/shared/mobile-ios-interactive-smoke-20261025/` に保全する。6GiB予算は新しい確認範囲の推定であり、RSS標本は共有CoreSimulatorを除くため総メモリpeakとは扱わない。この無料Simulator段階に追加ユーザー操作はない。
