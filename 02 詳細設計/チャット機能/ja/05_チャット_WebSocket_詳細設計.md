# チャット WebSocket 詳細設計

## 1. 文書目的

本書は、TranslaCatチャット機能1次MVPで使用するWebSocket/STOMPイベント詳細設計を定義する。

1次目標は、原文メッセージをリアルタイムに送受信し、非同期翻訳状態と結果を同じチャットルーム購読者に配信することである。

---

## 2. 基本構造

WebSocket endpoint:

```text
/ws
```

Client send destination:

```text
/app/chat/rooms/{roomId}/messages
```

Client subscribe destination:

```text
/topic/chat/rooms/{roomId}
```

---

## 3. Client → Serverイベント

### 3.1 chat.message.send

Destination:

```text
/app/chat/rooms/{roomId}/messages
```

Payload:

```json
{
  "eventType": "chat.message.send",
  "clientMessageId": "temp-uuid-001",
  "messageType": "TEXT",
  "content": "今日何食べる？"
}
```

処理基準:

- contentは空不可。
- messageTypeは1次ではTEXTのみ許可する。
- senderUserIdはpayloadではなくサーバー認証情報から取得する。

---

## 4. Server → Clientイベント

### 4.1 chat.message.created

原文メッセージ保存直後に配信する。

```json
{
  "eventType": "chat.message.created",
  "roomId": 1,
  "clientMessageId": "temp-uuid-001",
  "message": {
    "id": 100,
    "senderType": "USER",
    "senderUserId": 1,
    "senderName": "User A",
    "messageType": "TEXT",
    "content": "今日何食べる？",
    "status": "SENT",
    "createdAt": "2026-06-21T10:05:00",
    "translations": []
  }
}
```

### 4.2 chat.translation.pending

```json
{
  "eventType": "chat.translation.pending",
  "roomId": 1,
  "messageId": 100,
  "translations": [
    {
      "languageCode": "ko",
      "status": "PENDING"
    }
  ]
}
```

### 4.3 chat.translation.completed

```json
{
  "eventType": "chat.translation.completed",
  "roomId": 1,
  "messageId": 100,
  "translation": {
    "languageCode": "ko",
    "status": "COMPLETED",
    "translatedContent": "오늘 뭐 먹을래?",
    "completedAt": "2026-06-21T10:05:03"
  }
}
```

### 4.4 chat.translation.failed

```json
{
  "eventType": "chat.translation.failed",
  "roomId": 1,
  "messageId": 100,
  "translation": {
    "languageCode": "ko",
    "status": "FAILED",
    "failureReason": "AI_TRANSLATION_FAILED"
  }
}
```

### 4.5 chat.error

```json
{
  "eventType": "chat.error",
  "roomId": 1,
  "code": "CHAT_ROOM_ACCESS_DENIED",
  "message": "チャットルームへのアクセス権限がありません。"
}
```

---

## 5. 認証/権限

- WebSocket接続時、既存JWT認証情報を使用する。
- メッセージ送信時、ログインユーザーIDをサーバー側で識別する。
- payloadのsenderUserIdは信頼しない。
- active memberのみメッセージ送信可能とする。

---

## 6. 重複処理

FEは以下を基準に重複表示を防止する。

```text
message.id
```

optimistic UIを使用する場合は以下も利用する。

```text
clientMessageId
```

1次初期実装ではoptimistic UIなしで、サーバーの `chat.message.created` イベントを基準に追加する。

---

## 7. イベント配信タイミング

推奨:

```text
DB transaction commit成功後にイベント配信
```

候補:

```text
@TransactionalEventListener(phase = AFTER_COMMIT)
```
