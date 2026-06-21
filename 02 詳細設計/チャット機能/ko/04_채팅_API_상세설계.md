# 채팅 API 상세설계

## 1. 문서 목적

본 문서는 TranslaCat 채팅 기능 1차 MVP에서 사용할 REST API 상세설계를 정의한다.

WebSocket 메시지 송수신은 별도 문서에서 정의하며, 본 문서는 채팅방/멤버/메시지 조회 중심의 HTTP API를 다룬다.

---

## 2. 공통 정책

- 인증된 사용자만 호출 가능하다.
- 응답 형식은 기존 TranslaCat 공통 응답 래퍼를 따른다.
- 사용자가 채팅방 멤버가 아닌 경우 조회/수정/전송 관련 API는 거부한다.
- 목록 조회는 cursor 기반 pagination을 우선 사용한다.
- API path prefix는 `/api/v1/chat`을 사용한다.

---

## 3. 채팅방 생성

```text
POST /api/v1/chat/rooms
```

### Request

```json
{
  "roomType": "GROUP",
  "name": "일본어 연습방",
  "description": "일본어로 대화하는 방입니다.",
  "memberUserIds": [2, 3]
}
```

### Response

```json
{
  "id": 1,
  "roomType": "GROUP",
  "name": "일본어 연습방",
  "description": "일본어로 대화하는 방입니다.",
  "myRole": "OWNER",
  "memberCount": 3,
  "createdAt": "2026-06-21T10:00:00"
}
```

### 처리 기준

- 생성자는 OWNER로 등록한다.
- memberUserIds에 생성자 자신이 포함되어 있으면 중복 제거한다.
- GROUP은 2명 이상 멤버를 권장한다.
- DIRECT는 1:1 중복 방 생성 정책을 별도 검토한다.
- 1차에서는 DIRECT/GROUP만 허용한다.

---

## 4. 내 채팅방 목록 조회

```text
GET /api/v1/chat/rooms
```

### Query

```text
?limit=50&cursorRoomId={cursorRoomId}
```

1차에서는 전체 목록 조회로 시작해도 되지만, 추후 pagination을 고려한다.

### Response

```json
{
  "items": [
    {
      "id": 1,
      "roomType": "GROUP",
      "name": "일본어 연습방",
      "description": "일본어로 대화하는 방입니다.",
      "memberCount": 3,
      "lastMessage": "오늘 뭐 먹을래?",
      "lastMessageAt": "2026-06-21T10:05:00",
      "myRole": "OWNER"
    }
  ],
  "hasNext": false,
  "nextCursor": null
}
```

---

## 5. 채팅방 상세 조회

```text
GET /api/v1/chat/rooms/{roomId}
```

### Response

```json
{
  "id": 1,
  "roomType": "GROUP",
  "name": "일본어 연습방",
  "description": "일본어로 대화하는 방입니다.",
  "myRole": "OWNER",
  "memberCount": 3,
  "myLanguageSetting": {
    "originalLanguageCode": "ko",
    "translationLanguageCode": "ja",
    "showOriginal": true,
    "showTranslation": true
  }
}
```

### 권한

- active member만 조회 가능하다.

---

## 6. 채팅방 멤버 목록 조회

```text
GET /api/v1/chat/rooms/{roomId}/members
```

### Response

```json
{
  "items": [
    {
      "userId": 1,
      "name": "User A",
      "profileImageUrl": null,
      "role": "OWNER",
      "translationLanguageCode": "ja",
      "joinedAt": "2026-06-21T10:00:00"
    }
  ]
}
```

---

## 7. 채팅방 멤버 추가

```text
POST /api/v1/chat/rooms/{roomId}/members
```

### Request

```json
{
  "userIds": [4, 5]
}
```

### 처리 기준

- OWNER 또는 ADMIN만 추가 가능하다.
- 이미 active member인 사용자는 중복 추가하지 않는다.
- 과거 퇴실한 사용자는 기존 row restore 또는 신규 생성 정책을 결정한다.

---

## 8. 채팅방 나가기

```text
DELETE /api/v1/chat/rooms/{roomId}/members/me
```

### 처리 기준

- 자기 자신만 퇴실 처리한다.
- OWNER가 나갈 때 정책은 별도 검토가 필요하다.
- 1차에서는 OWNER 퇴실 제한을 우선 고려한다.

---

## 9. 내 언어 설정 변경

```text
PATCH /api/v1/chat/rooms/{roomId}/members/me/language-settings
```

### Request

```json
{
  "originalLanguageCode": "ko",
  "translationLanguageCode": "ja",
  "showOriginal": true,
  "showTranslation": true
}
```

### Response

```json
{
  "roomId": 1,
  "userId": 1,
  "originalLanguageCode": "ko",
  "translationLanguageCode": "ja",
  "showOriginal": true,
  "showTranslation": true
}
```

---

## 10. 메시지 목록 조회

```text
GET /api/v1/chat/rooms/{roomId}/messages
```

### Query

```text
?cursorMessageId={cursorMessageId}&limit=100
```

### Response

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
      "content": "오늘 뭐 먹을래?",
      "status": "SENT",
      "createdAt": "2026-06-21T10:05:00",
      "translations": [
        {
          "languageCode": "ja",
          "status": "COMPLETED",
          "translatedContent": "今日何食べる？"
        }
      ]
    }
  ],
  "hasNext": true,
  "nextCursorMessageId": 80
}
```

### 처리 기준

- 최신 100개 조회를 기본으로 한다.
- cursorMessageId가 있으면 해당 메시지보다 과거 메시지를 조회한다.
- 응답에는 메시지별 번역 상태를 포함한다.

---

## 11. API 에러 후보

| code | 설명 |
|---|---|
| CHAT_ROOM_NOT_FOUND | 채팅방 없음 |
| CHAT_ROOM_ACCESS_DENIED | 채팅방 접근 권한 없음 |
| CHAT_ROOM_MEMBER_NOT_FOUND | 멤버가 아님 |
| CHAT_ROOM_MEMBER_ALREADY_EXISTS | 이미 참여 중 |
| CHAT_MESSAGE_NOT_FOUND | 메시지 없음 |
| CHAT_MESSAGE_CONTENT_EMPTY | 메시지 내용 없음 |
| CHAT_MESSAGE_CONTENT_TOO_LONG | 메시지 길이 초과 |

---

## 12. 1차 API 구현 우선순위

```text
1. POST /api/v1/chat/rooms
2. GET /api/v1/chat/rooms
3. GET /api/v1/chat/rooms/{roomId}
4. GET /api/v1/chat/rooms/{roomId}/members
5. PATCH /api/v1/chat/rooms/{roomId}/members/me/language-settings
6. GET /api/v1/chat/rooms/{roomId}/messages
7. POST /api/v1/chat/rooms/{roomId}/members
8. DELETE /api/v1/chat/rooms/{roomId}/members/me
```
