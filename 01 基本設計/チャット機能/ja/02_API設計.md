# チャット機能 API設計

## 1. ドキュメント目的

本ドキュメントは、TranslaCatチャット機能 Phase 1 開発に必要なREST APIを定義する。

Phase 1では、REST APIとWebSocketの役割を分離する。

REST APIは、チャットルーム作成、チャットルーム一覧/詳細取得、メンバー取得、メッセージ履歴取得、ユーザー別言語設定取得/変更/初期化、WebSocket障害時の補助メッセージ保存経路を担当する。

リアルタイムメッセージ送受信、新規メッセージ受信、翻訳完了/失敗通知はWebSocketイベントで処理する。

## 2. 共通ルール

### 2.1 Base URL

```txt
/api/v1/chat
```

### 2.2 認証

すべてのAPIはログインユーザーを前提とする。

```txt
Authorization: Bearer {accessToken}
```

### 2.3 共通レスポンス構造

すべてのAPIレスポンスは共通レスポンスラッパーを使用する。

```json
{
  "data": {}
}
```

実際のレスポンス項目は `data` の内部に配置する。

### 2.4 権限基準

* チャットルームアクセス権限は `chat_room_member` を基準に確認する。
* `active = true` かつ `deleted_at is null` のメンバーのみ、アクセス可能なメンバーとみなす。
* チャットルームメンバーではないユーザーは、チャットルーム詳細、メンバー一覧、メッセージ一覧を取得できない。
* チャットルームメンバーではないユーザーはメッセージを保存できない。

### 2.5 Phase 1 API設計基準

* Phase 1のチャットルーム作成リクエストは `memberUserIds` を使用する。
* `memberUserIds` は内部ユーザーIDベースのPhase 1一時構造である。
* Phase 1.5ではpublicIdベースのユーザー検索および招待リクエストへ移行する。
* Phase 1のチャットルーム作成は基本的に `sourceType = MANUAL` として作成する。
* メッセージ一覧取得はcursor方式で処理する。
* offset方式のページネーションは使用しない。
* メッセージ原文保存と翻訳処理は分離する。
* 翻訳結果はメッセージレスポンスの `translations` に含める。

## 3. API一覧

| 区分              | Method | URL                                                    | Phase 1    |
| --------------- | ------ | ------------------------------------------------------ | ---------- |
| チャットルーム作成       | POST   | `/api/v1/chat/rooms`                                   | 対象         |
| 自分のチャットルーム一覧取得  | GET    | `/api/v1/chat/rooms`                                   | 対象         |
| チャットルーム詳細取得     | GET    | `/api/v1/chat/rooms/{chatRoomId}`                      | 対象         |
| チャットルームメンバー一覧取得 | GET    | `/api/v1/chat/rooms/{chatRoomId}/members`              | 対象         |
| 自分の言語設定取得       | GET    | `/api/v1/chat/rooms/{chatRoomId}/members/me/language`  | 対象         |
| 自分の言語設定変更       | PATCH  | `/api/v1/chat/rooms/{chatRoomId}/members/me/language`  | 対象         |
| 自分の言語設定初期化      | DELETE | `/api/v1/chat/rooms/{chatRoomId}/members/me/language`  | 対象         |
| メッセージ一覧取得       | GET    | `/api/v1/chat/rooms/{chatRoomId}/messages`             | 対象         |
| RESTメッセージ保存     | POST   | `/api/v1/chat/rooms/{chatRoomId}/messages`             | 対象         |
| チャットルームメンバー追加   | POST   | `/api/v1/chat/rooms/{chatRoomId}/members`              | Phase 1対象外 |
| チャットルーム退出       | DELETE | `/api/v1/chat/rooms/{chatRoomId}/members/me`           | Phase 1対象外 |
| メッセージ削除         | DELETE | `/api/v1/chat/rooms/{chatRoomId}/messages/{messageId}` | Phase 1対象外 |

## 4. チャットルーム作成

### 4.1 基本情報

```txt
POST /api/v1/chat/rooms
```

1:1チャットルームまたはグループチャットルームを作成する。

Phase 1では `memberUserIds` を使用する。この値は内部ユーザーIDベースの一時構造であり、Phase 1.5でpublicIdベースのリクエストへ移行する。

### 4.2 Request

```json
{
  "roomType": "DIRECT",
  "name": null,
  "description": null,
  "memberUserIds": [2]
}
```

### 4.3 Request fields

| 項目            | 型        | 必須 | 説明                |
| ------------- | -------- | -: | ----------------- |
| roomType      | string   |  Y | DIRECT, GROUP     |
| name          | string   |  N | チャットルーム名。GROUPで使用 |
| description   | string   |  N | チャットルーム説明         |
| memberUserIds | number[] |  Y | 招待対象ユーザーID一覧      |

### 4.4 処理基準

* DIRECT作成時、`memberUserIds` は1名である必要がある。
* GROUP作成時、`memberUserIds` は1名以上である必要がある。
* ログインユーザーは自動的にOWNERメンバーとなる。
* 作成対象ユーザーはMEMBERメンバーとなる。
* Phase 1作成チャットルームは `sourceType = MANUAL` を使用する。
* 同じユーザー組み合わせのDIRECTルームが既に存在し、`sourceType = MANUAL` の場合、既存ルームを返すことができる。
* OPEN roomType作成はPhase 1では許可しない。

### 4.5 Response

```json
{
  "data": {
    "id": 1,
    "roomType": "DIRECT",
    "sourceType": "MANUAL",
    "name": null,
    "description": null,
    "ownerId": 1,
    "active": true,
    "originalLanguageCode": "ko",
    "translationLanguageCode": "ja",
    "roomLanguageSettingApplied": false,
    "memberCount": 2,
    "createdAt": "2026-06-27T10:00:00",
    "updatedAt": "2026-06-27T10:00:00"
  }
}
```

## 5. 自分のチャットルーム一覧取得

### 5.1 基本情報

```txt
GET /api/v1/chat/rooms
```

ログインユーザーが参加中のチャットルーム一覧を取得する。

### 5.2 Response

```json
{
  "data": {
    "chatRooms": [
      {
        "id": 1,
        "roomType": "DIRECT",
        "sourceType": "MANUAL",
        "name": null,
        "description": null,
        "ownerId": 1,
        "memberCount": 2,
        "createdAt": "2026-06-27T10:00:00",
        "updatedAt": "2026-06-27T10:10:00"
      }
    ]
  }
}
```

### 5.3 Response fields

| 項目                      | 型           | 説明                       |
| ----------------------- | ----------- | ------------------------ |
| chatRooms               | array       | チャットルーム一覧                |
| chatRooms[].id          | number      | チャットルームID                |
| chatRooms[].roomType    | string      | DIRECT, GROUP, OPEN      |
| chatRooms[].sourceType  | string      | MANUAL, FRIEND, OPEN, AI |
| chatRooms[].name        | string/null | チャットルーム名                 |
| chatRooms[].description | string/null | チャットルーム説明                |
| chatRooms[].ownerId     | number/null | チャットルーム所有者ID             |
| chatRooms[].memberCount | number      | activeメンバー数              |
| chatRooms[].createdAt   | string      | 作成日時                     |
| chatRooms[].updatedAt   | string      | 更新日時                     |

## 6. チャットルーム詳細取得

### 6.1 基本情報

```txt
GET /api/v1/chat/rooms/{chatRoomId}
```

ログインユーザーが参加中の特定チャットルーム詳細情報を取得する。

### 6.2 Path parameters

| パラメータ      | 型      | 説明        |
| ---------- | ------ | --------- |
| chatRoomId | number | チャットルームID |

### 6.3 Response

```json
{
  "data": {
    "id": 1,
    "roomType": "DIRECT",
    "sourceType": "MANUAL",
    "name": null,
    "description": null,
    "ownerId": 1,
    "active": true,
    "originalLanguageCode": "ko",
    "translationLanguageCode": "ja",
    "roomLanguageSettingApplied": false,
    "memberCount": 2,
    "createdAt": "2026-06-27T10:00:00",
    "updatedAt": "2026-06-27T10:10:00"
  }
}
```

### 6.4 権限条件

* ログインユーザーが対象チャットルームのactiveメンバーである必要がある。
* メンバーではない場合、アクセスを制御する。

## 7. チャットルームメンバー一覧取得

### 7.1 基本情報

```txt
GET /api/v1/chat/rooms/{chatRoomId}/members
```

チャットルームのactiveメンバー一覧を取得する。

### 7.2 Response

```json
{
  "data": {
    "members": [
      {
        "id": 1,
        "chatRoomId": 1,
        "userId": 1,
        "name": "ユーザーA",
        "email": "user-a@example.com",
        "role": "OWNER",
        "active": true,
        "joinedAt": "2026-06-27T10:00:00",
        "leftAt": null
      }
    ]
  }
}
```

### 7.3 権限条件

* ログインユーザーが対象チャットルームのactiveメンバーである必要がある。

## 8. 自分のチャットルーム言語設定取得

### 8.1 基本情報

```txt
GET /api/v1/chat/rooms/{chatRoomId}/members/me/language
```

ログインユーザーの対象チャットルーム言語設定を取得する。

### 8.2 Response

```json
{
  "data": {
    "chatRoomId": 1,
    "userId": 1,
    "originalLanguageCode": "ko",
    "translationLanguageCode": "ja",
    "showOriginal": true,
    "showTranslation": true,
    "roomLanguageSettingApplied": false
  }
}
```

## 9. 自分のチャットルーム言語設定変更

### 9.1 基本情報

```txt
PATCH /api/v1/chat/rooms/{chatRoomId}/members/me/language
```

ログインユーザーの対象チャットルーム限定言語設定を変更する。

### 9.2 Request

```json
{
  "originalLanguageCode": "ko",
  "translationLanguageCode": "ja",
  "showOriginal": true,
  "showTranslation": true
}
```

### 9.3 Request fields

| 項目                      | 型       | 必須 | 説明     |
| ----------------------- | ------- | -: | ------ |
| originalLanguageCode    | string  |  Y | 原文基準言語 |
| translationLanguageCode | string  |  Y | 受信翻訳言語 |
| showOriginal            | boolean |  Y | 原文表示有無 |
| showTranslation         | boolean |  Y | 翻訳表示有無 |

### 9.4 Response

```json
{
  "data": {
    "chatRoomId": 1,
    "userId": 1,
    "originalLanguageCode": "ko",
    "translationLanguageCode": "ja",
    "showOriginal": true,
    "showTranslation": true,
    "roomLanguageSettingApplied": true
  }
}
```

## 10. 自分のチャットルーム言語設定初期化

### 10.1 基本情報

```txt
DELETE /api/v1/chat/rooms/{chatRoomId}/members/me/language
```

ログインユーザーの対象チャットルーム言語設定を初期化し、基本言語設定に戻す。

### 10.2 Response

```json
{
  "data": {
    "chatRoomId": 1,
    "userId": 1,
    "originalLanguageCode": "ko",
    "translationLanguageCode": "ja",
    "showOriginal": true,
    "showTranslation": true,
    "roomLanguageSettingApplied": false
  }
}
```

## 11. メッセージ一覧取得

### 11.1 基本情報

```txt
GET /api/v1/chat/rooms/{chatRoomId}/messages
```

チャットルームの最新メッセージまたはcursorId基準の過去メッセージを取得する。

### 11.2 Query parameters

| パラメータ    | 型      | 必須 | 説明                 |
| -------- | ------ | -: | ------------------ |
| cursorId | number |  N | 過去メッセージ取得基準メッセージID |

### 11.3 Request例

```txt
GET /api/v1/chat/rooms/1/messages
GET /api/v1/chat/rooms/1/messages?cursorId=100
```

### 11.4 Response

```json
{
  "data": {
    "messages": [
      {
        "id": 101,
        "chatRoomId": 1,
        "senderUserId": 1,
        "senderName": "ユーザーA",
        "senderEmail": "user-a@example.com",
        "senderType": "USER",
        "messageType": "TEXT",
        "content": "こんにちは",
        "status": "SENT",
        "translations": [
          {
            "id": 1001,
            "languageCode": "ko",
            "translatedContent": "안녕하세요",
            "status": "COMPLETED",
            "failureReason": null,
            "completedAt": "2026-06-27T10:10:05"
          }
        ],
        "createdAt": "2026-06-27T10:10:00",
        "updatedAt": "2026-06-27T10:10:00"
      }
    ],
    "nextCursorId": 101,
    "hasNext": true
  }
}
```

### 11.5 処理基準

* `cursorId` がない場合、最新メッセージ一覧を取得する。
* `cursorId` がある場合、対象IDより前のメッセージを取得する。
* レスポンスには次回取得で利用する `nextCursorId` を含める。
* さらに取得できるメッセージが存在する場合、`hasNext = true` を返す。
* 翻訳結果は `translations` 配列に含める。
* 翻訳が存在しない場合は `translations = []` を返す。

## 12. RESTベースメッセージ保存

### 12.1 基本情報

```txt
POST /api/v1/chat/rooms/{chatRoomId}/messages
```

チャットルームにテキストメッセージ原文を保存する。

Phase 1の基本メッセージ送信経路はWebSocketだが、RESTメッセージ保存APIはWebSocket実装前テストおよびWebSocket障害時のfallback経路として利用できる。

### 12.2 Request

```json
{
  "content": "こんにちは"
}
```

### 12.3 Response

```json
{
  "data": {
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

### 12.4 処理基準

* ログインユーザーがチャットルームactiveメンバーである必要がある。
* メッセージ原文を先に保存する。
* 翻訳処理はメッセージ保存と分離する。
* 翻訳リクエスト/結果反映は非同期で処理する。
* WebSocketが利用可能な場合、FEはWebSocket送信を優先する。

## 13. Phase 1 対象外API

### 13.1 チャットルームメンバー追加

```txt
POST /api/v1/chat/rooms/{chatRoomId}/members
```

Phase 1では対象外とする。

Phase 1.5で友達一覧ベースのグループ招待またはpublicIdベースの招待リクエストとして再設計する。

### 13.2 チャットルーム退出

```txt
DELETE /api/v1/chat/rooms/{chatRoomId}/members/me
```

Phase 1では対象外とする。

Phase 1.5で友達グループチャットおよび招待承認/拒否ポリシーとあわせて再検討する。

### 13.3 メッセージ削除

```txt
DELETE /api/v1/chat/rooms/{chatRoomId}/messages/{messageId}
```

Phase 1では対象外とする。

`ChatMessageStatus = DELETED` 構造は存在するが、ユーザー削除APIはPhase 1.5またはPhase 2で再検討する。

## 14. REST APIとWebSocketの役割区分

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

## 15. 備考

Phase 1のAPI設計は、基本チャットMVPを完成させるための最小範囲として定義する。

友達追加、友達一覧、publicIdベースのユーザー検索、グループ招待、チャットルーム退出、メッセージ削除、オープンチャット、AI参加者関連APIはPhase 1.5またはPhase 2で別途設計する。
