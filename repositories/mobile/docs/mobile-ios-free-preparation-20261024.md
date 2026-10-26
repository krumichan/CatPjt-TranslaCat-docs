# iOS 무료 로컬 준비 / iOS 無料ローカル準備

2026-10-24 사용자는 맥미니가 사용 가능해지면 무료 범위의 iOS 준비를 먼저 진행하도록 요청했다. 현재 앱은 Expo `~58.0.14`, React Native `0.88.0`이다. 기존 Android 기능과 가입 화면 개선을 보존한 소스 사본을 사용한다.

2026-10-24、ユーザーはMac miniの運用開始後、無料の範囲でiOSの準備を先に進めるよう依頼した。現在のアプリはExpo `~58.0.14`、React Native `0.88.0`。既存Android機能と登録画面改善を保持したソースコピーを使う。

## 준비 범위 / 準備範囲

먼저 lint·타입·전체 단위 테스트·공개 설정·Android/iOS JS export를 실행한다. 다음으로 별도 실행용 사본에서 iOS 프로젝트를 생성하고 식별자·URL scheme·사진 권한·entitlement를 확인한다. 지원되는 Xcode·CocoaPods·시뮬레이터 런타임이 준비되면 코드 서명을 사용하지 않는 Simulator 빌드와 비파괴 화면 확인을 진행한다. 각 단계의 실제 종료 코드와 입력 해시를 따로 남긴다. JS export나 native 프로젝트 생성만으로 iOS 앱 기동 성공을 뜻하지 않는다.

最初にlint・型・全単体テスト・公開設定・Android/iOS JS exportを実行する。次に別の実行用コピーでiOSプロジェクトを生成し、識別子・URL scheme・写真権限・entitlementを確認する。対応Xcode・CocoaPods・Simulator runtimeが揃った後、コード署名なしのSimulatorビルドと非破壊の画面確認を行う。終了コードと入力ハッシュを段階別に記録する。JS exportやプロジェクト生成だけではiOS起動成功とは扱わない。

Mac上のSimulator用ローカルビルドには有料Apple Developer登録やTestFlight配布は必要ない。成果物はSimulator用`.app`であり、個人iPhoneへ共有リンクでインストールする`.ipa`ではない。[Expo ローカルビルド / Expo 로컬 빌드](https://docs.expo.dev/guides/local-app-development/)

Mac의 시뮬레이터용 로컬 빌드는 유료 Apple Developer 등록이나 TestFlight 배포 없이 진행한다. 산출물은 시뮬레이터용 `.app`이며 개인 iPhone에 링크로 설치하는 `.ipa`와 구분한다.

## 도구와 실행 경계 / ツールと実行境界

Expo SDK58의 공식 지원 기준은 Node `22.13.x` 이상, iOS `16.4` 이상, Xcode `26.4` 이상이다. 게임 작업에서 사용하는 Xcode `26.1.1` 권장값은 이 앱의 지원 기준을 충족하지 않는다. 기존 도구를 보존하고, 앱 작업 프로세스의 `DEVELOPER_DIR`로 별도 Xcode를 선택한다. 전역 `xcode-select`, PATH와 NVM 설정을 바꾸지 않는다. [Expo SDK58 요구사항 / Expo SDK58 要件](https://docs.expo.dev/versions/v58.0.0/)

Expo SDK58の公式対応条件はNode `22.13.x`以上、iOS `16.4`以上、Xcode `26.4`以上。ゲーム向けXcode `26.1.1`の推奨値は本アプリの条件を満たさない。既存ツールを保持し、ジョブの`DEVELOPER_DIR`で別のXcodeを選ぶ。グローバル`xcode-select`・PATH・NVMは変更しない。

공용 [Mac 실행 운영 문서](../../shared/docs/mac-worker.md)를 적용한다. 도구 설치 확인과 공용 실행기 검증을 구분하며, 담당자의 실제 준비 완료 근거를 확인한 뒤 작업을 한 번 제출한다. 원본은 노트북에 보존하고 공개 소스·lockfile만 전송한다. 실제 `.env`, 서명키·APNs 자격·서비스계정·실사용자 데이터·다른 PC의 `node_modules`를 복사하지 않는다. 연결이 끊기면 같은 JobId를 회수하고 다른 작업·예약·큐를 임의로 중단하거나 삭제하지 않는다.

共通[Mac実行運用文書](../../shared/docs/mac-worker.md)に従う。ツールのインストールと共通実行経路の検証を区別し、担当者の実証後に一度だけ投入する。原本はノートPCへ保持し、公開ソースとlockfileだけを転送する。秘密設定・署名鍵・APNs認証情報・実ユーザーデータ・他PCの依存関係はコピーしない。接続断では同じJobIdを回収し、他作業・予約・キューを停止または削除しない。

現在の`build:development:ios`・`build:preview:ios`・`build:production:ios`は`eas build`を呼ぶクラウド用コマンドである。今回の無料ローカル準備には使わない。`npm run ios`は開発アプリ用Metro起動であり、初回nativeビルド完了の代わりにはならない。

현재 `build:development:ios`·`build:preview:ios`·`build:production:ios`는 `eas build`를 호출하는 클라우드 명령이다. 이번 무료 로컬 준비에 사용하지 않는다. `npm run ios`는 개발 앱용 Metro 기동이며 최초 native 빌드 완료를 대신하지 않는다.

## 별도 준비 항목 / 別途準備する項目

- 전체 Xcode의 최초 실행·라이선스·런타임 설치에서 사용자 로그인이 필요한 경우 맥 사용자에게 정확한 직접 조치를 안내한다. 비밀번호를 채팅으로 받지 않는다. [무료 Xcode / 無料Xcode](https://apps.apple.com/app/xcode/id497799835), [Simulator 준비 / Simulator準備](https://docs.expo.dev/workflow/ios-simulator/)
- Apple 로그인은 현재 구현돼 있지 않다. 이번 단계에서 유료 capability·팀·키 값을 만들거나 로그인 완료로 표시하지 않는다. [Apple 기능 안내 / Apple機能案内](https://developer.apple.com/programs/whats-included/)
- 현재 앱은 실제 기기에서만 푸시 토큰을 등록하도록 구성돼 있다. Simulator 검증에서 이 조건을 임의로 완화하거나 운영 APNs 성공으로 확대하지 않는다. 호환 Simulator 자체의 원격 알림 지원과 앱의 실제 자격·등록 정책은 구분한다. [Expo 푸시 준비 / Expo Push準備](https://docs.expo.dev/push-notifications/push-notifications-setup/)
- TestFlight·App Store 배포·유료 개발자 등록은 이번 무료 범위에 포함하지 않는다. 이용약관·개인정보처리방침 초안 공개와 운영 설정 변경도 이 작업에 포함하지 않는다.

Xcode初回設定でログインやライセンス操作が必要なら、具体的な操作をMacユーザーへ案内し、パスワードは受け取らない。未実装のAppleログイン・本番APNs・TestFlight・App Store・有料登録は今回の完了条件に含めない。規約草案の公開や本番設定も変更しない。

## 검증 상태 / 検証状態

2026-10-24 19:31 KST에 공용 Mac job `20261024T103037277Z-mac-0f84ace2`가 `Passed`·종료 코드 0으로 완료됐고 큐 예약도 해제됐다. 고정 소스 206개로 lint·typecheck·전체 598개 테스트(실패·생략 0개)·설정·Android/iOS JS export·iOS `prebuild --no-install`을 모두 통과했다. 입력·lockfile 변경과 소스 차이는 0개다. 소스 manifest SHA-256은 `95ebd0f961adfe7db49ac3a841447c5bbad3015158478401583bd09e162c8a0d`다.

2026-10-24 19:31 KST、共通Mac job `20261024T103037277Z-mac-0f84ace2`が`Passed`・終了コード0で完了し、キューの予約も解放された。固定ソース206ファイルでlint・typecheck・全598テスト（失敗・省略0件）・設定・Android/iOS JS export・iOS `prebuild --no-install`がすべて通過した。入力・lockfileの変更とソース差分は0件。ソースmanifestのSHA-256は`95ebd0f961adfe7db49ac3a841447c5bbad3015158478401583bd09e162c8a0d`。

実際に生成したプロジェクトではscheme `TranslaCatDev`、bundle identifier `com.krumichan.translacat.dev`、deployment target `16.4`、URL scheme `translacat-dev`・`com.krumichan.translacat.dev`・`exp+translacat-mobile`を確認した。写真ライブラリーの利用説明は設定済みで、カメラ・マイクの利用説明はない。既存通知設定から生成された`aps-environment: development`はAPNs認証情報の発行や本番通知の成功を示さない。

실제 생성한 프로젝트에서 scheme `TranslaCatDev`, bundle identifier `com.krumichan.translacat.dev`, deployment target `16.4`, URL scheme `translacat-dev`·`com.krumichan.translacat.dev`·`exp+translacat-mobile`을 확인했다. 사진 라이브러리 이용 설명은 있고 카메라·마이크 이용 설명은 없다. 기존 알림 설정에서 생성된 `aps-environment: development`는 APNs 자격 발급이나 운영 알림 성공을 의미하지 않는다.

## 2026-10-25 Simulator 실행 결과 / Simulator実行結果

가입 UI와 고정 소스의 개발일은 2026-10-24다. native 검증은 2026-10-25 00:32~00:42 KST에 별도로 실행했다. Mac job `20261024T150125788Z-mac-00c8ef20`(FIFO 117)의 worker 결과는 `Passed`·종료 코드 0이며, 의존성 설치·Pod 설치·Release arm64 빌드·Simulator 설치·기동·스크린샷을 포함한 24개 단계가 모두 종료 코드 0으로 끝났다. 입력은 검증된 소스 206개와 공개 제어 파일 1개이며, 입력 변경과 소스 차이는 0개였다.

登録UIと固定ソースの開発日は2026-10-24。native検証は2026-10-25 00:32~00:42 KSTに別途実行した。Mac job `20261024T150125788Z-mac-00c8ef20`（FIFO 117）のworker結果は`Passed`・終了コード0で、依存関係・Podインストール・Release arm64ビルド・Simulatorインストール・起動・スクリーンショットを含む全24段階が終了コード0で完了した。入力は検証済みソース206ファイルと公開制御ファイル1個で、入力変更とソース差分は0件。

사용한 도구는 작업 프로세스에서 선택한 Xcode `26.6`(17F113), Ruby `3.3.13`, CocoaPods `1.17.0`이다. iOS `26.5`(23F77) runtime의 iPhone 17 Pro를 전용 device set에서 기동했다. 전역 Xcode 선택은 바꾸지 않았고 코드 서명·유료 등록·클라우드 빌드·자격 발급은 하지 않았다. 실제 스크린샷에서 TranslaCat 게스트 채팅 홈·로그인 버튼·하단 탭을 확인했으며 오류 overlay는 없었다. 확인 뒤 해당 작업 전용 Simulator가 `Shutdown` 상태임을 확인했다.

使用ツールはジョブ内で選択したXcode `26.6`（17F113）、Ruby `3.3.13`、CocoaPods `1.17.0`。iOS `26.5`（23F77）runtimeのiPhone 17 Proを専用device setで起動した。グローバルXcode選択は変えず、コード署名・有料登録・クラウドビルド・認証情報の発行は行っていない。実際のスクリーンショットではTranslaCatのゲストチャットホーム・ログインボタン・下部タブを確認し、エラーoverlayはなかった。確認後、当該ジョブ専用Simulatorが`Shutdown`状態になったことも確認した。

이번 source check의 API 설정은 `UNSET_IN_EXACT_CHECKS_ADAPTER`였다. native 빌드에서는 공개 API `https://217.142.230.186/api/v1`을 프로세스 환경에만 지정하고, 생성된 공개 설정과 `.app` 내부 `EXConstants.bundle/app.config`에서 일치함을 확인했다. 실제 `.env` 전송은 0개다. JS 코드의 API는 `Constants.expoConfig.extra.apiBaseUrl`에서 읽으므로 `main.jsbundle`에 URL 문자열이 직접 없다는 관측과 구분한다. 설정 환경이 다른 source-check bundle과 native bundle을 동일 산출물로 주장하지 않는다.

source checkのAPI設定は`UNSET_IN_EXACT_CHECKS_ADAPTER`だった。nativeビルドでは公開API `https://217.142.230.186/api/v1`をプロセス環境だけに指定し、生成公開設定と`.app`内の`EXConstants.bundle/app.config`で一致を確認した。実際の`.env`転送は0件。JSは`Constants.expoConfig.extra.apiBaseUrl`からAPIを読むため、`main.jsbundle`にURL文字列が直接ないという観測とは区別する。設定環境の異なるsource-check bundleとnative bundleを同一成果物とは扱わない。

| 근거 / 根拠 | 값 / 値 |
| --- | --- |
| native JobId | `20261024T150125788Z-mac-00c8ef20` |
| source manifest SHA-256 | `95ebd0f961adfe7db49ac3a841447c5bbad3015158478401583bd09e162c8a0d` |
| 실행 어댑터 / 実行adapter SHA-256 | `cab458fe17605e539dc7f5412089af01424b024cfb7bf1dbb55da882d63ddee0` |
| worker result SHA-256 | `968341b546b4607be035c4221b6d56456cabe9078703d365c4237f352f7e5810` |
| `translacat-ios-simulator-arm64.app.zip` SHA-256 | `bfa5d686379355cfb60881a5a8ff1e131a57ab42599cb51e887fb2bc8fc1f888` |
| `translacat-ios-first-launch.png` SHA-256 | `47d1bd6946aeb2e27a5586ab789c0b643b2347ac6c206cf85c5b802e81d6157d` |

산출물은 코드 서명 없는 Simulator용 Release arm64 `.app.zip`이며 iPhone 설치용 `.ipa`나 TestFlight 배포본이 아니다. 앱 빌드·기동과 전용 Simulator 종료는 00:32~00:42 KST에 성공했다. 공용 proxy의 읽기 파일 경합으로 한때 큐 상태가 `Unconfirmed`에 머물렀으나, 정확한 같은 JobId·token·종료 코드 0·결과 SHA를 대조한 terminal 복구 후 2026-10-25 02:19:33 KST에 `Released`를 확인했다. 같은 job의 정상 결과는 02:21 KST에 회수했다. 최종 worker 상태는 `Finished`·소유 실행 프로세스 0개이며, 해당 proxy 작업 정의도 정리됐다. 담당자의 복구는 해당 소유 proxy만 대상으로 했고 다른 job 변경이나 Mac 프로세스 중단은 없었다. 앱 성공과 뒤늦은 공용 큐 해제를 별도 근거로 남긴다.

成果物はコード署名なしのSimulator用Release arm64 `.app.zip`で、iPhoneインストール用`.ipa`やTestFlight配布版ではない。アプリのビルド・起動と専用Simulatorの終了は00:32~00:42 KSTに成功した。共通proxyの読み取りファイル競合でキュー状態が一時`Unconfirmed`に留まったが、正確な同一JobId・token・終了コード0・結果SHAを照合したterminal復旧後、2026-10-25 02:19:33 KSTに`Released`を確認した。同じjobの通常結果は02:21 KSTに回収した。最終worker状態は`Finished`・所有する実行プロセス0件で、当該proxyのタスク定義も整理済み。担当者の復旧対象は当該所有proxyのみで、他jobの変更やMacプロセス停止はなかった。アプリ成功と後続の共通キュー解放は別々の根拠として残す。

ログイン復帰・写真選択・生年月日picker・エラー表示・キーボード動作を今回のホーム画面確認だけで受け入れ済みとは扱わない。Appleログインは未実装、本番APNsは未検証。実際の登録・メール送信・パスワード変更・退会操作は0件で、実ユーザーデータや本番設定も変更していない。新Android APKは作成せず、ユーザーが省略した実機テストは`WAIVED_BY_USER_NOT_TESTED`のまま保持する。

로그인 복귀·사진 선택·생년월일 picker·오류 표시·키보드 동작을 이번 홈 화면 확인만으로 수락 완료로 표시하지 않는다. Apple 로그인은 미구현이고 운영 APNs는 미검증이다. 실제 가입·메일 발송·비밀번호 변경·탈퇴는 0건이며 실사용자 데이터나 운영 설정도 변경하지 않았다. 새 Android APK는 만들지 않았고 사용자가 생략한 실기기 테스트는 `WAIVED_BY_USER_NOT_TESTED`로 유지한다.

작업별 소스 고정·공식 요건 조사·실제 결과는 현재 workspace의 `.codex-workspace/verification/shared/mobile-ios-free-preparation-20261024/`에 보존한다. 이 문서에 준비 절차가 있다는 사실만으로 공용 실행기·시뮬레이터 빌드가 완료된 것은 아니다.

ソース固定・公式要件調査・実結果はworkspaceの`.codex-workspace/verification/shared/mobile-ios-free-preparation-20261024/`へ保全する。本書の手順だけで共通実行器やSimulatorビルドが完了したとは扱わない。


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

## 2026-10-27 무료 iPhone 실기기 설치 / 無料iPhone実機インストール

24일 이메일 UI·25일 접근성 수정과 27일 실기기 검증을 구분한다. 새 제품 소스 변경은 0개다. 최신 검증 소스206(manifest `a26d6c4d28d6765067a7d8b2f62d62c31c0583b799fa84ea38e24c1dfc22bd7c`)과 원본/고정 사본의 해시가 일치했다. 이전598검사와 Simulator 빌드를 반복하지 않았다.

24日のメールUI・25日のアクセシビリティ修正と27日の実機検証を区別する。新規の製品ソース変更は0件。最新の検証ソース206（manifest `a26d6c4d28d6765067a7d8b2f62d62c31c0583b799fa84ea38e24c1dfc22bd7c`）と原本/固定コピーのhash一致を確認した。既存598検査・Simulatorビルドは繰り返していない。

공용 Mac #363 `20261026T231240881Z-mac-0a176e59`에서 08:12~08:17 KST에 iPhoneOS Release arm64 빌드와 strict/deep 서명 검사가 exit0이었다. 뒤이어 공개 인증서 추출의 optional CLI 인자를 분리한 검증도구 오류로 worker는 Failed1이 됐다. 원래 실패/성공 단계·결과를 보존했고, `--extract-certificates=<own prefix>`로 고친 후 #364 `20261026T232313013Z-mac-d79c181f`는 같은 앱의 검증·설치·기동·readback만 실행했다. #364에서 새 빌드·재서명은0이다.

共通Mac #363 `20261026T231240881Z-mac-0a176e59`は08:12~08:17 KSTにiPhoneOS Release arm64ビルドとstrict/deep署名検証がexit0だった。後続の公開証明書抽出でoptional CLI引数を分離した検証ツールの誤りによりworkerはFailed1となった。原失敗/成功段階を保全し、`--extract-certificates=<own prefix>`に修正した#364 `20261026T232313013Z-mac-d79c181f`では同一appの検証・インストール・起動・readbackのみ実行した。#364の新規ビルド・再署名は0件。

08:23 KST에 동일 USB iPhone으로 `TranslaCat (Dev)` (`com.krumichan.translacat.freetest`, version0.1.0) 설치와 해당bundle readback이 exit0이었다. 실제 Mach-O arm64/iPhoneOS platform2, 기존 인증서 일치, profile의 동일 기기·team·정확한 app identifier·유효기간, 내장 공개 API `https://217.142.230.186/api/v1`과 JS bundle을 확인했다. 유료 등록 없이 기존 Personal Team 인증서/자동 provisioning을 사용했다. 무료 테스트 사본의 생성된 native entitlement에서만 APNs를 제외했으며 제품의 푸시 소스는 바꾸지 않았다.

08:23 KST、同じUSB iPhoneへの`TranslaCat (Dev)`（`com.krumichan.translacat.freetest`、version0.1.0）のインストールと該当bundleのreadbackがexit0だった。Mach-O arm64/iPhoneOS platform2、既存証明書一致、profileの同一端末・team・正確なapp identifier・有効期限、埋込公開API `https://217.142.230.186/api/v1`とJS bundleを確認した。有料登録なしで既存Personal Team証明書/自動provisioningを使用。無料テストコピーの生成native entitlementからのみAPNsを除外し、製品pushソースは変更していない。

実行要求はSecurity/RequestDeniedでexit1となり、OSは署名・entitlement・未信頼profileの可能性を併記した。ローカル署名/profile検証は通過したため、iPhoneの「設定→一般→VPNとデバイス管理→自分の開発者App→信頼」を案内した。その後ユーザー本人から「信頼してappが開いた」と確認を得た。同じインストール済みappの実機手動起動は確認済みで、自動launchのexit1を事後成功へ書き換えない。#364全体はFailed1のまま保持し、インストール成功を起動成功・ユーザー受入完了へ拡大しない。再ビルド・再インストールは必要条件にしない。

실행 요청은 Security/RequestDenied로 exit1이었고 OS는 서명·entitlement·미신뢰 profile 가능성을 함께 표시했다. 로컬 서명/profile 검증은 통과했으므로 iPhone의 ‘설정→일반→VPN 및 기기 관리→본인 개발자 앱→신뢰’를 안내했다. 이후 사용자가 직접 ‘신뢰했고 앱이 열렸어’라고 확인했다. 설치된 같은 앱의 실기기 수동 기동은 확인됐으며, 자동 launch exit1을 사후 성공으로 바꾸지는 않는다. #364 전체는 Failed1을 유지하며 설치 성공을 실행 성공·사용자 수락으로 확대하지 않는다. 재빌드·재설치를 선행조건으로 만들지 않는다.

| 근거 / 根拠 | 값 / 値 |
| --- | --- |
| Native363 original result SHA256 | `ef0dd87e974bd5ff8133b5d5688e1e03097f20a0f54ae4a30c351d256b6fafbd` |
| Finalization364 result SHA256 | `fbff22ea932d98174c8bc5793fb5bf24ca4be31da0f9551c27f6fef028a5f89d` |
| Signed executable SHA256 | `91cb4a7cfe5d59d23bb34b746abe3bb1bcbe4af4b3da07fcdda845615fa8d639` |
| Embedded `main.jsbundle` SHA256 | `baf3a9157244aaa8eb13fafcd54456ca74d2b3f591d73066f58fe059188ba527` |
| Signed app file inventory SHA256 | `da8b0b94e3069be5001665f976bfac86b6d97d36db70db570f3d4412086b6e7d` (162files / 44,726,557 logical bytes) |
| Profile expiry KST / 有効期限 | `2026-11-02 22:15:50 +09:00` |

363/364는 모두 실제 종료·공용 Released·정상 회수·Aqua helper/service 종료·자기 proxy 삭제를 확인했다. 기존 source-check API미지정과 native API·free identifier 설정 차이를 보존한다. 실기기 수동 기동은 사용자 확인을 받았다. 구체적인 화면 동작·메일/가입/비밀번호 변경/탈퇴·Apple로그인·APNs는 이번 확인 범위 밖이며 TestFlight/AppStore/유료EAS/새 Android APK는 실행하지 않았다. Android 실기기는 사용자의 `WAIVED_BY_USER_NOT_TESTED` 그대로다. Git·운영DB/BE/FE/edge·전역SSH/Xcode·키체인 정책을 바꾸지 않았고 개인키/실제.env를 전송하지 않았다. 법적 초안 local/public 제외와 정책URL null을 유지한다.

363/364は終了・共通Released・通常回収・Aqua helper/service終了・自己proxy削除を確認済み。source-checkのAPI未指定とnativeのAPI/free identifier設定差を保持する。実機の手動起動はユーザー確認済み。詳細な画面動作・実メール/登録/パスワード変更/退会・Appleログイン・APNsは今回の確認範囲外。TestFlight/AppStore/有料EAS/新Android APKは実行していない。Android実機はユーザーの`WAIVED_BY_USER_NOT_TESTED`を維持。Git・本番DB/BE/FE/edge・グローバルSSH/Xcode・キーチェーンpolicy変更、秘密鍵/実.env転送は0件。法的draftのlocal/public除外・policy URL nullも維持する。

原結果・署名app inventory・所有path限定の容量/保有理由/整理条件は `.codex-workspace/verification/shared/mobile-ios-personal-device-20261026/iphone-installed-verification-v1.json` と接続receiptへ記録。現在の署名appはMac上のDerivedData内にのみ保有し、cacheの整理はapp/manifest/receipt保全と明示範囲の確認後に分けて行う。今回は削除0件。

원본 결과·서명 앱 inventory·자기 경로 한정 용량/보유 이유/정리 조건은 `.codex-workspace/verification/shared/mobile-ios-personal-device-20261026/iphone-installed-verification-v1.json`과 연결 receipt에 기록했다. 현재 서명 앱은 Mac DerivedData 안에만 보유하므로 앱/manifest/receipt 보존과 명시적 범위 확인 후 캐시 정리를 별도로 다룬다. 이번 삭제는0개다.


無料Personal Teamの実機インストールと手動起動まで完了し、現時点の必須ユーザー操作はない。Release appにJS bundleを内蔵しておりMetroやUSB接続を実行条件にしていない。署名profile有効期限は上表の実測値で、期限後に継続する際は再署名/再インストールが必要。証拠は同taskの`iphone-human-trust-and-open-confirmed-v1.json`と`completion.json`へ接続する。

무료 Personal Team 실기기 설치와 수동 기동까지 완료했으며 지금 필요한 추가 사용자 조치는 없다. Release 앱에 JS bundle을 내장하므로 Metro나 USB 연결을 실행 조건으로 두지 않는다. 서명 profile 유효기간은 위 표의 실측값이며 만료 후 계속 테스트하려면 재서명/재설치가 필요하다. 근거는 같은 task의 `iphone-human-trust-and-open-confirmed-v1.json`과 `completion.json`에 연결한다.
