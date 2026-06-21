# チャット API 詳細設計

## 1. 文書目的

本書は、TranslaCatチャット機能1次MVPで使用するREST API詳細設計を定義する。

WebSocketメッセージ送受信は別文書で定義し、本書ではチャットルーム/メンバー/メッセージ取得を中心に扱う。

---

## 2. 共通方針

- 認証済みユーザーのみ呼び出し可能。
- レスポンス形式は既存TranslaCat共通レスポンスに従う。
- ユーザーがチャットルームメンバーではない場合、取得/修正/送信関連APIは拒否する。
- API path prefixは `/api/v1/chat` を使用する。

---

## 3. チャットルーム作成

```text
POST /api/v1/chat/rooms
```

Request:

```json
{
  "roomType": "GROUP",
  "name": "日本語練習ルーム",
  "description": "日本語で会話するルームです。",
  "memberUserIds": [2, 3]
}
```

Response:

```json
{
  "id": 1,
  "roomType": "GROUP",
  "name": "日本語練習ルーム",
  "description": "日本語で会話するルームです。",
  "myRole": "OWNER",
  "memberCount": 3,
  "createdAt": "2026-06-21T10:00:00"
}
```

処理基準:

- 作成者はOWNERとして登録する。
- memberUserIdsに作成者自身が含まれる場合は重複除去する。
- 1次ではDIRECT/GROUPのみ許可する。

---

## 4. 自分のチャットルーム一覧取得

```text
GET /api/v1/chat/rooms
```

Response:

```json
{
  "items": [
    {
      "id": 1,
      "roomType": "GROUP",
      "name": "日本語練習ルーム",
      "description": "日本語で会話するルームです。",
      "memberCount": 3,
      "lastMessage": "今日何食べる？",
      "lastMessageAt": "2026-06-21T10:05:00",
      "myRole": "OWNER"
    }
  ],
  "hasNext": false,
  "nextCursor": null
}
```

---

## 5. チャットルーム詳細取得

```text
GET /api/v1/chat/rooms/{roomId}
```

Response:

```json
{
  "id": 1,
  "roomType": "GROUP",
  "name": "日本語練習ルーム",
  "description": "日本語で会話するルームです。",
  "myRole": "OWNER",
  "memberCount": 3,
  "myLanguageSetting": {
    "originalLanguageCode": "ja",
    "translationLanguageCode": "ko",
    "showOriginal": true,
    "showTranslation": true
  }
}
```

---

## 6. チャットルームメンバー一覧取得

```text
GET /api/v1/chat/rooms/{roomId}/members
```

Response:

```json
{
  "items": [
    {
      "userId": 1,
      "name": "User A",
      "profileImageUrl": null,
      "role": "OWNER",
      "translationLanguageCode": "ko",
      "joinedAt": "2026-06-21T10:00:00"
    }
  ]
}
```

---

## 7. チャットルームメンバー追加

```text
POST /api/v1/chat/rooms/{roomId}/members
```

Request:

```json
{
  "userIds": [4, 5]
}
```

処理基準:

- OWNERまたはADMINのみ追加可能。
- 既にactive memberのユーザーは重複追加しない。

---

## 8. チャットルーム退出

```text
DELETE /api/v1/chat/rooms/{roomId}/members/me
```

処理基準:

- 自分自身のみ退出処理する。
- OWNER退出ポリシーは別途検討する。
- 1次ではOWNER退出制限を優先検討する。

---

## 9. 自分の言語設定変更

```text
PATCH /api/v1/chat/rooms/{roomId}/members/me/language-settings
```

Request:

```json
{
  "originalLanguageCode": "ja",
  "translationLanguageCode": "ko",
  "showOriginal": true,
  "showTranslation": true
}
```

---

## 10. メッセージ一覧取得

```text
GET /api/v1/chat/rooms/{roomId}/messages
```

Query:

```text
?cursorMessageId={cursorMessageId}&limit=100
```

Response:

```json
{
  "items": [
    {
      "id": 100,
      "roomId": 1,
      "senderType": "USER",
      "senderUserId": 1,
      "senderName": "User A",
      "messageType": "TEXT",
      "content": "今日何食べる？",
      "status": "SENT",
      "createdAt": "2026-06-21T10:05:00",
      "translations": [
        {
          "languageCode": "ko",
          "status": "COMPLETED",
          "translatedContent": "오늘 뭐 먹을래?"
        }
      ]
    }
  ],
  "hasNext": true,
  "nextCursorMessageId": 80
}
```

---

## 11. APIエラー候補

| code | 説明 |
|---|---|
| CHAT_ROOM_NOT_FOUND | チャットルームなし |
| CHAT_ROOM_ACCESS_DENIED | アクセス権限なし |
| CHAT_ROOM_MEMBER_NOT_FOUND | メンバーではない |
| CHAT_ROOM_MEMBER_ALREADY_EXISTS | 既に参加中 |
| CHAT_MESSAGE_NOT_FOUND | メッセージなし |
| CHAT_MESSAGE_CONTENT_EMPTY | メッセージ内容なし |
| CHAT_MESSAGE_CONTENT_TOO_LONG | メッセージ長さ超過 |
