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

초기 정적 감사에서는 즉시 수정이 필요한 iOS 소스 결함을 확정하지 않았다. iOS 개발용 식별자·scheme, 사진 설명·SecureStore와 브라우저 인증 경로는 기존 설정에 존재한다. 실제 iOS native 컴파일·기동·로그인 복귀·사진 선택·키보드 동작은 별도 실행 근거가 필요하다. 가입 UI의 사용자 APK 실기기 테스트 생략은 성공 판정으로 바꾸지 않는다.

初回の静的監査では、直ちに修正が必要なiOSソース欠陥は確定していない。開発用識別子・scheme・写真説明・SecureStore・ブラウザー認証経路は既存設定にある。nativeコンパイル・起動・認証復帰・写真選択・キーボード動作は別途実行証拠が必要。登録UIのユーザー実機テスト省略を成功判定へ置き換えない。

작업별 소스 고정·공식 요건 조사·실제 결과는 현재 workspace의 `.codex-workspace/verification/shared/mobile-ios-free-preparation-20261024/`에 보존한다. 이 문서에 준비 절차가 있다는 사실만으로 공용 실행기·시뮬레이터 빌드가 완료된 것은 아니다.

ソース固定・公式要件調査・実結果はworkspaceの`.codex-workspace/verification/shared/mobile-ios-free-preparation-20261024/`へ保全する。本書の手順だけで共通実行器やSimulatorビルドが完了したとは扱わない。
