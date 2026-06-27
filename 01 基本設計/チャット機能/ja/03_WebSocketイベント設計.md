# チャット機能 WebSocketイベント設計

## 1. ドキュメント目的

本ドキュメントは、TranslaCatチャット機能 Phase 1 開発に必要なWebSocketイベント構造を定義する。

Phase 1では、WebSocketをリアルタイムメッセージ送受信、新規メッセージ受信、翻訳完了/失敗通知に利用する。

メッセージ履歴取得、チャットルーム作成、チャットルーム一覧/詳細取得、メンバー一覧取得、言語設定取得/変更はREST APIで処理する。

## 2. 基本方針

* WebSocketはリアルタイム性が必要な処理にのみ利用する。
* メッセージ履歴取得はREST APIで処理する。
* メッセージ原文保存後、即時に `chat.message.created` イベントを発行する。
* 翻訳処理は非同期で行う。
* 翻訳成功時は `chat.translation.completed` イベントを発行する。
* 翻訳失敗時は `chat.translation.failed` イベントを発行する。
* WebSocket接続が切断または再接続された場合、不足している可能性のあるメッセージはREST APIで再取得する。
* チャットルームメンバーではないユーザーは、対象チャットルームtopicを購読したりメッセージを送信したりできない。
* Phase 1では `chat.translation.pending` を別イベントとして発行しない。
* PENDING状態はメッセージ取得レスポンスまたはメッセージpayloadの `translations.status` で表現する。

## 3. 接続Endpoint

```txt
/ws/chat
```

クライアントは上記endpointへWebSocket/STOMP接続を行う。

## 4. 認証方式

WebSocket接続時、ログインユーザーのaccess tokenを渡す。

推奨方式はSTOMP CONNECT headerにaccess tokenを含める方式である。

```txt
Authorization: Bearer {accessToken}
```

BEはWebSocket接続またはメッセージ処理時にaccess tokenを検証し、ログインユーザーを識別する。

## 5. Subscribe Destination

```txt
/topic/chat/rooms/{chatRoomId}
```

チャットルーム参加者は対象topicを購読する。

### 権限条件

* ログインユーザーが対象チャットルームのactiveメンバーである必要がある。
* `chat_room_member.active = true`
* `chat_room_member.deleted_at is null`
* 条件を満たさないユーザーは購読したりメッセージを受信したりできない。

## 6. Client Send Destination

```txt
/app/chat/rooms/{chatRoomId}/messages
```

クライアントはメッセージ送信時、上記destinationへメッセージをpublishする。

### 権限条件

* ログインユーザーが対象チャットルームのactiveメンバーである必要がある。
* メンバーではないユーザーのメッセージ送信は拒否する。

## 7. イベント一覧

### 7.1 Client → Server

| eventType           | Destination                             | 説明                      |
| ------------------- | --------------------------------------- | ----------------------- |
| `chat.message.send` | `/app/chat/rooms/{chatRoomId}/messages` | ユーザーがチャットルームへメッセージを送信する |

### 7.2 Server → Client

| eventType                    | Destination                      | 説明                        |
| ---------------------------- | -------------------------------- | ------------------------- |
| `chat.message.created`       | `/topic/chat/rooms/{chatRoomId}` | メッセージ原文保存完了後、新規メッセージを通知する |
| `chat.translation.completed` | `/topic/chat/rooms/{chatRoomId}` | メッセージ翻訳完了後、翻訳結果を通知する      |
| `chat.translation.failed`    | `/topic/chat/rooms/{chatRoomId}` | メッセージ翻訳失敗後、失敗状態を通知する      |
| `chat.error`                 | user queue または room topic        | WebSocket処理中に発生したエラーを通知する |

## 8. Phase 1対象外 / 保留イベント

| eventType                  | 理由                                                              |
| -------------------------- | --------------------------------------------------------------- |
| `chat.translation.pending` | Phase 1では別イベントとして発行しない。PENDING状態はメッセージレスポンスのtranslations状態で表現する |
| `chat.message.deleted`     | Phase 1ではメッセージ削除APIが対象外である                                      |
| `chat.member.joined`       | Phase 1.5の友達グループ招待/参加ポリシーで再検討する                                 |
| `chat.member.left`         | Phase 1.5の退出ポリシーで再検討する                                          |
| `chat.typing`              | Phase 2以降で検討する                                                  |
| `chat.read`                | Phase 2以降の既読/未読機能で検討する                                          |

## 9. Server → Client 共通Envelope

Server → Clientイベントは以下の構造を基本とする。

```json
{
  "eventType": "chat.message.created",
  "roomId": 1,
  "payload": {}
}
```

| 項目        | 型      | 説明             |
| --------- | ------ | -------------- |
| eventType | string | イベントタイプ        |
| roomId    | number | チャットルームID      |
| payload   | object | イベントごとのpayload |

## 10. chat.message.send

### 10.1 方向

```txt
Client → Server
```

### 10.2 Destination

```txt
/app/chat/rooms/{chatRoomId}/messages
```

### 10.3 Payload

```json
{
  "content": "こんにちは"
}
```

### 10.4 Payload fields

| 項目      | 型      | 必須 | 説明      |
| ------- | ------ | -: | ------- |
| content | string |  Y | メッセージ原文 |

### 10.5 処理基準

* ログインユーザーがチャットルームactiveメンバーであることを確認する。
* メッセージ原文をDBへ保存する。
* 保存されたメッセージを `chat.message.created` イベントとして発行する。
* 翻訳処理はメッセージ保存後に非同期でリクエストする。

## 11. chat.message.created

### 11.1 方向

```txt
Server → Client
```

### 11.2 Destination

```txt
/topic/chat/rooms/{chatRoomId}
```

### 11.3 Payload

```json
{
  "eventType": "chat.message.created",
  "roomId": 1,
  "payload": {
    "id": 101,
    "chatRoomId": 1,
    "senderUserId": 1,
    "senderName": "ユーザーA",
    "senderEmail": "user-a@example.com",
    "senderType": "USER",
    "messageType": "TEXT",
    "content": "こんにちは",
    "status": "SENT",
    "translations": [],
    "createdAt": "2026-06-27T10:10:00",
    "updatedAt": "2026-06-27T10:10:00"
  }
}
```

### 11.4 Payload fields

| 項目           | 型           | 説明               |
| ------------ | ----------- | ---------------- |
| id           | number      | メッセージID          |
| chatRoomId   | number      | チャットルームID        |
| senderUserId | number/null | 送信ユーザーID         |
| senderName   | string/null | 送信者名             |
| senderEmail  | string/null | 送信者メール           |
| senderType   | string      | USER, AI, SYSTEM |
| messageType  | string      | TEXT, SYSTEM     |
| content      | string      | メッセージ原文          |
| status       | string      | SENT, DELETED    |
| translations | array       | メッセージ翻訳一覧        |
| createdAt    | string      | 作成日時             |
| updatedAt    | string      | 更新日時             |

### 11.5 FE処理

* 受信したメッセージをメッセージ一覧下部に追加する。
* 同一 `id` のメッセージが既に存在する場合は重複追加しない。
* 翻訳がまだ存在しない場合は原文のみ表示する。
* `translations` にPENDING状態が含まれる場合は翻訳待機UIを表示できる。

## 12. chat.translation.completed

### 12.1 方向

```txt
Server → Client
```

### 12.2 Destination

```txt
/topic/chat/rooms/{chatRoomId}
```

### 12.3 Payload

```json
{
  "eventType": "chat.translation.completed",
  "roomId": 1,
  "payload": {
    "messageId": 101,
    "languageCode": "ko",
    "translatedContent": "안녕하세요",
    "status": "COMPLETED",
    "failureReason": null,
    "completedAt": "2026-06-27T10:10:05",
    "translation": {
      "id": 1001,
      "languageCode": "ko",
      "translatedContent": "안녕하세요",
      "status": "COMPLETED",
      "failureReason": null,
      "completedAt": "2026-06-27T10:10:05"
    }
  }
}
```

### 12.4 Payload fields

| 項目                | 型           | 説明            |
| ----------------- | ----------- | ------------- |
| messageId         | number      | 翻訳対象メッセージID   |
| languageCode      | string      | 翻訳言語コード       |
| translatedContent | string      | 翻訳文           |
| status            | string      | COMPLETED     |
| failureReason     | string/null | 失敗理由。成功時はnull |
| completedAt       | string      | 翻訳完了日時        |
| translation       | object      | 翻訳結果オブジェクト    |

### 12.5 FE処理

* `messageId` に該当するメッセージを探す。
* `languageCode` を基準に既存翻訳とマージする。
* 同じ言語の翻訳が既に存在する場合は最新結果で更新する。
* 現在ユーザーの言語設定に合う翻訳のみ画面に表示する。
* 重複翻訳を画面に表示しない。

## 13. chat.translation.failed

### 13.1 方向

```txt
Server → Client
```

### 13.2 Destination

```txt
/topic/chat/rooms/{chatRoomId}
```

### 13.3 Payload

```json
{
  "eventType": "chat.translation.failed",
  "roomId": 1,
  "payload": {
    "messageId": 101,
    "languageCode": "ko",
    "translatedContent": null,
    "status": "FAILED",
    "failureReason": "TRANSLATION_TIMEOUT",
    "completedAt": null,
    "translation": {
      "id": 1001,
      "languageCode": "ko",
      "translatedContent": null,
      "status": "FAILED",
      "failureReason": "TRANSLATION_TIMEOUT",
      "completedAt": null
    }
  }
}
```

### 13.4 Payload fields

| 項目                | 型           | 説明           |
| ----------------- | ----------- | ------------ |
| messageId         | number      | 翻訳対象メッセージID  |
| languageCode      | string      | 翻訳言語コード      |
| translatedContent | string/null | 失敗時はnull     |
| status            | string      | FAILED       |
| failureReason     | string/null | 失敗理由         |
| completedAt       | string/null | 失敗時はnull     |
| translation       | object      | 翻訳失敗状態オブジェクト |

### 13.5 FE処理

* `messageId` に該当するメッセージを探す。
* `languageCode` を基準に既存翻訳状態をFAILEDへ更新する。
* 翻訳失敗UIを表示する。
* 原文メッセージは維持する。

## 14. chat.error

### 14.1 方向

```txt
Server → Client
```

### 14.2 Payload

```json
{
  "eventType": "chat.error",
  "roomId": 1,
  "payload": {
    "code": "CHAT_ROOM_ACCESS_DENIED",
    "message": "チャットルームへのアクセス権限がありません。",
    "occurredAt": "2026-06-27T10:10:00"
  }
}
```

### 14.3 Payload fields

| 項目         | 型      | 説明       |
| ---------- | ------ | -------- |
| code       | string | エラーコード   |
| message    | string | エラーメッセージ |
| occurredAt | string | 発生日時     |

### 14.4 主要エラーコード

| コード                         | 説明              |
| --------------------------- | --------------- |
| CHAT_ROOM_ACCESS_DENIED     | チャットルームアクセス権限なし |
| CHAT_ROOM_NOT_FOUND         | チャットルームなし       |
| CHAT_MESSAGE_EMPTY          | メッセージ内容なし       |
| CHAT_MESSAGE_SEND_FAILED    | メッセージ送信失敗       |
| CHAT_WEBSOCKET_UNAUTHORIZED | WebSocket認証失敗   |

## 15. 再接続および同期

WebSocket接続が切断された後に再接続された場合、FEはREST APIで最新メッセージを再取得する。

```txt
GET /api/v1/chat/rooms/{chatRoomId}/messages
```

### 処理基準

* WebSocket再接続成功後、最新メッセージ一覧をRESTで取得する。
* 既存メッセージ一覧と `id` を基準にマージする。
* 既に存在するメッセージは重複追加しない。
* 不足していたメッセージや翻訳結果を反映する。
* WebSocketは履歴取得を担当しない。

## 16. REST APIとWebSocketの役割区分

| 処理              | REST API | WebSocket |
| --------------- | -------: | --------: |
| チャットルーム作成       |        Y |         N |
| チャットルーム一覧取得     |        Y |         N |
| チャットルーム詳細取得     |        Y |         N |
| チャットルームメンバー一覧取得 |        Y |         N |
| メッセージ初期/過去取得    |        Y |         N |
| メッセージリアルタイム送信   | fallback |         Y |
| メッセージリアルタイム受信   |        N |         Y |
| 翻訳完了通知          |        N |         Y |
| 翻訳失敗通知          |        N |         Y |
| 言語設定取得/変更       |        Y |         N |

## 17. 備考

Phase 1では、基本メッセージ送受信と翻訳完了/失敗反映に集中する。

typing indicator、read receipt、member joined/left、message deletedイベントはPhase 1.5またはPhase 2で別途設計する。
