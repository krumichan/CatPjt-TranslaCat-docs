<a id="데스크톱-공용-실행-운영"></a>

# 데스크톱 공용 실행 운영 / デスクトップ共通実行の運用

2026-10-23 사용자는 TranslaCat 웹·앱과 Unity·Unreal RPG·게임 서버 등의 실행, 빌드, 테스트를 데스크톱에 맡기고 각 작업 채팅에 공유하도록 지시했다. 채팅, 코딩과 코드 원본은 노트북에서 유지한다.

2026-10-23、ユーザーはTranslaCatのWeb・アプリ、Unity・UnrealのRPG、ゲームサーバーなどの起動・ビルド・テストをデスクトップに任せ、各作業チャットに共有するよう指示した。チャット、コーディング、コード原本はノートPCに保持する。

<a id="실행-위치와-도구"></a>

## 실행 위치와 도구 / 実行場所とツール

- 공용 도구: 현재 project-cat root의 `.codex-workspace/verification/shared/desktop-workloads-20261023/`.

  共通ツール：現在のproject-catルートの`.codex-workspace/verification/shared/desktop-workloads-20261023/`。
- 프로젝트 위치·선행 도구: 위 폴더의 `projects.json`. 이 파일은 프로젝트 목록과 준비 상태이며 자동 실행 프로필은 아니다.

  プロジェクトの場所・前提ツール：上記フォルダーの`projects.json`。このファイルはプロジェクト一覧と準備状況を示すもので、自動実行プロファイルではない。
- 데스크톱: `codexbuilder@192.168.3.27`, 이름 `DESKTOP-UJPDGVL`, 작업 root `C:/Users/codexbuilder/CodexBuild`.

  デスクトップ：`codexbuilder@192.168.3.27`、コンピューター名`DESKTOP-UJPDGVL`、作業ルート`C:/Users/codexbuilder/CodexBuild`。
- 기존 SSH 키·고정 서버 지문을 재사용한다. 전역 PATH/NVM이나 개인키를 바꾸지 않는다.

  既存のSSH鍵と固定したサーバーフィンガープリントを再利用する。グローバルPATH/NVMや秘密鍵は変更しない。
- 기존 FE 빌드·단위 테스트는 `desktop-build-20261020/Invoke-Build.ps1`을 사용한다. 이 실행기도 공용 잠금에 연결했다.

  既存FEのビルド・単体テストには`desktop-build-20261020/Invoke-Build.ps1`を使う。この実行器も共通ロックに接続している。
- 다른 프로젝트의 원격 실행 어댑터는 `Invoke-DesktopWork.ps1`로 실행한다. 어댑터는 담당 채팅이 실제 프로젝트의 기존 빌드·테스트 도구를 재사용해 작성한다.

  他プロジェクトのリモート実行アダプターは`Invoke-DesktopWork.ps1`で実行する。担当チャットが、実際のプロジェクトの既存ビルド・テストツールを再利用してアダプターを作成する。

<a id="공용-실행기"></a>

## 공용 실행기 / 共通実行器

```powershell
$desktopRunner = 'D:\프로젝트\project-cat\.codex-workspace\verification\shared\desktop-workloads-20261023\Invoke-DesktopWork.ps1'

# 읽기 전용 연결 확인과 무거운 작업 잠금 상태
powershell.exe -NoProfile -ExecutionPolicy RemoteSigned -File $desktopRunner -Check
powershell.exe -NoProfile -ExecutionPolicy RemoteSigned -File $desktopRunner -LockStatus

# 담당 채팅이 준비한 소스 사본과 원격 실행용 PowerShell 어댑터
powershell.exe -NoProfile -ExecutionPolicy RemoteSigned -File $desktopRunner `
  -Project unity -Task Test `
  -InputDirectory '<준비한 소스 사본 폴더>' -ScriptPath '<원격 실행 어댑터.ps1>'

# 기존 작업 결과만 회수하며 재실행하지 않는다.
powershell.exe -NoProfile -ExecutionPolicy RemoteSigned -File $desktopRunner -Collect '<JobId>'
```

コード例のコメント：読み取り専用の接続確認と重い作業のロック状態。担当チャットが準備したソースコピーとリモート実行用PowerShellアダプター。既存作業の結果だけを回収し、再実行はしない。

`InputDirectory`는 원본이 아니라 담당 채팅이 만든 실행용 사본을 지정한다. 상대 폴더 구조와 필요한 원본 파일을 보존한다. 실행기는 Git clone·commit·push를 수행하지 않으며, 사본을 SCP로 보내고 파일별 SHA-256·크기·목록을 검증한다. `.git`, 개인키, 실제 `.env`, 서명 파일, SQL 백업, 의존성·Unity Library 등은 사본에 넣지 않는다. 비밀 설정이 필요한 작업은 데스크톱에 별도로 준비된 개발용 설정 경로를 사용하며 비밀 본문을 로그나 메시지로 전달하지 않는다.

`InputDirectory`には原本ではなく、担当チャットが作成した実行用コピーを指定する。相対フォルダー構造と必要な原本ファイルを保持する。実行器はGit clone・commit・pushを行わず、コピーをSCPで送信し、各ファイルのSHA-256・サイズ・一覧を検証する。`.git`、秘密鍵、実際の`.env`、署名ファイル、SQLバックアップ、依存関係・Unity Libraryなどはコピーに含めない。秘密設定が必要な作業では、デスクトップに別途準備した開発用設定のパスを使い、秘密情報の本文をログやメッセージに出さない。

소스 전송은 작업별 새 입력 폴더를 사용한다. 현재 공용 전송 자체에는 증분 동기화·캐시 재사용·원본 변경 회수가 포함되지 않는다. 큰 프로젝트의 캐시 재사용과 개발 서버용 지속 작업 폴더는 프로젝트별 어댑터에서 따로 관리한다. 삭제·덮어쓰기는 소유 경로를 검증하고 보존 규칙에 따라 처리한다.

ソース送信には作業ごとに新しい入力フォルダーを使う。現在の共通転送機能自体には、差分同期・キャッシュ再利用・原本変更の回収は含まれない。大規模プロジェクトのキャッシュ再利用と開発サーバー用の継続作業フォルダーは、プロジェクト別アダプターで別途管理する。削除・上書きは所有パスを検証し、保全ルールに従って扱う。

원격 어댑터는 Windows PowerShell 5.1 호환 스크립트다. PowerShell 7이 필요한 프로젝트는 어댑터에서 설치된 `pwsh.exe`를 명시적으로 호출한다. 다음 프로세스 환경변수를 사용한다.

リモートアダプターはWindows PowerShell 5.1互換スクリプトとする。PowerShell 7が必要なプロジェクトでは、アダプターからインストール済みの`pwsh.exe`を明示的に呼び出す。次のプロセス環境変数を使う。

| 변수 / 変数 | 의미 / 意味 |
| --- | --- |
| `CODEX_DESKTOP_JOB_ID` | 고유 작업 ID<br>一意の作業ID |
| `CODEX_DESKTOP_JOB_ROOT` | 원격 작업 폴더<br>リモート作業フォルダー |
| `CODEX_DESKTOP_INPUT_DIR` | 검증된 사본 경로. 사본 없는 진단 작업은 빈 값<br>検証済みコピーのパス。コピーのない診断作業では空の値 |
| `CODEX_DESKTOP_RESULTS_DIR` | 노트북으로 회수할 로그·보고서·산출물 저장 경로<br>ノートPCに回収するログ・レポート・成果物の保存パス |
| `CODEX_DESKTOP_TOOLS_DIR` | 데스크톱 공용 도구 경로<br>デスクトップ共通ツールのパス |
| `CODEX_DESKTOP_RUNTIME_CATALOG` | 준비한 SDK 실행 파일·버전·검증 기록의 `tools/runtimes.json` 경로<br>準備済みSDKの実行ファイル・バージョン・検証記録を記載する`tools/runtimes.json`のパス |
| `CODEX_DESKTOP_PWSH` | 준비된 경우 격리 PowerShell 7 실행 파일 경로<br>準備済みの場合、分離したPowerShell 7実行ファイルのパス |

Node 22.22.2 경로는 작업 프로세스의 PATH 앞에 추가한다. 런타임 catalog가 준비되면 `be`·`ll` 프로젝트는 격리 JDK의 JAVA_HOME/PATH를, `chat`·`game-server`·`unity`·`unreal`은 격리 .NET의 DOTNET_ROOT/PATH를 이 프로세스에만 적용한다. 필수 catalog가 없으면 실행 전에 중단한다. Python·Android SDK·엔진 및 다른 이름의 프로젝트는 담당 어댑터에서 정확한 경로와 버전을 확인한다. 시스템·사용자의 전역 PATH를 바꾸지 않는다.

Node 22.22.2のパスを作業プロセスのPATH先頭に追加する。ランタイムcatalogの準備後、`be`・`ll`プロジェクトには分離したJDKのJAVA_HOME/PATHを、`chat`・`game-server`・`unity`・`unreal`には分離した.NETのDOTNET_ROOT/PATHを、このプロセスだけに適用する。必須catalogがなければ実行前に停止する。Python・Android SDK・エンジン、および他の名前のプロジェクトについては、担当アダプターで正確なパスとバージョンを確認する。システム・ユーザーのグローバルPATHは変更しない。

어댑터는 모든 무거운 자식 작업의 완료까지 동기 대기하고, 실패 시 실제 종료 코드를 반환해야 한다. `Start-Process`만 호출한 뒤 즉시 종료하면 잠금도 풀리므로 무거운 백그라운드 작업의 우회 실행에 사용하지 않는다. 지속 서비스의 Start는 상태·종료 관리를 별도로 구현한다. 결과의 JobId·ScriptSha256·ExitCode와 단계별 보고서를 확인한다. `InputFiles`는 전송 사본의 검증 수이며 원본 프로젝트 전체나 기능 전체를 검증했다는 뜻은 아니다. 다른 채팅이 원본을 수정할 수 있으므로 원본 해시와 사본 해시의 일치 여부는 담당 채팅이 따로 확인한다.

アダプターは、すべての重い子作業が完了するまで同期的に待機し、失敗時には実際の終了コードを返さなければならない。`Start-Process`だけを呼んで直ちに終了するとロックも解除されるため、重いバックグラウンド作業を迂回実行する用途には使わない。常駐サービスのStartには、状態・終了管理を別途実装する。結果のJobId・ScriptSha256・ExitCodeと段階別レポートを確認する。`InputFiles`は転送コピーの検証数であり、原本プロジェクト全体や機能全体を検証した意味ではない。他チャットが原本を変更する可能性があるため、原本とコピーのハッシュ一致は担当チャットが別途確認する。

<a id="동시-실행과-결과"></a>

## 동시 실행과 결과 / 同時実行と結果

2026-10-23 후속 사용자 지시는 **실측 자원에 여유가 있으면 빌드 전용 작업은 제한적으로 병행하고, 게임 플레이·화면 조작 테스트는 서로 겹치지 않게 한다**는 것이다. 특정 채팅 전체가 끝나기를 기다리는 규칙은 아니다. CPU/RAM/GPU/디스크 표본의 시각과 작업 ID를 기록하고, 시작 직전에 다시 확인한다. 측정할 수 없는 디스크 IO를 여유롭다고 추정하지 않는다.

2026-10-23のユーザーの追加指示は、**実測でリソースに余裕があればビルド専用作業を限定的に並行し、ゲームプレイ・画面操作のテストは互いに重ねない**というものだ。特定チャット全体の完了を待つルールではない。CPU/RAM/GPU/ディスクの標本時刻と作業IDを記録し、開始直前に再確認する。測定できないディスクIOに余裕があるとは推測しない。

**현재 적용 상태:** 공용 실행기는 여전히 `CodexBuild/locks`의 동일한 배타적 파일 잠금을 사용한다. 제한 병행 슬롯과 CPU/RAM 상한을 활성화하는 수정은 자동 승인 검토에서 거부되어 배포하지 않았다. 따라서 아직 `-Mode Build` 같은 새 옵션을 사용할 수 없으며, 운영 원칙 변경을 잠금 우회 허가로 해석하지 않는다. 검토할 구체 변경 범위는 [제한 병행 실행 제안](desktop-concurrency-proposal.md)에 있다. 현재 공용 잠금이 Free이면 기존 실행기로 빌드를 진행할 수 있다.

**現在の適用状況：** 共通実行器は引き続き`CodexBuild/locks`の同じ排他的ファイルロックを使う。限定並行スロットとCPU/RAM上限を有効にする変更は、自動承認審査で拒否されたため配備していない。このため、まだ`-Mode Build`のような新オプションは使えず、運用原則の変更をロック迂回の許可と解釈しない。審査対象の具体的な変更範囲は[限定並行実行の提案](desktop-concurrency-proposal.md)にある。現在の共通ロックがFreeなら、既存実行器でビルドを進められる。

기존 잠금이 사용 중이면 해당 작업 ID와 실제 실행을 확인한다. 자동 대기열이나 자동 재실행은 없다. 잠금 보유 worker가 자식 작업 완료를 기다리며, SSH가 끊겼다고 임의로 잠금을 제거하거나 같은 작업을 다시 시작하지 않는다.

既存ロックが使用中なら、該当作業IDと実際の実行を確認する。自動待ち行列や自動再実行はない。ロックを保持するworkerが子作業の完了を待ち、SSHが切れたからといって任意にロックを削除したり、同じ作業を再開したりしない。

이 잠금은 공용 실행기를 거치는 작업끼리 적용된다. 별도로 열린 에디터·게임·서버를 자동 종료하지 않는다. 기존 에디터나 지속 서버가 자원을 사용하고 있으면 담당 채팅이 확인해 조정한다. 잠금 보유 부모가 강제 종료되고 자식만 살아남는 경우도 실제 프로세스를 확인한 뒤 다음 작업을 선택한다.

このロックは、共通実行器を経由する作業同士に適用される。別途開いたエディター・ゲーム・サーバーを自動終了しない。既存エディターや常駐サーバーがリソースを使っている場合、担当チャットが確認して調整する。ロック保持親プロセスが強制終了され、子だけが残った場合も、実際のプロセスを確認したうえで次の作業を選ぶ。

로컬 결과는 공용 도구의 `runs/<JobId>/results/`에 저장한다. 이미 회수한 결과가 있으면 `retrieved-<시각>/results/`에 별도 보존한다. 어댑터가 `CODEX_DESKTOP_RESULTS_DIR`에 저장한 APK·보고서·스크린샷 등도 함께 회수한다. 원본 프로젝트에 자동으로 덮어쓰지 않는다.

ローカルの結果は、共通ツールの`runs/<JobId>/results/`に保存する。回収済み結果がある場合は、`retrieved-<시각>/results/`に別途保全する。アダプターが`CODEX_DESKTOP_RESULTS_DIR`に保存したAPK・レポート・スクリーンショットなども併せて回収する。原本プロジェクトへ自動で上書きしない。

연결 실패만으로 노트북 빌드로 전환하지 않는다. 사용자가 외부·데스크톱 사용 불가·로컬 실행을 명시한 경우에 기존 로컬 절차를 사용한다. 실패한 부분과 필요한 직접 작업은 최종 응답에 명시하고 종료한다. 휴대폰 사용자에게 질문 카드나 commentary만으로 설치·로그인·승인 요청을 남기지 않는다.

接続失敗だけを理由にノートPCでのビルドへ切り替えない。ユーザーが外出先・デスクトップ利用不可・ローカル実行を明示した場合、既存のローカル手順を使う。失敗した部分と必要な直接操作は最終応答に記載して終了する。スマートフォン利用者に、質問カードやcommentaryだけでインストール・ログイン・承認の依頼を残さない。

<a id="프로젝트별-경계"></a>

## 프로젝트별 경계 / プロジェクト別の境界

- FE: 기존 빌드·단위 테스트 경로가 검증되어 있다. 기동·E2E는 별도 어댑터와 개발용 설정이 필요하다.

  FE：既存のビルド・単体テスト経路は検証済み。起動・E2Eには別のアダプターと開発用設定が必要。
- BE·LL·CHAT·AI: 정확한 런타임과 격리된 테스트 설정이 필요하다. 운영 DB·실사용자 데이터에 연결해 검증하거나 migration을 자동 실행하지 않는다. AI의 실제 Dockerfile은 Python3.11/Linux이며 PaddleOCR/OpenCV용 OS 라이브러리도 설치한다. 임의 Windows Python 설치만으로 전체 API 환경이 재현됐다고 판단하지 않는다.

  BE・LL・CHAT・AI：正確なランタイムと分離したテスト設定が必要。本番DB・実ユーザーデータに接続して検証したり、migrationを自動実行したりしない。AIの実際のDockerfileはPython3.11/Linuxで、PaddleOCR/OpenCV用のOSライブラリもインストールする。任意のWindows Pythonを入れただけで、API環境全体を再現できたと判断しない。
- Mobile: JS/TS 검증, Metro, Android 빌드·기기 테스트를 구분한다. 노트북의 `build-local-android.mjs`는 개인 설정·경로에 의존하므로 그대로 전송해 실행하지 않는다. 개발용 SDK·서명 설정을 확인한다. iOS 네이티브 빌드는 Mac 또는 기존 클라우드 빌드 경로를 사용한다.

  Mobile：JS/TS検証、Metro、Androidビルド・実機テストを区別する。ノートPCの`build-local-android.mjs`は個人設定・パスに依存するため、そのまま転送して実行しない。開発用SDK・署名設定を確認する。iOSネイティブビルドにはMacまたは既存のクラウドビルド経路を使う。
- Unity: 현재 원본은 `D:/프로젝트/GamePjt-BlockCoopRPG-Unity`, 엔진은 `6000.3.26f1`이다. `Client/Unity`와 상대 참조되는 `Shared`, `Server`, `Tools` 구조 및 `.meta`를 보존한다. `Tools/Test-ArchitectureRules.ps1`은 순수 규칙 검증의 후보이며, 실행 시점의 파일·고정 SHA·옵션을 확인한다. 기존 사용자별 캐시 링크를 다른 실행과 충돌시키지 않는다.

  Unity：現在の原本は`D:/프로젝트/GamePjt-BlockCoopRPG-Unity`、エンジンは`6000.3.26f1`。`Client/Unity`と相対参照される`Shared`、`Server`、`Tools`の構造、および`.meta`を保持する。`Tools/Test-ArchitectureRules.ps1`は純粋なルール検証の候補であり、実行時点のファイル・固定SHA・オプションを確認する。既存のユーザー別キャッシュリンクを他の実行と競合させない。
- .NET 게임 서버: 현재 구조 이전이 진행 중이다. 현재 서버 프로젝트와 SDK pin을 다시 읽고 테스트·빌드한다. 기존 서버·MySQL·계정 저장 API와 저장 데이터를 임의로 중단하거나 복사하지 않는다.

  .NETゲームサーバー：現在、構造の移行が進行中。現在のサーバープロジェクトとSDK pinを読み直してテスト・ビルドする。既存サーバー・MySQL・アカウント保存APIと保存データを任意に停止・コピーしない。
- Unreal: 원본 `D:/GameProjects/Fablewood`와 보존본 `D:/프로젝트/GamePjt-FableCoopRPG-Unreal`를 유지한다. 이번 OnlineFlow 개발 사본은 `D:/GameProjects/Fablewood-OnlineFlow-20261023`의 Game/Backend 두 디렉터리다. 작업별 `Tools/Get-SourceManifest.ps1` 허용 목록과 SHA로 선택한 사본만 전송하며, `Build/`와 `ArtSource/Meshes/*.obj` 원본을 캐시로 제외하지 않는다. 데스크톱 UE5.8.4·MSVC14.50·Windows SDK26100·NETFXSDK4.8 설치 범위는 아래 기록을 따른다. 실행 직전 충분한 디스크와 해당 작업의 런타임 준비 상태를 별도로 확인한다.

  Unreal：原本`D:/GameProjects/Fablewood`と保全コピー`D:/프로젝트/GamePjt-FableCoopRPG-Unreal`を維持する。今回のOnlineFlow開発コピーは`D:/GameProjects/Fablewood-OnlineFlow-20261023`内のGame/Backendの両ディレクトリー。作業ごとの`Tools/Get-SourceManifest.ps1`の許可リストとSHAで選んだコピーだけを転送し、`Build/`と`ArtSource/Meshes/*.obj`の原本をキャッシュとして除外しない。デスクトップのUE5.8.4・MSVC14.50・Windows SDK26100・NETFXSDK4.8のインストール範囲は下記記録に従う。実行直前に、十分なディスク容量と該当作業のランタイム準備状況を別途確認する。
- 에셋·GPU 작업: RTX3070Ti 8GB의 실제 메모리 요구량을 확인한다. 현재 모델 생성 작업이 웹 서비스에서 실행된다면 로컬 데스크톱 GPU로 자동 이전된 것으로 보고하지 않는다.

  アセット・GPU作業：RTX3070Ti 8GBに対する実際のメモリ要件を確認する。現在のモデル生成作業がWebサービスで実行される場合、ローカルデスクトップGPUへ自動移行したと報告しない。

게임의 기존 Start/LAN 런처에는 DB·API 확보, 방화벽 승인, 입력 대기, 종료 시 DB 백업이 포함될 수 있다. 단순 SSH Start로 감싸지 않는다. 별도 개발용 저장소·계정·포트, 시작/상태/종료와 로그 회수를 구현한 뒤 기동한다. 화면 조작은 별도 원격 GUI 연결을 준비한다. Scene·Blueprint·에셋 저장 변경은 노트북 원본으로 명시적으로 회수하고 충돌을 확인한다.

ゲームの既存Start/LANランチャーには、DB・APIの確保、ファイアウォール承認、入力待ち、終了時のDBバックアップが含まれる場合がある。単純なSSH Startで包まない。別の開発用保存先・アカウント・ポートと、開始/状態/終了・ログ回収を実装してから起動する。画面操作には別途リモートGUI接続を準備する。Scene・Blueprint・アセットの保存変更はノートPCの原本へ明示的に回収し、競合を確認する。

<a id="준비-상태와-작업-채팅"></a>

## 준비 상태와 작업 채팅 / 準備状況と作業チャット

초기 조사에서 데스크톱은 Ryzen5 5600X, RAM16GB, RTX3070Ti 8GB, C: 여유 약54GiB다. Node22.22.2와 기존 Unity2022.3.62f1/6000.2.6f1, .NET8, VS2022는 확인했다. 이후 공식 archive의 checksum을 검증해 공용 tools 아래에 다음 런타임을 격리 준비했다. 기존 시스템 도구와 전역 PATH는 유지했다.

初期調査時のデスクトップはRyzen5 5600X、RAM16GB、RTX3070Ti 8GB、C:空き約54GiB。Node22.22.2、既存のUnity2022.3.62f1/6000.2.6f1、.NET8、VS2022を確認した。その後、公式archiveのchecksumを検証し、共通tools配下に次のランタイムを分離して準備した。既存のシステムツールとグローバルPATHは維持した。

| 런타임 / ランタイム | 버전 / バージョン | 데스크톱 실행 파일 / デスクトップの実行ファイル |
| --- | --- | --- |
| .NET SDK | 10.0.401 | `C:/Users/codexbuilder/CodexBuild/tools/dotnet-sdk-10.0.401/dotnet.exe` |
| JDK | 21.0.12.1 | `C:/Users/codexbuilder/CodexBuild/tools/jdk-21.0.12.1-1/bin/java.exe` |
| PowerShell | 7.6.8 | `C:/Users/codexbuilder/CodexBuild/tools/powershell-7.6.8/pwsh.exe` |

버전·URL·해시는 도구 폴더 및 원격 tools의 `runtimes.json`에 보존했다. 초기 SDK 준비 당시 C: 여유 약51GiB는 현재 잔여 공간을 뜻하지 않는다. Unreal의 UE5.8.4와 C++ 도구는 아래 설치 기록으로 갱신한다. Unity6000.3.26f1·AndroidSDK·Python3.11/Linux 환경은 별도 준비가 필요하다. 실제 엔진·SDK 준비 상태와 프로젝트별 어댑터·설정은 작업 결과 및 `projects.json`을 확인한다. 다른 버전 설치만으로 준비 완료라고 판단하지 않는다.

バージョン・URL・ハッシュはツールフォルダーとリモートtoolsの`runtimes.json`に保全した。初期SDK準備時のC:空き約51GiBは、現在の残容量を示すものではない。UnrealのUE5.8.4とC++ツールについては、下記インストール記録で更新する。Unity6000.3.26f1・AndroidSDK・Python3.11/Linux環境は別途準備が必要。実際のエンジン・SDK準備状況とプロジェクト別アダプター・設定は、作業結果と`projects.json`を確認する。別バージョンのインストールだけで準備完了と判断しない。

<a id="2026-10-23-unreal-환경-설치-완료"></a>

### 2026-10-23 Unreal 환경 설치 완료 / 2026-10-23 Unreal環境のインストール完了

데스크톱 `DESKTOP-UJPDGVL`의 `codexbuilder` 계정에 아래 도구를 준비했다. 표의 `<tools>`는 `C:/Users/codexbuilder/CodexBuild/tools`다. 설치·파일 확인 근거는 `.codex-workspace/verification/shared/fablewood-desktop-setup-20261023/desktop-environment-installed-20261023T104223700614Z-0b9ce5aa.json`이며, 게임의 최종 검증 결과와 구분한다.

デスクトップ`DESKTOP-UJPDGVL`の`codexbuilder`アカウントに、下記ツールを準備した。表の`<tools>`は`C:/Users/codexbuilder/CodexBuild/tools`を指す。インストール・ファイル確認の根拠は`.codex-workspace/verification/shared/fablewood-desktop-setup-20261023/desktop-environment-installed-20261023T104223700614Z-0b9ce5aa.json`であり、ゲームの最終検証結果とは区別する。

| 도구 / ツール | 확인 버전 / 確認バージョン | 데스크톱 경로·설치 범위 / デスクトップのパス・インストール範囲 |
| --- | --- | --- |
| Unreal Engine | 5.8.4, CL 58902052 | `<tools>/fablewood-engine-5.8.4/UE_5.8` — Installed Build |
| MSVC | 14.50.35717 | `<tools>/vs2026-buildtools/VC/Tools/MSVC/14.50.35717` |
| Windows SDK | 10.0.26100.0 | `C:/Program Files (x86)/Windows Kits/10` |
| NETFXSDK | 4.8 | `<tools>/netfxsdk-4.8` — Include/Lib 61개와 HKCU 검색 경로 등록. Framework 런타임·targeting pack 설치를 뜻하지 않는다.<br>`<tools>/netfxsdk-4.8` — Include/Lib 61個とHKCU検索パスの登録。Frameworkランタイム・targeting packのインストールを意味しない。 |
| PowerShell | 7.6.8 | `<tools>/powershell-7.6.8/pwsh.exe` |
| .NET SDK | 10.0.401 | `<tools>/dotnet-sdk-10.0.401/dotnet.exe` |
| Docker CLI/Engine | 29.9.2, Linux | `C:/Users/codexbuilder/AppData/Local/Programs/DockerDesktop/resources/bin/docker.exe` — 해당 작업에서 Linux 엔진 준비 상태 확인<br>`C:/Users/codexbuilder/AppData/Local/Programs/DockerDesktop/resources/bin/docker.exe` — 該当作業でLinuxエンジンの準備状況を確認 |

`CODEX_FABLE_ENGINE_ROOT`와 도구 PATH는 작업 프로세스에만 지정한다. Docker는 다음 작업에서도 살아 있다고 가정하지 않고 같은 작업 안에서 Linux 엔진 상태를 확인한다. 기존 사용자 Docker 설정·자격 증명은 변경하지 않는다.

`CODEX_FABLE_ENGINE_ROOT`とツールのPATHは作業プロセスだけに指定する。Dockerが次の作業でも起動しているとは仮定せず、同じ作業内でLinuxエンジンの状態を確認する。既存ユーザーのDocker設定・資格情報は変更しない。

현재 전용 서버 경로는 `UnrealEditor-Cmd.exe -server -NullRHI`이고, 온라인 UI 검증은 Editor 서버 1개와 Editor 클라이언트 2개를 사용한다. 이 Installed Build는 독립 Server Target을 지원하지 않으므로 `FablewoodServer.exe` 생성에는 Server Target을 지원하는 소스 엔진 구성이 별도로 필요하다. Windows Development Game 패키징과 해당 패키지의 실제 플레이는 별도 검증이며, 일반 Game 실행 파일의 `-server` 실행을 전용 서버 검증으로 간주하지 않는다.

現在の専用サーバー経路は`UnrealEditor-Cmd.exe -server -NullRHI`で、オンラインUI検証にはEditorサーバー1個とEditorクライアント2個を使う。このInstalled Buildは独立したServer Targetをサポートしないため、`FablewoodServer.exe`の生成にはServer Target対応のソースエンジン構成が別途必要。Windows Development Gameのパッケージングと、そのパッケージの実プレイは別の検証であり、通常のGame実行ファイルを`-server`で起動しても専用サーバーの検証とはみなさない。

Session 0의 D3D11 화면 생성은 실제로 `DXGI_ERROR_NOT_CURRENTLY_AVAILABLE`로 실패했다. 렌더링 UI 검증은 현재 사용자와 SID가 같은 활성 대화형 세션이 정확히 하나인지 확인하는 InteractiveToken 브리지를 사용한다. 세션 번호를 고정하지 않으며, 권한 상승·다른 계정의 자격 증명 없이 실행하고 모든 자식 종료까지 공용 잠금을 유지한다. `-NullRHI` 검사 성공만으로 화면 검증 성공을 판단하지 않는다.

Session 0でのD3D11画面生成は、実際に`DXGI_ERROR_NOT_CURRENTLY_AVAILABLE`で失敗した。レンダリングUI検証には、現在のユーザーと同じSIDを持つアクティブな対話セッションが正確に一つあることを確認するInteractiveTokenブリッジを使う。セッション番号は固定せず、権限昇格・他アカウントの資格情報を使わずに実行し、すべての子の終了まで共通ロックを維持する。`-NullRHI`検査の成功だけで画面検証成功と判断しない。

현재 Unreal 검증은 일부 단계가 완료된 상태다. `20261023T115009777Z-aee36c0b` 작업에서 현재 선택 소스 324개의 보존을 확인하고 Editor C++ 빌드·맵 준비, 네이티브 22개 검사(실패·경고 0), Windows Development Game 패키징이 모두 통과했다. 패키지 `20261023-115556-406Z-624981c5`는 데스크톱의 해당 작업 `src/Game/Builds/Windows-20261023-115556-406Z-624981c5`에 보존되어 있다. 이 패키지는 아직 실행 검증하지 않았으며 `playTested=false`다. 근거는 공용 `runs/20261023T115009777Z-aee36c0b/results/`의 `onlineflow-build-package-validation.json`, `native/index.json`, `packaging/20261023-115556-406Z-624981c5/result.json`이다.

現在のUnreal検証は、一部段階が完了した状態。`20261023T115009777Z-aee36c0b`作業で、現在選択したソース324個の保全を確認し、Editor C++ビルド・マップ準備、ネイティブ検査22件（失敗・警告0）、Windows Development Gameパッケージングがすべて通過した。パッケージ`20261023-115556-406Z-624981c5`は、デスクトップの該当作業内の`src/Game/Builds/Windows-20261023-115556-406Z-624981c5`に保全している。このパッケージの実行検証はまだ行っておらず、`playTested=false`。根拠は共通`runs/20261023T115009777Z-aee36c0b/results/`の`onlineflow-build-package-validation.json`、`native/index.json`、`packaging/20261023-115556-406Z-624981c5/result.json`。

이후 `20261023T121549120Z-81140b02` 작업은 공식 Docker 재시작 후 활성 대화형 세션에서 Editor 전용 서버 1개와 클라이언트 2개의 등록 단계 UI 검증을 통과했다. 서로 다른 두 계정, 이동·물약 사용, 클라이언트 1의 정상 OS 종료(0), 클라이언트 2의 의도한 강제 종료(-1), 서버 정상 종료(0), 저장 재화·아이템·장착·캐릭터 요약의 일치 및 리스 해제를 확인했다. 하지만 `owned-api-restart` 단계의 일반 예외로 전체 작업은 실패했으며, API 재시작 이후 로그인·복원 검증은 완료되지 않았다. 등록 단계 성공을 전체 온라인 흐름 성공으로 간주하지 않는다. 실제 보고서는 `runs/20261023T121549120Z-81140b02/results/online/20261023-211641-a68ff98f/summary.json`이다.

続く`20261023T121549120Z-81140b02`作業では、公式のDocker再起動後、アクティブな対話セッションでEditor専用サーバー1個とクライアント2個の登録段階UI検証に通過した。異なる二つのアカウント、移動・ポーション使用、クライアント1の正常なOS終了（0）、クライアント2の意図した強制終了（-1）、サーバーの正常終了（0）、保存した通貨・アイテム・装備・キャラクター要約の一致とリース解放を確認した。ただし、`owned-api-restart`段階の一般例外により作業全体は失敗し、API再起動後のログイン・復元検証は完了していない。登録段階の成功をオンラインフロー全体の成功とみなさない。実際のレポートは`runs/20261023T121549120Z-81140b02/results/online/20261023-211641-a68ff98f/summary.json`。

후속 진단 `20261023T123531393Z-37843a76`은 Docker 엔진 미준비로 사전 점검에서 실패했으며 서비스 조작을 시도하지 않았다. 최신 읽기 전용 프로세스 관측에서도 Docker 프로세스가 없었으므로 현재 온라인 검증 실행 준비가 완료됐다고 표시하지 않는다. 종료 원인은 확인되지 않았고, 앞선 API 재시작 예외와 같은 원인이라고 단정하지 않는다. 해당 진단의 소스 324개와 기존 자격 증명·키는 보존됐다. 근거는 `runs/20261023T123531393Z-37843a76/results/owned-backend-restart-diagnostic.json`이다. 이전 실패·성공 기록과 설치 도구는 유지하며, 다음 작업에서도 실제 Linux Docker 준비 상태를 다시 확인한다.

後続診断`20261023T123531393Z-37843a76`は、Dockerエンジン未準備のため事前点検で失敗し、サービス操作は試みていない。最新の読み取り専用プロセス観測でもDockerプロセスがなかったため、現在のオンライン検証実行の準備を完了と表示しない。終了原因は未確認で、先のAPI再起動例外と同じ原因とは断定しない。該当診断のソース324個と既存の資格情報・鍵は保全した。根拠は`runs/20261023T123531393Z-37843a76/results/owned-backend-restart-diagnostic.json`。以前の失敗・成功記録とインストール済みツールを維持し、次の作業でも実際のLinux Docker準備状況を再確認する。

각 채팅에는 이 공용 문서와 실행기를 안내한다. 현재 진행 중인 기능 개발을 유지하면서 다음 기동·빌드·테스트부터 데스크톱 경로를 사용한다. 완료된 과거 작업을 재개하거나 새 기능·배포를 시작하라는 지시가 아니다. 준비되지 않은 도구·로그인·실기기 연결·저장공간 문제는 구체적인 직접 작업과 링크를 최종 응답으로 안내한다.

各チャットには、この共通文書と実行器を案内する。現在進行中の機能開発を維持し、次の起動・ビルド・テストからデスクトップ経路を使う。完了済みの過去作業の再開や、新機能・配備の開始を指示するものではない。未準備のツール・ログイン・実機接続・ストレージ容量の問題は、具体的な直接操作とリンクを最終応答で案内する。

Git·커밋·푸시·배포·비밀 설정·DB 데이터 변경의 기존 승인 조건을 유지한다.

Git・コミット・push・配備・秘密設定・DBデータ変更について、既存の承認条件を維持する。

<a id="2026-10-23-검증-기록"></a>

## 2026-10-23 검증 기록 / 2026-10-23 検証記録

- 공용 전송 smoke 작업 `20261023T031346627Z-a3c8b918`에서 준비한 입력 2개를 검증하고, 데스크톱 Node 실행과 산출물 회수를 확인했다.

  共通転送のsmoke作業`20261023T031346627Z-a3c8b918`で、準備した入力2個を検証し、デスクトップでのNode実行と成果物回収を確認した。
- 공용 원격 실행기 QA 6/6, 잠금 QA 7/7, 소스 사본 필터 QA 16/16이 통과했다. 실패 종료 코드·결과 재회수·동시 작업 차단도 검증 범위에 포함한다.

  共通リモート実行器QA 6/6、ロックQA 7/7、ソースコピーフィルターQA 16/16が通過した。失敗終了コード・結果の再回収・同時作業の遮断も検証範囲に含む。
- 격리 런타임 설치 작업 `20261023T033557167Z-f6fa708f`가 성공했다. 공식 archive 해시·실행 버전과 읽기 전용 프로세스 PATH 확인을 수행했다. 최종 원격 worker와 로컬 원본의 SHA-256이 일치한다.

  分離ランタイムのインストール作業`20261023T033557167Z-f6fa708f`が成功した。公式archiveのハッシュ・実行バージョンと、読み取り専用のプロセスPATH確認を行った。最終リモートworkerとローカル原本のSHA-256が一致する。
- SDK 준비 후 전체 wrapper probe 재실행은 다른 실제 RPG 작업이 공용 슬롯을 사용 중이어서 보류했다. 위 검증이 각 앱·게임의 전체 빌드·기동 성공을 뜻하지 않는다. FE의 이전 단위 테스트·빌드 성공도 현재 변경 중인 소스의 재검증으로 간주하지 않는다.

  SDK準備後のwrapper probe全体の再実行は、他の実際のRPG作業が共通スロットを使用中だったため保留した。上記検証は、各アプリ・ゲームの全体ビルド・起動の成功を意味しない。FEの過去の単体テスト・ビルド成功も、現在変更中のソースの再検証とはみなさない。
- 관련 채팅 16개에 정책과 실행기를 안내했고, 그중 런타임 관련 채팅 7개에는 준비된 SDK 경로를 추가 전달했다. 메시지 수락 기록은 공용 도구의 `notifications.json`에 보존했다.

  関連チャット16件に方針と実行器を案内し、そのうちランタイム関連の7件には準備済みSDKのパスを追加送信した。メッセージの受理記録は共通ツールの`notifications.json`に保全した。

세부 증거는 공용 도구의 `tests/transport-20261023032733-c393f44e/verification.json`, `tests/lock-results-20261023030920-688292a0.json`, `tests/runner-review-fixtures-7a5007d5ecc744319d591d4c94fdfc3c/review-results.json`, `tests/runtime-validation-20261023.json`에 보존했다.

詳細な根拠は共通ツールの`tests/transport-20261023032733-c393f44e/verification.json`、`tests/lock-results-20261023030920-688292a0.json`、`tests/runner-review-fixtures-7a5007d5ecc744319d591d4c94fdfc3c/review-results.json`、`tests/runtime-validation-20261023.json`に保全した。
