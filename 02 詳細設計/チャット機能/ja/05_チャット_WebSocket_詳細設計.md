# チャットWebSocket詳細設計

## 1. 目的

本書はチャット機能WebSocket詳細設計基準を整理する。

## 2. Endpoint候補

```text
/ws
```

## 3. Destination候補

```text
/app/chat/rooms/{roomId}/messages
/topic/chat/rooms/{roomId}
```

## 4. イベント候補

- chat.message.created
- chat.translation.pending
- chat.translation.completed
- chat.translation.failed
- chat.error

## 5. 詳細内容

Payload詳細はWebSocket実装段階で補強する。
