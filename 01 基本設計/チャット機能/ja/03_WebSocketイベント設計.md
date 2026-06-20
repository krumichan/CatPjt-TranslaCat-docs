# WebSocketイベント設計

## 1. 基本方針

- WebSocketはリアルタイムメッセージ送受信に利用する。
- メッセージ履歴取得はREST APIで処理する。
- 新規メッセージと翻訳完了イベントはWebSocketで配信する。
- 認証方式は今後確定する。

## 2. 接続Endpoint

```text
/ws/chat
```

## 3. Subscribe Topic

```text
/topic/chat/rooms/{roomId}
```

チャットルーム参加者は該当topicを購読する。

## 4. Client Send Endpoint

```text
/app/chat/rooms/{roomId}/messages
```

## 5. Event Types

| Event Type | 方向 | 説明 |
|---|---|---|
| message.created | Server -> Client | 新規メッセージ作成 |
| translation.pending | Server -> Client | 翻訳待機状態作成 |
| translation.completed | Server -> Client | 翻訳完了 |
| translation.failed | Server -> Client | 翻訳失敗 |
| message.deleted | Server -> Client | メッセージ削除 |
| error | Server -> Client | エラー |

## 6. message.created

```json
{
  "eventType": "message.created",
  "roomId": 1,
  "message": {
    "id": 100,
    "senderType": "USER",
    "senderUserId": 1,
    "senderName": "User A",
    "originalLanguageCode": "KO",
    "originalText": "오늘 뭐 먹을래?",
    "createdAt": "2026-06-20T10:00:00"
  }
}
```

## 7. translation.completed

```json
{
  "eventType": "translation.completed",
  "roomId": 1,
  "messageId": 100,
  "translation": {
    "languageCode": "JA",
    "translatedText": "今日何食べる？",
    "status": "COMPLETED"
  }
}
```

## 8. FE処理方針

- `message.created`受信時、メッセージ一覧下部に原文メッセージを追加する。
- 翻訳がまだない場合は `翻訳中...` またはloading状態を表示する。
- `translation.completed`受信時、該当messageIdに翻訳文を追加する。
- 現在ユーザーの表示対象言語ではない翻訳はUIに表示しない。
- WebSocket切断時は再接続を試行する。
