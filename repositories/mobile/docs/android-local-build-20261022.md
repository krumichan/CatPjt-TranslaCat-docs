# Windows Android 로컬 빌드 환경

사용자는 2026-10-22 로컬 Android 빌드 환경 구축과 APK 생성을 요청하고, Google SDK 라이선스 동의·설치 및 기존 Expo 개발용 서명키의 로컬 사용을 전부 승인했다. 모바일 원격 접속 중이므로 PC 화면 조작은 Codex가 수행한다. 유료 서비스 등록은 포함하지 않는다.

## 환경과 경계

현재 외장 workspace를 기준으로 작업한다. 기존 C: 백업은 수정하지 않는다. Windows native Gradle을 사용하며 EAS 클라우드 빌드·서버 배포·Expo 계정 변경을 수행하지 않는다. EAS local build의 Windows/WSL 지원과 구분한다.

| 항목 | 준비 값 |
| --- | --- |
| Node | 기존 NVM Node 22.22.2 |
| Java | 기존 Corretto 21.0.3 |
| Expo / React Native | 기존 lockfile의 58.0.14 / 0.88.0 |
| Gradle | 설치된 Expo 템플릿의 9.4.1 |
| Android SDK | `%LOCALAPPDATA%/Android/Sdk` |
| SDK / Build Tools | 공식 `platforms;android-37.0` / `37.0.0` |
| NDK / CMake | `27.1.12297006` / `3.31.6` |
| 출력 | ARM64 release APK, JS bundle 포함 |
| 앱 ID / scheme | `com.krumichan.translacat.dev` / `translacat-dev` |
| API | `https://217.142.230.186/api/v1` |

공식 Command-line tools ZIP `commandlinetools-win-15859902_latest.zip`을 Google 페이지의 SHA-256 `90ae805d20434428bffcb699c290860f19bb5f66a67e6b330067e3de801fb04a`과 대조했다. 사용자의 명시적 동의 뒤 라이선스를 수락하고 필요한 SDK 패키지를 설치했다. `platforms;android-37` 이름은 공식 목록에 없어서 첫 설치가 중단됐고, 목록에서 확인한 `platforms;android-37.0`으로 설치를 완료했다. 실패 로그도 보존한다.

현재 D:는 exFAT이므로 디렉터리 junction을 사용할 수 없다. 별도 빌드 사본으로 의존성 전체를 복사하는 첫 시도는 느린 외장 I/O로 중단했다. 이 작업의 robocopy 프로세스만 소유권을 확인해 중지했으며 부분 복사본·실패 로그도 보존했다. 기존 의존성을 그대로 사용하는 Expo 표준 `android/` native 출력 방식으로 전환한다. 소스 사본과 해시는 중앙 폴더에 보관한다. Android 도구의 한글 경로 문제는 현재 workspace에 연결한 ASCII 별칭 `U:`를 통해 `U:/CatPjt-TranslaCat-mobile`에서 빌드하여 처리한다. Expo가 드라이브 루트의 package.json을 찾지 못하므로 앱을 별칭 루트에 직접 놓지 않는다. 이전 `T:`는 중앙 검증 폴더의 APK 검사에만 사용한다. 다른 드라이브 연결을 덮어쓰지 않는다.

Windows autolinking이 실제 `D:` 경로와 `U:` 별칭을 섞으면 React Native codegen에서 서로 다른 drive root를 상대 경로로 변환할 수 없다는 오류가 발생했다. 생성된 settings.gradle의 프로젝트 descriptor와 autolinking JSON 경로만 같은 `U:` 경로로 정규화했다. node_modules의 도구 원본을 패치하지 않았다. 이 보정 후 이전에 실패한 gesture-handler를 포함한 codegen과 네이티브 C++ 빌드 단계가 진행됐다. 실패와 재실행 로그를 별도로 보존한다.

## 서명과 Firebase

기존 Expo 개발 앱의 기본 서명키만 읽어 NTFS 비공개 폴더 `%LOCALAPPDATA%/TranslaCat/android-development-signing`에 보관했다. 상속 권한을 제거하고 현재 사용자·SYSTEM·관리자만 접근하도록 설정했다. 키 값이나 비밀번호를 출력하거나 소스·GitHub에 넣지 않는다. 기존 Expo 자격을 생성·수정하지 않았다.

기존 `2c0940d4-e9d2-45c5-88d2-bf90eed8cdd6` APK를 설치한 SDK의 apksigner로 실제 검증했다. 인증서 SHA-256은 `600dc03d957e64a9ff9c85160f81fde10d7a9721a527a81277a86982455647d9`이며 내려받은 개발용 키의 인증서와 일치한다. 이 APK는 package `com.krumichan.translacat.dev`, versionCode 1, versionName 0.1.0, minSdk24, targetSdk36, compileSdk37이다. 과거 SDK가 없어서 미검증으로 남긴 기록과 이번 검증을 구분한다.

EAS Secret File의 내용을 로컬에서 읽을 수 없어서, 기존 Firebase CLI 로그인으로 공식 Management API에서 해당 Android 앱의 클라이언트 설정만 받았다. `translacat-mobile-dev`와 개발 package를 검증했다. FCM 서버 서비스 계정 개인 키는 요청·내장하지 않았고 Firebase 원격 설정은 변경하지 않았다.

## 실행

모바일 저장소에서 `npm.cmd run build:local:android`를 실행한다. PC 전용 경로와 lockfile 해시는 비공개 폴더의 `build-settings.json`에 저장한다. source·빌드·검증 근거는 `.codex-workspace/verification/shared/android-local-build-20261022/`에 보관한다.

빌드 명령은 소스 사본·해시를 만든 뒤 Expo prebuild와 고정 Gradle wrapper의 `:app:assembleRelease`를 실행한다. `--no-clean`으로 native 폴더를 지우지 않으며, 소유 기록이 없는 기존 native 폴더는 자동 수정하지 않는다. prebuild가 변경한 package.json은 원문으로 복원하고 빌드 후 소스 해시를 다시 확인한다. 생성한 `android/`는 기존 gitignore 대상인 빌드 도구 표준 출력이다. 서명 설정은 비밀번호를 코드에 복사하지 않고 비공개 JSON을 빌드 프로세스에서만 읽는다. 완성 APK의 서명과 기존 인증서 일치가 확인돼야 `apk-built.json` 및 설치용 APK를 내보낸다. 전역 NVM·PATH·JAVA_HOME을 수정하지 않는다.

최신 미읽음 표시 수정은 `ChatRoomScreen.tsx`에서 받은 메시지도 공통 안 읽음 숫자를 표시하고 0·null은 숨기는 변경이다. 서버·웹·DB를 다시 배포하지 않는다. 변경 전 화면 소스와 기존 APK를 보존한다.

## 완료한 빌드와 검증

2026-10-22 20:14 KST Windows native release 빌드가 `BUILD SUCCESSFUL`로 완료됐다. 마지막 실행은 33분 1초, Gradle 작업 786개 중 754개 실행·32개 캐시 재사용이었다. 이 시간은 처음 필요한 네이티브 도구·캐시를 준비한 이번 실행의 측정값이며 후속 빌드 시간을 보장하지 않는다.

| 산출물 | 결과 |
| --- | --- |
| APK | `artifacts/translacat-dev-local-2026-10-22T10-38-59-579Z-arm64.apk` |
| 크기 | 31,359,244 bytes, ARM64 |
| APK SHA-256 | `0e4ee0d591d13a315d3510f61804e4f9c82066153c785fa7ee42d81fe193cc97` |
| bundle SHA-256 | `6fb953bbdde1e9793cd32f540a5f41fa6cf11f2106fab92975a486c324ce37aa` |
| 서명 | apksigner 실제 검증·기존 인증서 일치 PASS |
| 매니페스트 | package·versionCode1·INTERNET·scheme·release 비디버그·ARM64 PASS |
| 공개 설정 | API·development·EAS project·package·scheme·owner·slug 7개 PASS |
| ZIP | 전체 CRC·경로·중복 및 zipalign 16KiB 페이지 정렬 PASS |
| 기존 글자 잘림 수정 | 실제 resources.arsc의 AppTheme 참조·TextView parent·typed boolean false PASS |
| 소스 | 151개 빌드 전후 해시 일치, UI 수정·알림 preference·badge 표식 포함 |
| 빌드 명령 | Node 구문·최종 프로젝트 ESLint PASS |

기존 미읽음 화면 수정의 452개 테스트·typecheck 통과 근거는 `mobile-message-unread-display-20261022`에 보존돼 있다. 이번 환경 구축에서 앱 기능 코드를 추가 변경하지 않았고 UI 파일 SHA-256 `a8599df59423ba4bcda6e069e172b5d4911593c1350eb68729d971f364d2cab5`를 보존했다. 새 로컬 APK의 기기 설치·미읽음 숫자 표시·Google 로그인·푸시 수신은 아직 확인하지 않았다.

최종 APK와 `apk-built.json`, `apk-verification.json`, `local-apk-text-resources.json`, `apk-zipalign.log`, `local-build-aligned.log`는 위 중앙 작업 폴더에 있다. 원본 package.json·README, 기존 APK, 부분 의존성 복사본과 실패 로그도 보존했다. Git 초기화·staging·commit·push·서버 배포·유료 등록을 하지 않았다.

## 휴대폰용 다운로드 링크

사용자가 정확한 APK와 대상 Expo 프로젝트를 지정한 질문에 **“승인 — 해당 APK를 이 Expo 프로젝트에 업로드”**라고 명시적으로 답했다. 이전 자동 검토의 승인 근거 부족 사유가 해결되어 동일한 공식 `eas upload`를 한 번 실행했다. 이전 거절 기록은 `apk-share-review-block.json`에 보존하고 해결 근거를 추가했다.

[검증된 로컬 APK 설치 페이지](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/b965582f-5822-4b59-8154-9ced4bd7f997)

Expo 기록은 2026-10-22 20:37 KST에 생성됐다. 공식 `build:view`로 프로젝트 ID·owner·slug·Android·INTERNAL·FINISHED 상태를 확인했다. Expo 공식 HTTPS 파일을 다시 내려받아 크기 31,359,244 bytes 및 SHA-256 `0e4ee0d591d13a315d3510f61804e4f9c82066153c785fa7ee42d81fe193cc97`가 검증된 원본과 정확히 일치함을 확인했다. APK 하나만 저장했으며 클라우드 재빌드·소스 폴더·개인 키 업로드·요금제 변경은 요청하지 않았다.

업로드 직후 CLI의 Git root 안내문이 JSON stdout 앞에 섞여 첫 결과 파싱만 실패했다. 원본 출력을 보존하고 끝의 공식 URL을 복구했으며 재업로드하지 않았다. `expo-upload.json`, `expo-upload-stderr.log`, `expo-upload-readback.json`, `apk-share.json` 및 다운로드 파일을 중앙 작업 폴더에 보존했다. 최종 완료 근거는 `local-build-completed.json`이다.

휴대폰에서는 기존 앱을 지우지 않고 설치 페이지에서 새 APK로 업데이트한다. 서명·package는 기존 앱과 같지만 실제 업데이트 설치·로그인·푸시·미읽음 숫자 표시는 사용자 기기에서 아직 확인하지 않았다. APK 공유 성공과 기기 기능 확인을 구분한다. 후속 로컬 빌드는 위 명령을 사용하고 새 산출물은 별도로 검증한다. Expo 업로드 횟수나 서비스 정책이 무제한이라는 보장은 하지 않는다.

## 공식 참고

- [Android SDK Command-line tools와 약관](https://developer.android.com/studio#command-tools)
- [SDK Manager](https://developer.android.com/tools/sdkmanager)
- [Expo 로컬 release 빌드와 기존 키 재사용](https://docs.expo.dev/guides/local-app-production/)
- [Firebase Android 클라이언트 설정 조회](https://firebase.google.com/docs/reference/firebase-management/rest/v1beta1/projects.androidApps/getConfig)


## 새 로컬 APK 사용자 확인 후속

사용자가 새 APK 업데이트 후 “받은 메시지에도 안 읽음 숫자 잘 보여”라고 직접 보고했다. Expo 설치 페이지 `b965582f-5822-4b59-8154-9ced4bd7f997` 안내 이후의 **업데이트 설치 및 받은 메시지의 양수 미읽음 표시 USER_REPORTED_PASS**로 기록한다. 근거는 `.codex-workspace/verification/shared/android-local-build-20261022/device-user-acceptance.json`이다. 기존 APK·로그·자동 검사 및 업로드 파일 해시 근거는 보존한다.

사용자 폰의 설치 바이너리를 독립 식별하거나 모든 시나리오를 직접 조작한 것은 아니다. 이번 확인으로 전역 알림 OFF/ON, 방별 무음, 읽음 후 배지·알림창 정리, 계정 전환 및 iPhone까지 통과했다고 확대하지 않는다. 이전 절의 새 APK 미제공·수신 메시지 표시 수락 대기는 이 후속으로 갱신하며 나머지 검증 범위는 그대로 둔다.

## 알림 설정·방별 무음·배지 사용자 확인 후속

사용자는 새 로컬 APK 설치·수신 메시지 표시 확인에 이어 “전체 알림이랑 방별 무음, 배지까지 확인했어 문제 없어!”라고 보고했다. 이 세 항목은 USER_REPORTED_PASS로 갱신한다. 근거는 `android-local-build-20261022/notification-device-user-acceptance.json`이다. 이전 초기·배포·APK·자동 검사 기록은 보존한다. 계정 전환·iPhone·모든 경합 시나리오를 독립 검증한 것으로 확대하지 않는다.

## 같은 Expo 프로젝트 APK 공유 상시 승인 후속

2026-10-23 사용자는 신고·탈퇴 APK 31,448,720바이트/SHA-256 `925f9a06591ef5393626cd7eccd38ab7914409269176997edb615b890a0184cf`의 공유 질문에 “이건 항상 승인할게”라고 직접 답했다. 검증한 로컬 APK 하나를 기존 `https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile`에 무료 업로드해 설치 링크를 만드는 작업은 이후 같은 범위에서 다시 승인받지 않는다. 각 파일의 서명·크기·해시 검증과 공유 결과 대조는 계속 수행한다. 근거는 `mobile-safety-deletion-20261023/upload-authorization.json`이다.

이 승인은 유료 클라우드 빌드, 요금제·결제 변경, 다른 계정/프로젝트 업로드, 개인 키·소스 폴더 업로드를 포함하지 않는다. 공식 `eas upload`는 로컬 앱 파일을 공유하는 명령이며, 현재 표준 빌드는 PC의 Expo prebuild와 Gradle로 수행한다. [공식 CLI](https://docs.expo.dev/eas/cli/#eas-upload)와 [공식 로컬 빌드 안내](https://docs.expo.dev/guides/local-app-overview/)를 참조한다. Free 요금제 초과 사용이 자동 추가 과금을 발생시키지 않는다는 [요금 FAQ](https://docs.expo.dev/billing/faq/)와 별개로, 업로드 무제한·공유 파일 영구 보관을 보장하지 않는다. 서비스 제한이 발생하면 유료 변경을 자동 실행하지 않고 로컬 원본을 보존한다.
