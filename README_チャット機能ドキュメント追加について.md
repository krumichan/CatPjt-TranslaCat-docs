# TranslaCat チャット機能ドキュメント追加について

このzipは、`CatPjt-TranslaCat-docs` リポジトリのルートに展開して利用する想定のドキュメントセットです。
韓国語版と日本語版をそれぞれ用意しています。

## 追加される主な内容

```text
00 要件/チャット機能/ko/
00 要件/チャット機能/ja/
01 基本設計/チャット機能/ko/
01 基本設計/チャット機能/ja/
99 Project管理/チャット機能/ko/
99 Project管理/チャット機能/ja/
.github/ISSUE\_TEMPLATE/
```

## 使い方

1. zipを展開します。
2. 展開された中身を `CatPjt-TranslaCat-docs` のルートにコピーします。
3. 内容を確認します。
4. GitHubへpushします。

```bash
git status
git add .
git commit -m "docs: add chatting feature planning documents"
git push origin main
```

## 次回以降のChatGPT引き継ぎ方法

新しいチャットで作業を続ける場合は、以下のURLを共有してください。

```text
現在状況ドキュメント:
00 要件/チャット機能/ja/00\_チャット機能\_現在状況.md
または
00 要件/チャット機能/ko/00\_채팅기능\_현재상황.md

GitHub Project:
https://github.com/users/krumichan/projects/2

現在作業中Issue:
作業中のGitHub Issue URL
```

