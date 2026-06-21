# チャットAPI詳細設計

## 1. 目的

本書はチャット機能REST API詳細設計基準を整理する。

## 2. 1次API候補

```text
POST   /api/v1/chat/rooms
GET    /api/v1/chat/rooms
GET    /api/v1/chat/rooms/{roomId}
GET    /api/v1/chat/rooms/{roomId}/members
PATCH  /api/v1/chat/rooms/{roomId}/members/me/language-settings
GET    /api/v1/chat/rooms/{roomId}/messages
```

## 3. 詳細内容

Request/Response DTOはAPI実装段階で補強する。
