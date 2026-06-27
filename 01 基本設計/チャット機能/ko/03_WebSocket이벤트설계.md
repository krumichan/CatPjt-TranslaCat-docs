# 채팅 기능 WebSocket 이벤트 설계

## 1. 문서 목적

본 문서는 TranslaCat 채팅 기능 Phase 1 개발에 필요한 WebSocket 이벤트 구조를 정의한다.

Phase 1에서는 WebSocket을 실시간 메시지 송수신, 신규 메시지 수신, 번역 완료/실패 알림에 사용한다.

메시지 이력 조회, 채팅방 생성, 채팅방 목록/상세 조회, 멤버 목록 조회, 언어 설정 조회/변경은 REST API로 처리한다.

## 2. 기본 방침

* WebSocket은 실시간성이 필요한 처리에만 사용한다.
* 메시지 이력 조회는 REST API로 처리한다.
* 메시지 원문 저장 후 즉시 `chat.message.created` 이벤트를 발행한다.
* 번역 처리는 비동기로 수행한다.
* 번역 성공 시 `chat.translation.completed` 이벤트를 발행한다.
* 번역 실패 시 `chat.translation.failed` 이벤트를 발행한다.
* WebSocket 연결이 끊기거나 재접속된 경우, 누락 가능성이 있는 메시지는 REST API로 다시 조회한다.
* 채팅방 멤버가 아닌 사용자는 해당 채팅방 topic을 구독하거나 메시지를 전송할 수 없다.
* Phase 1에서는 `chat.translation.pending`을 별도 이벤트로 발행하지 않는다.
* PENDING 상태는 메시지 조회 응답 또는 메시지 payload의 `translations.status`로 표현한다.

## 3. 연결 Endpoint

```txt
/ws/chat
```

클라이언트는 위 endpoint로 WebSocket/STOMP 연결을 수행한다.

## 4. 인증 방식

WebSocket 연결 시 로그인 사용자의 access token을 전달한다.

권장 방식은 STOMP CONNECT header에 access token을 포함하는 방식이다.

```txt
Authorization: Bearer {accessToken}
```

BE는 WebSocket 연결 또는 메시지 처리 시 access token을 검증하고, 로그인 사용자를 식별한다.

## 5. Subscribe Destination

```txt
/topic/chat/rooms/{chatRoomId}
```

채팅방 참여자는 해당 topic을 구독한다.

### 권한 조건

* 로그인 사용자가 해당 채팅방의 active 멤버여야 한다.
* `chat_room_member.active = true`
* `chat_room_member.deleted_at is null`
* 조건을 만족하지 않는 사용자는 구독하거나 메시지를 수신할 수 없다.

## 6. Client Send Destination

```txt
/app/chat/rooms/{chatRoomId}/messages
```

클라이언트는 메시지 전송 시 위 destination으로 메시지를 publish한다.

### 권한 조건

* 로그인 사용자가 해당 채팅방의 active 멤버여야 한다.
* 멤버가 아닌 사용자의 메시지 전송은 거부한다.

## 7. 이벤트 목록

### 7.1 Client → Server

| eventType           | Destination                             | 설명                  |
| ------------------- | --------------------------------------- | ------------------- |
| `chat.message.send` | `/app/chat/rooms/{chatRoomId}/messages` | 사용자가 채팅방에 메시지를 전송한다 |

### 7.2 Server → Client

| eventType                    | Destination                      | 설명                          |
| ---------------------------- | -------------------------------- | --------------------------- |
| `chat.message.created`       | `/topic/chat/rooms/{chatRoomId}` | 메시지 원문 저장 완료 후 신규 메시지를 전달한다 |
| `chat.translation.completed` | `/topic/chat/rooms/{chatRoomId}` | 메시지 번역 완료 후 번역 결과를 전달한다     |
| `chat.translation.failed`    | `/topic/chat/rooms/{chatRoomId}` | 메시지 번역 실패 후 실패 상태를 전달한다     |
| `chat.error`                 | user queue 또는 room topic         | WebSocket 처리 중 발생한 오류를 전달한다 |

## 8. Phase 1 제외 / 보류 이벤트

| eventType                  | 사유                                                                     |
| -------------------------- | ---------------------------------------------------------------------- |
| `chat.translation.pending` | Phase 1에서는 별도 이벤트로 발행하지 않는다. PENDING 상태는 메시지 응답의 translations 상태로 표현한다 |
| `chat.message.deleted`     | Phase 1에서는 메시지 삭제 API가 대상이 아니다                                         |
| `chat.member.joined`       | Phase 1.5의 친구 그룹 초대/참가 정책에서 재검토한다                                      |
| `chat.member.left`         | Phase 1.5의 퇴실 정책에서 재검토한다                                               |
| `chat.typing`              | Phase 2 이후 검토한다                                                        |
| `chat.read`                | Phase 2 이후 읽음/미읽음 기능에서 검토한다                                            |

## 9. Server → Client 공통 Envelope

Server → Client 이벤트는 아래 구조를 기본으로 한다.

```json
{
  "eventType": "chat.message.created",
  "roomId": 1,
  "payload": {}
}
```

| 필드        | 타입     | 설명           |
| --------- | ------ | ------------ |
| eventType | string | 이벤트 타입       |
| roomId    | number | 채팅방 ID       |
| payload   | object | 이벤트별 payload |

## 10. chat.message.send

### 10.1 방향

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
  "content": "안녕하세요"
}
```

### 10.4 Payload fields

| 필드      | 타입     | 필수 | 설명     |
| ------- | ------ | -: | ------ |
| content | string |  Y | 메시지 원문 |

### 10.5 처리 기준

* 로그인 사용자가 채팅방 active 멤버인지 확인한다.
* 메시지 원문을 DB에 저장한다.
* 저장된 메시지를 `chat.message.created` 이벤트로 발행한다.
* 번역 처리는 메시지 저장 후 비동기로 요청한다.

## 11. chat.message.created

### 11.1 방향

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
    "senderName": "사용자A",
    "senderEmail": "user-a@example.com",
    "senderType": "USER",
    "messageType": "TEXT",
    "content": "안녕하세요",
    "status": "SENT",
    "translations": [],
    "createdAt": "2026-06-27T10:10:00",
    "updatedAt": "2026-06-27T10:10:00"
  }
}
```

### 11.4 Payload fields

| 필드           | 타입          | 설명               |
| ------------ | ----------- | ---------------- |
| id           | number      | 메시지 ID           |
| chatRoomId   | number      | 채팅방 ID           |
| senderUserId | number/null | 발신 사용자 ID        |
| senderName   | string/null | 발신자 이름           |
| senderEmail  | string/null | 발신자 이메일          |
| senderType   | string      | USER, AI, SYSTEM |
| messageType  | string      | TEXT, SYSTEM     |
| content      | string      | 메시지 원문           |
| status       | string      | SENT, DELETED    |
| translations | array       | 메시지 번역 목록        |
| createdAt    | string      | 생성일시             |
| updatedAt    | string      | 수정일시             |

### 11.5 FE 처리

* 수신한 메시지를 메시지 목록 하단에 추가한다.
* 동일 `id`의 메시지가 이미 존재하면 중복 추가하지 않는다.
* 번역이 아직 없으면 원문만 표시한다.
* `translations`에 PENDING 상태가 포함되어 있으면 번역 대기 UI를 표시할 수 있다.

## 12. chat.translation.completed

### 12.1 방향

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
    "languageCode": "ja",
    "translatedContent": "こんにちは",
    "status": "COMPLETED",
    "failureReason": null,
    "completedAt": "2026-06-27T10:10:05",
    "translation": {
      "id": 1001,
      "languageCode": "ja",
      "translatedContent": "こんにちは",
      "status": "COMPLETED",
      "failureReason": null,
      "completedAt": "2026-06-27T10:10:05"
    }
  }
}
```

### 12.4 Payload fields

| 필드                | 타입          | 설명               |
| ----------------- | ----------- | ---------------- |
| messageId         | number      | 번역 대상 메시지 ID     |
| languageCode      | string      | 번역 언어 코드         |
| translatedContent | string      | 번역문              |
| status            | string      | COMPLETED        |
| failureReason     | string/null | 실패 사유. 성공 시 null |
| completedAt       | string      | 번역 완료일시          |
| translation       | object      | 번역 결과 객체         |

### 12.5 FE 처리

* `messageId`에 해당하는 메시지를 찾는다.
* `languageCode` 기준으로 기존 번역과 병합한다.
* 같은 언어 번역이 이미 있으면 최신 결과로 갱신한다.
* 현재 사용자의 언어 설정에 맞는 번역만 화면에 표시한다.
* 중복 번역을 화면에 표시하지 않는다.

## 13. chat.translation.failed

### 13.1 방향

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
    "languageCode": "ja",
    "translatedContent": null,
    "status": "FAILED",
    "failureReason": "TRANSLATION_TIMEOUT",
    "completedAt": null,
    "translation": {
      "id": 1001,
      "languageCode": "ja",
      "translatedContent": null,
      "status": "FAILED",
      "failureReason": "TRANSLATION_TIMEOUT",
      "completedAt": null
    }
  }
}
```

### 13.4 Payload fields

| 필드                | 타입          | 설명           |
| ----------------- | ----------- | ------------ |
| messageId         | number      | 번역 대상 메시지 ID |
| languageCode      | string      | 번역 언어 코드     |
| translatedContent | string/null | 실패 시 null    |
| status            | string      | FAILED       |
| failureReason     | string/null | 실패 사유        |
| completedAt       | string/null | 실패 시 null    |
| translation       | object      | 번역 실패 상태 객체  |

### 13.5 FE 처리

* `messageId`에 해당하는 메시지를 찾는다.
* `languageCode` 기준으로 기존 번역 상태를 FAILED로 갱신한다.
* 번역 실패 UI를 표시한다.
* 원문 메시지는 유지한다.

## 14. chat.error

### 14.1 방향

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
    "message": "채팅방 접근 권한이 없습니다.",
    "occurredAt": "2026-06-27T10:10:00"
  }
}
```

### 14.3 Payload fields

| 필드         | 타입     | 설명     |
| ---------- | ------ | ------ |
| code       | string | 오류 코드  |
| message    | string | 오류 메시지 |
| occurredAt | string | 발생일시   |

### 14.4 주요 오류 코드

| 코드                          | 설명              |
| --------------------------- | --------------- |
| CHAT_ROOM_ACCESS_DENIED     | 채팅방 접근 권한 없음    |
| CHAT_ROOM_NOT_FOUND         | 채팅방 없음          |
| CHAT_MESSAGE_EMPTY          | 메시지 내용 없음       |
| CHAT_MESSAGE_SEND_FAILED    | 메시지 전송 실패       |
| CHAT_WEBSOCKET_UNAUTHORIZED | WebSocket 인증 실패 |

## 15. 재접속 및 동기화

WebSocket 연결이 끊긴 후 재접속되면 FE는 REST API로 최신 메시지를 다시 조회한다.

```txt
GET /api/v1/chat/rooms/{chatRoomId}/messages
```

### 처리 기준

* WebSocket 재접속 성공 후 최신 메시지 목록을 REST로 조회한다.
* 기존 메시지 목록과 `id` 기준으로 병합한다.
* 이미 존재하는 메시지는 중복 추가하지 않는다.
* 누락된 메시지나 번역 결과를 반영한다.
* WebSocket은 이력 조회를 담당하지 않는다.

## 16. REST API와 WebSocket 역할 구분

| 처리           | REST API | WebSocket |
| ------------ | -------: | --------: |
| 채팅방 생성       |        Y |         N |
| 채팅방 목록 조회    |        Y |         N |
| 채팅방 상세 조회    |        Y |         N |
| 채팅방 멤버 목록 조회 |        Y |         N |
| 메시지 초기/과거 조회 |        Y |         N |
| 메시지 실시간 송신   | fallback |         Y |
| 메시지 실시간 수신   |        N |         Y |
| 번역 완료 알림     |        N |         Y |
| 번역 실패 알림     |        N |         Y |
| 언어 설정 조회/변경  |        Y |         N |

## 17. 비고

Phase 1에서는 기본 메시지 송수신과 번역 완료/실패 반영에 집중한다.

typing indicator, read receipt, member joined/left, message deleted 이벤트는 Phase 1.5 또는 Phase 2에서 별도 설계한다.
