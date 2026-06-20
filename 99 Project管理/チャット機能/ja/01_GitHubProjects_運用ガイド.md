# GitHub Projects 運用ガイド

## 1. 目的

TranslaCatチャット機能開発はFE/BE/docs repoが連動するため、作業管理はGitHub Projectsで統合管理する。

## 2. Project名

```text
TranslaCat Chatting Development
```

## 3. ステータスカラム

| カラム | 意味 |
|---|---|
| Backlog | いずれ対応する候補作業 |
| Todo | 今回の開発範囲に含まれ、近日着手する作業 |
| In Progress | 現在作業中 |
| Review | 実装後の確認/レビュー/テスト中 |
| Done | 完了 |
| Hold | 保留またはブロック中 |

## 4. Custom Fields

| Field | 値 |
|---|---|
| Phase | 1次, 2次 |
| Area | BE, FE, Docs, QA, Infra, AI, Common |
| Priority | High, Medium, Low |
| Type | Feature, Docs, Test, Research, Refactor, Bug |
| Target Repo | BE, FE, Docs, AI |

## 5. Issue作成先

| 作業種類 | Issue作成先 |
|---|---|
| BE作業 | CatPjt-TranslaCat-be |
| FE作業 | CatPjt-TranslaCat-fe |
| 文書/企画作業 | CatPjt-TranslaCat-docs |
| 共通/運用/設計判断 | CatPjt-TranslaCat-docs |

## 6. Issueタイトル規則

```text
[BE] チャットルーム作成API実装
[FE] チャットルーム一覧画面実装
[DOCS] DB設計作成
[QA] WebSocketメッセージ送受信確認
[ARCH] Redis導入可否検討
```

## 7. Epic運用方式

docs repoにEpic Issueを作成する。

```text
[EPIC] 1次チャットMVP開発
[EPIC] 2次チャット拡張開発
```

各BE/FE/Docs作業IssueはEpicに接続する。

## 8. 新しいChatGPTチャットへの引き継ぎ規則

新しいチャットで作業を続ける場合、以下3つのURLを共有する。

```text
1. 現在状況ドキュメントURL
2. GitHub Project URL
3. 現在作業中Issue URL
```

## 9. 作業終了時の更新規則

作業が完了するたびに以下を更新する。

- GitHub Project状態
- 関連Issueチェックリスト
- docs repoの現在状況ドキュメント
- 必要に応じてAPI/DB/WebSocket設計ドキュメント
