# 모바일 글자 잘림 전체 점검

# モバイルの文字切れを全体点検

2026-10-22, 사용자는 직전 Android APK `6b711f15-8158-4472-a809-24ecd6a4e5c7`에서 빨간 **강퇴 · 재입장 차단** 문구가 정상 표시됨을 스크린샷으로 확인했다. 같은 화면의 역할 배지는 처음에는 **참여자**, 재진입하면 **참여**로 잘렸다. 이 문서는 배지 하나와 앱 전반의 글자 측정·공간 배치를 함께 점검한 후속 기록이다.

2026-10-22、ユーザーは直前の Android APK `6b711f15-8158-4472-a809-24ecd6a4e5c7` で、赤い **강퇴 · 재입장 차단（強制退室・再入室をブロック）** の文言が正常に表示されることをスクリーンショットで確認した。同じ画面の役割バッジは、最初は **참여자（参加者）**、再入場すると **참여** に切れていた。本書は、バッジ 1 件とアプリ全体の文字測定・空間配置を併せて点検した後続記録である。

## 공통 Android 원인과 대응

## Android 共通の原因と対応

역할 문자열은 `ParticipantIdentity`에서 항상 **참여자**로 만들어진다. 데이터나 한 줄 제한으로 마지막 글자를 제거하는 처리는 없다. 현재 설치된 React Native 0.88.0의 `ReactTextView`에는 Android 15 이상에서 `useBoundsForWidth`를 끄는 처리가 없고, 측정용 `TextLayoutManager`는 다른 줄바꿈 기준을 쓴다.

役割の文字列は `ParticipantIdentity` で常に **참여자（参加者）** と生成される。データや 1 行制限で末尾の文字を除去する処理はない。現在インストールされている React Native 0.88.0 の `ReactTextView` には、Android 15 以降で `useBoundsForWidth` を無効にする処理がなく、測定用の `TextLayoutManager` は異なる改行基準を使う。

[React Native 공식 수정 dcdb52b / #58280](https://github.com/react/react-native/commit/dcdb52b)는 Android의 글자 폭 측정과 실제 그리기 기준이 달라 마지막 글자나 줄이 사라지는 문제를 설명한다. OEM 시스템 글꼴 및 삼성 기기의 사례와 뷰 재활용도 다룬다. 이번 증상과 일치하는 근거이지만, 사용자의 Android 버전·네이티브 글자 측정값을 직접 수집해 같은 원인으로 재현한 것은 아니다.

[React Native の公式修正 dcdb52b / #58280](https://github.com/react/react-native/commit/dcdb52b) は、Android の文字幅測定と実際の描画基準の違いにより、末尾の文字や行が消える問題を説明している。OEM システムフォント、Samsung 端末の事例、ビューの再利用も扱う。今回の症状に合致する根拠だが、ユーザーの Android バージョン・ネイティブ文字測定値を直接収集して同じ原因で再現したわけではない。

로컬 Expo 플러그인 `plugins/with-android-text-layout.cjs`에서 생성된 AppTheme의 `android:textViewStyle`을 전용 `TranslaCatTextView`에 연결한다. 기존 텍스트 스타일이 있으면 그 스타일을 부모로 상속하며, 기본 부모는 `Widget.AppCompat.TextView`다. 전용 스타일의 `android:useBoundsForWidth`를 명시적으로 `false`로 두어 Android 기본 스타일보다 우선하도록 했다.

ローカル Expo プラグイン `plugins/with-android-text-layout.cjs` で、生成された AppTheme の `android:textViewStyle` を専用の `TranslaCatTextView` に接続する。既存の文字スタイルがあればそれを親として継承し、既定の親は `Widget.AppCompat.TextView` とする。専用スタイルの `android:useBoundsForWidth` を明示的に `false` とし、Android の既定スタイルより優先するようにした。

[Android 테마 속성 우선순위](https://developer.android.com/reference/android/content/res/Resources.Theme#obtainStyledAttributes(android.util.AttributeSet,%20int[],%20int,%20int))와 [AOSP TextView 생성자](https://raw.githubusercontent.com/aosp-mirror/platform_frameworks_base/android-15.0.0_r1/core/java/android/widget/TextView.java)를 확인했다. 현재 RN의 초기화·재활용 경로는 이 값을 다시 바꾸지 않는다. Android 35 미만은 해당 속성을 읽지 않으며 `tools:targetApi="35"`는 lint 설명이다. RN Java 소스·SDK·의존성을 수정하거나 업그레이드하지 않았다. 공식 수정과 마찬가지로 Android 15의 글꼴 잉크 돌출부 여백 예약 대신 기존 advance 기준을 유지하므로 글꼴 끝의 미세한 잉크 영역은 실기기 확인 범위다. 향후 공식 수정이 포함된 RN으로 올릴 때 이 설정의 제거를 검토한다.

[Android のテーマ属性の優先順位](https://developer.android.com/reference/android/content/res/Resources.Theme#obtainStyledAttributes(android.util.AttributeSet,%20int[],%20int,%20int))と [AOSP TextView コンストラクター](https://raw.githubusercontent.com/aosp-mirror/platform_frameworks_base/android-15.0.0_r1/core/java/android/widget/TextView.java)を確認した。現在の RN の初期化・再利用経路は、この値を再変更しない。Android 35 未満は当該属性を読まず、`tools:targetApi="35"` は lint 向けの説明である。RN の Java ソース・SDK・依存関係は変更もアップグレードもしていない。公式修正と同様に、Android 15 のフォントのインク突出部分の余白確保ではなく既存の advance 基準を維持するため、文字末尾の微細なインク領域は実機確認の範囲となる。今後、公式修正を含む RN に更新するときに、この設定の除去を検討する。

## 화면별 점검과 보강

## 画面別の点検と補強

UI 소스 전체에서 글자 크기에 맞추는 폭, 가로 배치의 축소, 고정 높이, 줄 수 제한과 `overflow`를 조사하고 두 범위의 독립 읽기 검토를 수행했다. 아래 표는 소스 검토와 수정 내역이며 모든 Android/iPhone의 실제 화면 검증을 뜻하지 않는다.

UI ソース全体で、文字サイズに合わせる幅、横配置の縮小、固定高、行数制限、`overflow` を調査し、2 範囲の独立した読み取りレビューを実施した。以下の表はソースレビューと変更内容であり、すべての Android/iPhone の実画面検証を意味しない。

| 영역 | 점검 결과와 수정 |
| --- | --- |
| 참여자·역할 배지 | 원문은 정상. 긴 이름은 줄바꿈하고 아바타·역할 배지의 폭과 글자 높이를 축소하지 않게 보강했다. |
| 계정·알림 상태 배지 | 배지 최대 폭을 화면 안으로 제한하고 세로 수축을 방지했다. 서비스 표시의 점과 글자 줄 높이도 명시했다. |
| 안 읽은 메시지 개수 | Text의 `overflow: hidden`을 없애고 배경·패딩은 View, 글자·줄 높이는 Text로 분리했다. 긴 방 이름은 남은 폭에서 줄바꿈한다. |
| 친구 선택·대화 | `선택`에서 `선택됨`으로 바뀔 때도 버튼 최종 안쪽 폭을 사용하고 필요한 글자 높이를 보존한다. |
| 오픈방 참여 | `이 프로필로 참여` 버튼도 최종 안쪽 폭·가운데 정렬·자동 높이를 사용한다. |
| 대화방 메뉴 | 제목은 닫기 버튼 옆에서 줄바꿈되고, 메뉴 항목은 두 줄이면 높이가 늘어난다. |
| 하단 탭 | 시스템 글꼴 배율에 따라 라벨 확대분만큼 탭 높이를 추가한다. 내비게이션 기본 라벨은 한 줄로 표시된다. |
| 아이콘 | 고정 44 높이 안의 뒤로·새로고침·메뉴·닫기 장식 문자만 글꼴 확대를 끈다. 부모 버튼에 접근성 이름이 있으며 본문·역할·버튼 문구의 글꼴 확대는 유지한다. |
| 로그인·설정·ID·AI·채팅 본문 | 글자 끝을 자르는 문자열 처리나 고정 높이·줄 제한은 추가하지 않았다. 공통 Android 설정이 이 영역에도 적용된다. |
| 의도된 줄임 | 사용자가 요청한 한 줄 방 제목과 오픈방 목록 설명 세 줄은 유지한다. 로고 이미지의 잘라내기는 문자와 구분한다. |

| 領域 | 点検結果と変更 |
| --- | --- |
| 参加者・役割バッジ | 原文は正常。長い名前は改行し、アバター・役割バッジの幅と文字高が縮まないよう補強した。 |
| アカウント・通知状態バッジ | バッジの最大幅を画面内に制限し、縦方向の縮小を防いだ。サービス表示の点と文字の行高も明示した。 |
| 未読メッセージ数 | Text の `overflow: hidden` を除去し、背景・padding は View、文字・行高は Text に分離した。長いルーム名は残り幅で改行する。 |
| 友達選択・会話 | `선택`（選択）から `선택됨`（選択済み）に変わるときも、ボタンの最終的な内幅を使用し、必要な文字高を保つ。 |
| オープンルームへの参加 | `이 프로필로 참여`（このプロフィールで参加）ボタンにも、最終的な内幅・中央揃え・自動高を使う。 |
| チャットルームメニュー | タイトルは閉じるボタンの横で改行し、メニュー項目が 2 行になれば高さが増える。 |
| 下部タブ | システムのフォント倍率に応じ、ラベルの拡大量だけタブの高さを追加する。ナビゲーションの既定ラベルは 1 行で表示される。 |
| アイコン | 固定高 44 内の戻る・更新・メニュー・閉じるの装飾文字だけフォント拡大を無効にする。親ボタンにはアクセシビリティ名があり、本文・役割・ボタン文言のフォント拡大は維持する。 |
| ログイン・設定・ID・AI・チャット本文 | 文字末尾を切る文字列処理や固定高・行制限は追加していない。共通 Android 設定はこの領域にも適用される。 |
| 意図した省略 | ユーザーが求めたルームタイトル 1 行と、オープンルーム一覧の説明 3 行を維持する。ロゴ画像の切り抜きは文字と区別する。 |

## 검증

## 検証

플러그인 집중 테스트 **7/7**과 기존 테스트를 합한 전체 **329/329**를 통과했다. 집중 범위는 기존 AppTheme·스플래시·글꼴·색상 보존, 기존 텍스트 스타일 상속과 자식 설정 우선 적용, 반복 시 동일 결과, 모호하거나 누락된 테마 처리, 실제 Expo Android styles mod 실행과 기존 mod 체인 유지다.

プラグインの集中テスト **7/7** と既存テストを合わせた全体 **329/329** が通過した。集中範囲は、既存 AppTheme・スプラッシュ・フォント・色の保持、既存文字スタイルの継承と子設定の優先適用、繰り返し時の同一結果、曖昧または欠落したテーマの処理、実際の Expo Android styles mod の実行と既存 mod チェーンの維持である。

현재 동결 입력은 126개이며 manifest SHA-256은 `fb4427ef22bf730a5fc9e636d2480aa1ea3b271b1afc815187a37afa6bd35d85`다. 전체 lint 경고 0·TypeScript·device 공개 HTTPS config·Android/iOS/web JavaScript export가 모두 exit 0이며 검사 중 소스 변경은 없었다. 전체 검사 영수증 SHA-256은 `5de7bb693caf292c4589cfbc98cdb52058ad82422dfdb5070b3efa714223bc80`이다. JavaScript 검사만으로 Android 테마 적용이나 재진입 시 표시 성공을 기록하지 않는다.

現在の凍結入力は 126 件で、manifest SHA-256 は `fb4427ef22bf730a5fc9e636d2480aa1ea3b271b1afc815187a37afa6bd35d85`。全体 lint 警告 0・TypeScript・device 公開 HTTPS config・Android/iOS/web JavaScript export はすべて exit 0 で、検査中のソース変更はなかった。全体検査記録の SHA-256 は `5de7bb693caf292c4589cfbc98cdb52058ad82422dfdb5070b3efa714223bc80`。JavaScript 検査だけで Android テーマの適用や再入場時の表示成功を記録しない。

실제 APK의 `resources.arsc`에서 AppTheme → TranslaCatTextView 연결과 `android:useBoundsForWidth`의 typed boolean `false` 검사를 통과했다. [AOSP 공개 속성 ID](https://raw.githubusercontent.com/aosp-mirror/platform_frameworks_base/android-15.0.0_r1/core/res/res/values/public-final.xml)는 `textViewStyle=0x01010084`, `useBoundsForWidth=0x01010698`이다. 전체 스타일 구성 1,384개를 읽었고 두 대상 스타일은 각각 기본 구성 1개이며 충돌하는 다른 변형은 없었다. 부모는 `Widget.AppCompat.TextView`, boolean type은 18·data는 0이었다. 직전 APK에는 이 연결과 전용 스타일이 없음을 확인했다. 이 검사는 이름이 지정된 컴파일 스타일 연결을 증명하며, 실행 시 AndroidManifest·Activity의 실제 테마 선택이나 삼성 기기 화면의 확인과 구분한다.

実際の APK の `resources.arsc` で、AppTheme → TranslaCatTextView の接続と `android:useBoundsForWidth` の typed boolean `false` の検査が通過した。[AOSP 公開属性 ID](https://raw.githubusercontent.com/aosp-mirror/platform_frameworks_base/android-15.0.0_r1/core/res/res/values/public-final.xml)は `textViewStyle=0x01010084`、`useBoundsForWidth=0x01010698`。全スタイル構成 1,384 件を読み、対象の 2 スタイルはそれぞれ基本構成 1 件で、競合する別 variant はなかった。親は `Widget.AppCompat.TextView`、boolean type は 18・data は 0 だった。直前の APK にはこの接続と専用スタイルがないことを確認した。この検査は名前で特定したコンパイル済みスタイルの接続を証明し、実行時の AndroidManifest・Activity の実際のテーマ選択や Samsung 端末の画面確認とは区別する。

## 휴대폰 확인

## 端末での確認

2026-10-22 새 APK 안내와 재진입·새로고침 확인 요청 뒤 사용자가 **“확인했어 이제 다음 작업은 머야?”**라고 답했다. 이번 글자 표시 확인은 사용자의 현재 Android 기기·글꼴 설정에서 수락한 것으로 기록한다. 정확한 설치 바이너리를 독립 확인하거나 큰 글꼴·모든 화면 조합·iPhone·초대와 관리 변경 기능 전체를 새로 검증한 것으로 확대하지 않는다. 사용자 답변과 범위는 중앙 작업 폴더의 `user-acceptance-20261022.json`, 갱신 전 문서는 `audit-before-user-acceptance-20261022.md`에 보존했다.

2026-10-22、新しい APK の案内と再入場・更新の確認依頼後、ユーザーは **『確認した、次は何の作業？』** と回答した。今回の文字表示確認は、ユーザーの現在の Android 端末・フォント設定で受け入れられたものと記録する。正確なインストール済みバイナリを独立確認したことや、大きなフォント・全画面の組み合わせ・iPhone・招待と管理変更機能全体を新たに検証したことには広げない。ユーザー回答と範囲は中央作業フォルダーの `user-acceptance-20261022.json`、更新前の文書は `audit-before-user-acceptance-20261022.md` に保存した。

이 수락 후 [#53](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/53)의 실제 확인 체크를 현재 기기·글꼴 범위로 좁혀 완료하고 closed/completed로 처리했다. 기존 Project 12 자동화가 해당 항목을 Done으로 바꾼 것을 재조회했으며 중복 보드 수정은 하지 않았다. 근거는 `issues/user-acceptance-confirmed.json`이다. #52와 다른 항목은 유지했다. 이번 후속은 기록 갱신만이며 제품 소스·추가 빌드·서버·Git 변경·파일 삭제는 없다.

この受け入れ後、[#53](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/53) の実確認チェックを現在の端末・フォント範囲に限定して完了し、closed/completed とした。既存 Project 12 の自動化が当該項目を Done に変えたことを再取得で確認し、重複したボード変更は行わなかった。根拠は `issues/user-acceptance-confirmed.json`。#52 と他の項目は維持した。この後続作業は記録更新のみであり、製品ソース・追加ビルド・サーバー・Git の変更やファイル削除はない。

수정 APK로 업데이트한 뒤 같은 참여자 화면에서 **참여자·관리자·방장** 문구를 확인한다. 뒤로 나갔다 재진입하고 새로고침해도 전체 문구가 유지되는지 확인한다. 화면 확인을 위해 실제 강퇴·위임·종료 버튼을 누를 필요는 없다.

修正 APK に更新してから同じ参加者画面で、**참여자・관리자・방장（参加者・管理者・ルームオーナー）** の文言を確認する。戻って再入場し、更新しても文言全体が維持されるかを確認する。画面確認のために実際の強制退室・委任・終了ボタンを押す必要はない。

큰 글꼴 설정에서는 메뉴·친구 선택·오픈방 참여·하단 탭도 확인 대상이다. iPhone 빌드·실기기 화면은 아직 검증하지 않았다. 이번 스크린샷을 초대·복사·공유·관리 변경 기능 전체의 성공으로 확대하지 않는다.

大きなフォント設定では、メニュー・友達選択・オープンルーム参加・下部タブも確認対象となる。iPhone ビルド・実機画面はまだ検証していない。今回のスクリーンショットを、招待・コピー・共有・管理変更機能全体の成功へ広げない。

## 보존과 추적

## 保存と追跡

제품 기존 파일 8개를 수정하고 플러그인·집중 테스트 2개를 추가했다. 기존 원본 8개, 이전 APK 입력과의 비교, diff·검사 로그·빌드 근거는 `.codex-workspace/verification/shared/mobile-text-layout-audit-20261022/`에 보존한다. 파일 삭제·Git staging·commit·push·서버 배포·키 변경·유료 작업은 수행하지 않았다.

既存の製品ファイル 8 件を変更し、プラグイン・集中テストの 2 件を追加した。既存原本 8 件、以前の APK 入力との比較、diff・検査ログ・ビルド根拠は `.codex-workspace/verification/shared/mobile-text-layout-audit-20261022/` に保存する。ファイル削除・Git staging・commit・push・サーバーデプロイ・キー変更・有料作業は実施していない。

기존 모바일 진행 이슈 [#52](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/52)와 새 전용 이슈 [#53](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/53)에서 실제 기기 확인 대기를 추적한다. 중복 검색 후 #53을 만들었고 기존 [모바일 Project 12](https://github.com/users/krumichan/projects/12)에 1회 추가하고 다시 읽어 확인했다. 최종 APK 검증 후 #53의 APK 준비 체크와 설치 링크 및 [#52의 기존 댓글](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/52#issuecomment-6312207253)의 링크·공통 텍스트 수정 문장만 각각 1회 갱신하고 정확한 본문 재조회를 확인했다. 근거는 `issues/final-apk-public-update-confirmed.json`에 보존했다. #53은 OPEN·In progress, 실제 기기 체크는 미완료이며 확인 전에는 Done으로 표시하지 않는다. 공개 이슈에는 짧은 증상·수정 범위·검증 결과만 남기며 개인 닉네임·코드·내부 운영 정보를 넣지 않았다.

既存のモバイル進行 issue [#52](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/52) と新しい専用 issue [#53](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/53) で実機確認待ちを追跡する。重複を検索してから #53 を作り、既存の[モバイル Project 12](https://github.com/users/krumichan/projects/12) に 1 回追加し、再取得で確認した。最終 APK 検証後、#53 の APK 準備チェックとインストールリンク、および [#52 の既存コメント](https://github.com/krumichan/CatPjt-TranslaCat-docs/issues/52#issuecomment-6312207253) のリンク・共通文字修正の文だけをそれぞれ 1 回更新し、正確な本文の再取得を確認した。根拠は `issues/final-apk-public-update-confirmed.json` に保存した。#53 は OPEN・In progress、実機チェックは未完了で、確認前に Done と表示しない。公開 issue には短い症状・修正範囲・検証結果のみを残し、個人のニックネーム・コード・内部運用情報は入れていない。

## 새 Android APK

## 新しい Android APK

무료 Android 빌드 잔여량과 기존 계정·프로젝트·공개 연결 설정을 재확인한 뒤 [APK 58bbf721-3a3a-4f82-a825-6862a3c6789b](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/58bbf721-3a3a-4f82-a825-6862a3c6789b)를 2026-10-22 10:34:22 KST에 1회 접수했고 **10:48:33 KST FINISHED**를 확인했다. 설치 파일은 **93,456,647 bytes**, SHA-256은 `8c113403c5d1c0b30451956bfe97148c38fa5e2be9392e1659201a627c2741ff`이다.

無料 Android ビルドの残数と既存アカウント・プロジェクト・公開接続設定を再確認後、[APK 58bbf721-3a3a-4f82-a825-6862a3c6789b](https://expo.dev/accounts/kurumichans-team/projects/translacat-mobile/builds/58bbf721-3a3a-4f82-a825-6862a3c6789b) を 2026-10-22 10:34:22 KST に 1 回受付し、**10:48:33 KST FINISHED** を確認した。インストールファイルは **93,456,647 bytes**、SHA-256 は `8c113403c5d1c0b30451956bfe97148c38fa5e2be9392e1659201a627c2741ff`。

공식 메타데이터와 기본 TLS 다운로드 1회, ZIP·내장 번들, 공개 설정 7항목, 직전 APK와 번들 변경, 동결 소스 126개의 변경 0 및 컴파일 스타일 검사가 모두 통과했다. API 메타데이터 조회가 지연됐지만 새 빌드 요청이나 실제 파일 전송은 중복하지 않았다. 유료 작업·원격 키/환경 설정 변경·서버 배포·의존성 설치·Git 작업·파일 삭제는 없었다. AndroidManifest 전체·서명·ZIP 전체 CRC·클라우드 소스 전체 바이트 일치와 실제 폰 표시는 이 검사로 확인하지 않았다.

公式メタデータと通常の TLS ダウンロード 1 回、ZIP・内蔵 bundle、公開設定 7 項目、直前 APK からの bundle 変更、凍結ソース 126 件の変更 0、コンパイル済みスタイル検査がすべて通過した。API メタデータ取得は遅れたが、新たなビルド要求や実ファイル転送は重複しなかった。有料作業・リモートキーと環境設定の変更・サーバーデプロイ・依存関係のインストール・Git 操作・ファイル削除はなかった。AndroidManifest 全体・署名・ZIP 全体の CRC・クラウド全ソースのバイト一致・実端末表示は、この検査では確認していない。

최종 근거는 중앙 작업 폴더의 `apk/apk-completed.json` (SHA-256 `f01e5159e516baa888bf11aaaf38667a096092f4be2db810bb4b1e6aba10f9fd`), `apk/verification/verification-manifest.json` (`de1593aacf2dd18e364a3650f3b912d9d2b228527ef9832a83dc11062854ff28`), `apk/verification/compiled-text-style-verification.json` (`649f89f1fd53605d6f69368988da1e5cb157b4550efc1a488b81bc0b8056e3c9`)에 보존했다. 루트가 결과와 세 기록의 SHA-256을 다시 읽어 대조했다. 이후 위 사용자 응답으로 현재 Android 글자 표시 확인을 수락했다. 빌드 당시 영수증의 pending 값은 당시 기록으로 그대로 보존한다.

最終根拠は中央作業フォルダーの `apk/apk-completed.json`（SHA-256 `f01e5159e516baa888bf11aaaf38667a096092f4be2db810bb4b1e6aba10f9fd`）、`apk/verification/verification-manifest.json`（`de1593aacf2dd18e364a3650f3b912d9d2b228527ef9832a83dc11062854ff28`）、`apk/verification/compiled-text-style-verification.json`（`649f89f1fd53605d6f69368988da1e5cb157b4550efc1a488b81bc0b8056e3c9`）に保存した。root が結果と 3 記録の SHA-256 を再読して照合した。その後、前述のユーザー回答により現在の Android 文字表示確認を受け入れた。ビルド時の記録の pending 値は当時の記録としてそのまま保存する。
