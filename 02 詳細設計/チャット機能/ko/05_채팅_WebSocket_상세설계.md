# 채팅 WebSocket 상세설계

## 1. 문서 목적

본 문서는 TranslaCat 채팅 기능 1차 MVP에서 사용할 WebSocket/STOMP 이벤트 상세설계를 정의한다.

1차 목표는 원문 메시지를 실시간으로 송수신하고, 비동기 번역 상태와 결과를 같은 채팅방 구독자에게 전달하는 것이다.

---

## 2. 기본 구조

### 2.1 WebSocket endpoint

```text
/ws
```

### 2.2 Client send destination

```text
/app/chat/rooms/{roomId}/messages
```

### 2.3 Client subscribe destination

```text
/topic/chat/rooms/{roomId}
```

---

## 3. Client → Server 이벤트

### 3.1 chat.message.send

사용자가 메시지를 전송할 때 사용한다.

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
  "content": "오늘 뭐 먹을래?"
}
```

### 처리 기준

- `content`는 빈 값일 수 없다.
- `messageType`은 1차에서 `TEXT`만 허용한다.
- `clientMessageId`는 FE 중복 방지 또는 optimistic UI용으로 사용할 수 있다.
- 서버 저장 후 server message id를 발급한다.

---

## 4. Server → Client 이벤트

### 4.1 chat.message.created

원문 메시지가 저장된 직후 발행한다.

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
    "content": "오늘 뭐 먹을래?",
    "status": "SENT",
    "createdAt": "2026-06-21T10:05:00",
    "translations": []
  }
}
```

---

### 4.2 chat.translation.pending

번역 대기 상태가 생성되었을 때 발행한다.

```json
{
  "eventType": "chat.translation.pending",
  "roomId": 1,
  "messageId": 100,
  "translations": [
    {
      "languageCode": "ja",
      "status": "PENDING"
    }
  ]
}
```

---

### 4.3 chat.translation.completed

번역이 완료되었을 때 발행한다.

```json
{
  "eventType": "chat.translation.completed",
  "roomId": 1,
  "messageId": 100,
  "translation": {
    "languageCode": "ja",
    "status": "COMPLETED",
    "translatedContent": "今日何食べる？",
    "completedAt": "2026-06-21T10:05:03"
  }
}
```

---

### 4.4 chat.translation.failed

번역이 실패했을 때 발행한다.

```json
{
  "eventType": "chat.translation.failed",
  "roomId": 1,
  "messageId": 100,
  "translation": {
    "languageCode": "ja",
    "status": "FAILED",
    "failureReason": "AI_TRANSLATION_FAILED"
  }
}
```

---

### 4.5 chat.error

WebSocket 처리 중 사용자에게 전달 가능한 오류가 발생했을 때 발행한다.

```json
{
  "eventType": "chat.error",
  "roomId": 1,
  "code": "CHAT_ROOM_ACCESS_DENIED",
  "message": "채팅방 접근 권한이 없습니다."
}
```

---

## 5. 인증/권한

- WebSocket 연결 시 기존 JWT 인증 정보를 사용한다.
- 메시지 전송 시 로그인 사용자 ID를 서버에서 식별한다.
- payload의 senderUserId는 신뢰하지 않는다.
- 채팅방 active member만 메시지를 전송할 수 있다.
- 채팅방 active member만 해당 room topic을 구독할 수 있도록 검토한다.

1차에서 topic subscribe 권한 검증이 어려운 경우, 메시지 전송/조회 권한 검증을 우선하고 subscribe 제한은 후속 보강할 수 있다.

---

## 6. 중복 처리

FE는 다음 기준으로 중복 메시지 표시를 방지한다.

```text
message.id
```

optimistic UI를 사용할 경우 다음 값도 사용한다.

```text
clientMessageId
```

1차 초기 구현에서는 optimistic UI 없이 서버의 `chat.message.created` 이벤트를 기준으로 메시지를 추가하는 방식을 우선한다.

---

## 7. 이벤트 발행 시점

권장 시점:

```text
DB transaction commit 성공 후 이벤트 발행
```

이유:

- 이벤트는 발행되었지만 DB 저장이 rollback되는 상황을 방지한다.
- 새로고침 후 메시지가 사라지는 문제를 방지한다.

구현 후보:

```text
@TransactionalEventListener(phase = AFTER_COMMIT)
```

---

## 8. 1차 구현 우선순위

```text
1. WebSocket endpoint 설정
2. STOMP send/subscribe 설정
3. chat.message.send 수신
4. 권한 검증
5. 메시지 저장
6. chat.message.created 발행
7. chat.translation.pending 발행
8. chat.translation.completed/failed 발행
9. FE 연동 확인
```
