# チャット機能 REST API設計

## 1. 共通

- すべてのAPIはログインユーザーを基準に処理する。
- チャットルームへのアクセス権限がない場合は403を返却する。
- 存在しないチャットルームまたはメッセージは404を返却する。

## 2. チャットルームAPI

### 2.1 チャットルーム作成

```http
POST /api/v1/chat/rooms
```

#### Request

```json
{
  "roomType": "GROUP",
  "name": "Study Room",
  "memberUserIds": [2, 3],
  "studyLanguageCode": "JA",
  "receiveTranslationLanguageCode": "JA"
}
```

#### Response

```json
{
  "id": 1,
  "roomType": "GROUP",
  "name": "Study Room",
  "myRole": "OWNER",
  "memberCount": 3
}
```

### 2.2 自分のチャットルーム一覧取得

```http
GET /api/v1/chat/rooms
```

### 2.3 チャットルーム詳細取得

```http
GET /api/v1/chat/rooms/{roomId}
```

### 2.4 チャットルーム言語設定変更

```http
PATCH /api/v1/chat/rooms/{roomId}/my-settings
```

#### Request

```json
{
  "studyLanguageCode": "JA",
  "receiveTranslationLanguageCode": "EN"
}
```

### 2.5 グループチャットメンバー追加

```http
POST /api/v1/chat/rooms/{roomId}/members
```

### 2.6 チャットルーム退出

```http
POST /api/v1/chat/rooms/{roomId}/leave
```

## 3. メッセージAPI

### 3.1 最新100件メッセージ取得

```http
GET /api/v1/chat/rooms/{roomId}/messages?limit=100
```

### 3.2 過去メッセージ取得

```http
GET /api/v1/chat/rooms/{roomId}/messages?beforeMessageId=100&beforeCreatedAt=2026-06-20T10:00:00&limit=100
```

### 3.3 メッセージ削除

```http
DELETE /api/v1/chat/rooms/{roomId}/messages/{messageId}
```

## 4. メッセージレスポンス例

```json
{
  "items": [
    {
      "id": 100,
      "roomId": 1,
      "senderType": "USER",
      "senderUserId": 1,
      "senderName": "User A",
      "originalLanguageCode": "KO",
      "originalText": "오늘 뭐 먹을래?",
      "messageType": "TEXT",
      "status": "SENT",
      "createdAt": "2026-06-20T10:00:00",
      "translations": [
        {
          "languageCode": "JA",
          "translatedText": "今日何食べる？",
          "status": "COMPLETED"
        }
      ]
    }
  ],
  "nextCursor": {
    "beforeMessageId": 1,
    "beforeCreatedAt": "2026-06-19T09:00:00"
  },
  "hasMore": true
}
```
