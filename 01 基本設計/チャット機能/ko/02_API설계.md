# 채팅 기능 REST API 설계

## 1. 공통

- 모든 API는 로그인 사용자를 기준으로 처리한다.
- 채팅방 접근 권한이 없는 경우 403을 반환한다.
- 존재하지 않는 채팅방 또는 메시지는 404를 반환한다.

## 2. 채팅방 API

### 2.1 채팅방 생성

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

### 2.2 내 채팅방 목록 조회

```http
GET /api/v1/chat/rooms
```

### 2.3 채팅방 상세 조회

```http
GET /api/v1/chat/rooms/{roomId}
```

### 2.4 채팅방 언어 설정 변경

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

### 2.5 그룹 채팅 멤버 추가

```http
POST /api/v1/chat/rooms/{roomId}/members
```

### 2.6 채팅방 퇴실

```http
POST /api/v1/chat/rooms/{roomId}/leave
```

## 3. 메시지 API

### 3.1 최신 100개 메시지 조회

```http
GET /api/v1/chat/rooms/{roomId}/messages?limit=100
```

### 3.2 과거 메시지 조회

```http
GET /api/v1/chat/rooms/{roomId}/messages?beforeMessageId=100&beforeCreatedAt=2026-06-20T10:00:00&limit=100
```

### 3.3 메시지 삭제

```http
DELETE /api/v1/chat/rooms/{roomId}/messages/{messageId}
```

## 4. 메시지 응답 예시

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
