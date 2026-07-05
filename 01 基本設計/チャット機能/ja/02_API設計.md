# チャット機能 REST API設計

## 1. 目的

Phase 1基本チャットAPIとPhase 1.5プロフィール/友達/ブロック/友達チャットAPIを統合整理する。

## 2. 共通規則

Base URL:

```text
/api/v1
```

公開API以外はJWTログインユーザーを基準として処理する。

共通レスポンス例:

```json
{
  "resultCode": 200,
  "message": "success",
  "body": {},
  "guid": null,
  "createDate": "2026-07-05 18:00:00"
}
```

実データは `body` に含める。

## 3. API一覧

### プロフィール/ユーザー
- `GET /api/v1/users/me/profile`
- `PATCH /api/v1/users/me/profile`
- `GET /api/v1/users/search?publicId={publicId}`

### 友達リクエスト
- `POST /api/v1/friend-requests`
- `GET /api/v1/friend-requests/received`
- `GET /api/v1/friend-requests/sent`
- `PATCH /api/v1/friend-requests/{requestId}/accept`
- `PATCH /api/v1/friend-requests/{requestId}/reject`
- `PATCH /api/v1/friend-requests/{requestId}/cancel`

### 友達
- `GET /api/v1/friends`
- `DELETE /api/v1/friends/{friendUserId}`

### ブロック
- `POST /api/v1/blocks`
- `GET /api/v1/blocks`
- `DELETE /api/v1/blocks/{blockedUserId}`

### 友達チャット
- `POST /api/v1/chat/friends/{friendUserId}/direct-room`
- `POST /api/v1/chat/friends/group-rooms`

### 既存チャット
- `POST /api/v1/chat/rooms`
- `GET /api/v1/chat/rooms`
- `GET /api/v1/chat/rooms/{roomId}`
- `GET /api/v1/chat/rooms/{roomId}/messages`
- `POST /api/v1/chat/rooms/{roomId}/messages`

メッセージ/WebSocket/翻訳/言語設定の詳細フローはPhase 1設計を維持する。

## 4. 自分のプロフィール取得

```http
GET /api/v1/users/me/profile
```

例:

```json
{
  "resultCode": 200,
  "message": "success",
  "body": {
    "userId": 1,
    "publicId": "TC-H6VR-9KQD",
    "nickname": "krumi",
    "profileImageUrl": null,
    "bio": "こんにちは"
  },
  "guid": null,
  "createDate": "2026-07-05 18:00:00"
}
```

## 5. 自分のプロフィール修正

```http
PATCH /api/v1/users/me/profile
```

Request:

```json
{
  "nickname": "krumi",
  "profileImageUrl": "https://example.com/profile.png",
  "bio": "こんにちは"
}
```

方針:
- publicIdは修正不可
- nickname validation適用
- profileImageUrl/bioは方針範囲でnull可
- ファイルアップロードは後続Issue

## 6. publicIdユーザー検索

```http
GET /api/v1/users/search?publicId=TC-PFSN-CLNA
```

代表情報:
- userId
- publicId
- nickname
- profileImageUrl
- 友達関係状態
- ブロック関連アクション可否

方針:
- 自分自身へのリクエスト防止
- 既に友達の場合は重複防止
- PENDING重複防止
- ブロック関係でのリクエスト制限

## 7. 友達リクエスト送信

```http
POST /api/v1/friend-requests
```

```json
{
  "receiverPublicId": "TC-PFSN-CLNA"
}
```

条件:
- 対象存在
- 自分自身ではない
- 既に友達ではない
- 同一PENDINGなし
- 双方向ブロック関係なし

成功時は `PENDING`。

## 8. 受信/送信リクエスト

```http
GET /api/v1/friend-requests/received
GET /api/v1/friend-requests/sent
```

FE通知センター:
- 受信: 承認/拒否
- 送信: キャンセル

## 9. リクエスト状態変更

承認:

```http
PATCH /api/v1/friend-requests/{requestId}/accept
```

- 受信者のみ
- PENDINGのみ
- 成功時に友達関係作成
- 重複関係作成禁止

拒否:

```http
PATCH /api/v1/friend-requests/{requestId}/reject
```

- 受信者のみ
- PENDINGのみ
- `REJECTED`へ変更

キャンセル:

```http
PATCH /api/v1/friend-requests/{requestId}/cancel
```

- 送信者のみ
- PENDINGのみ
- `CANCELED`へ変更

## 10. 友達一覧/削除

```http
GET /api/v1/friends
```

友達カード代表情報:
- friend userId
- publicId
- nickname
- profileImageUrl
- 友達関係情報

利用箇所:
- 一覧表示
- 検索フィルター
- FRIEND DIRECT
- FRIEND GROUPメンバー選択

削除:

```http
DELETE /api/v1/friends/{friendUserId}
```

- 友達関係削除
- 既存ルーム/メッセージは削除しない
- 新規友達チャット開始可否は現在関係で検証

## 11. ブロック

ブロック:

```http
POST /api/v1/blocks
```

```json
{
  "blockedPublicId": "TC-3567-W4EZ"
}
```

方針:
- 自分自身不可
- 重複不可
- 友達リクエスト制限
- FRIEND DIRECT新規開始制限
- FRIEND GROUP選択/作成制限
- ブロック後に友達一覧から除外

一覧:

```http
GET /api/v1/blocks
```

解除:

```http
DELETE /api/v1/blocks/{blockedUserId}
```

解除後の以前の友達関係は自動復旧しないことを基本とする。

## 12. FRIEND DIRECT

```http
POST /api/v1/chat/friends/{friendUserId}/direct-room
```

Request bodyなし。

処理:
1. 認証ユーザー確認
2. 対象ユーザー確認
3. 友達関係確認
4. 双方向ブロック確認
5. 同一ユーザー組み合わせの `DIRECT + FRIEND` 検索
6. 存在すれば既存ルーム返却
7. なければ新規作成
8. 両ユーザーをメンバー登録
9. ルームレスポンス返却

方針:
- `roomType = DIRECT`
- `sourceType = FRIEND`
- MANUAL DIRECTは再利用しない

代表レスポンス:

```json
{
  "resultCode": 200,
  "message": "success",
  "body": {
    "id": 10,
    "roomType": "DIRECT",
    "sourceType": "FRIEND",
    "name": null,
    "description": null,
    "memberCount": 2,
    "active": true
  },
  "guid": null,
  "createDate": "2026-07-05 18:00:00"
}
```

## 13. FRIEND GROUP

```http
POST /api/v1/chat/friends/group-rooms
```

Request:

```json
{
  "name": "週末日本語勉強ルーム",
  "description": "友達と日本語を勉強するルーム",
  "memberUserIds": [2, 3]
}
```

処理:
1. 認証ユーザー確認
2. メンバー一覧重複/有効性検証
3. 全員が現在友達か確認
4. 双方向ブロック確認
5. `GROUP + FRIEND`作成
6. 作成者をOWNERとして追加
7. 選択友達をMEMBERとして追加
8. ルームレスポンス返却

方針:
- memberUserIdsに自分自身を含めない
- 全対象は現在友達
- ブロック関係ユーザーを含めない
- Phase 1.5では同一メンバー組み合わせのグループ重複作成を許可

## 14. 既存MANUAL API

```http
POST /api/v1/chat/rooms
```

維持方針:
- API/機能を削除しない
- 既存MANUALルーム一覧/詳細/メッセージ維持
- 一般ユーザーUIでは内部userId直接入力作成フロー非公開
- 通常入口は友達一覧ベースAPI

## 15. FE状態/エラー処理

区分:
- Loading
- Empty
- Validation Error
- API Error
- Retry可能状態
- 認証期限切れ
- 関係/権限検証失敗

チャットルーム移動後、ルーム詳細、メッセージ、言語設定、WebSocketフローが正常接続されること。

## 16. Phase 2拡張

既存チャット構造を維持し、sourceTypeと方針を拡張する。

- OPEN
- AI
- 仮想プロフィール
- AIメンバー
- 公開ルーム検索
- 運営/通報機能
