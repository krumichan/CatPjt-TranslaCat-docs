# 채팅 WebSocket 상세설계

## 1. 목적

본 문서는 채팅 기능 WebSocket 상세설계 기준을 정리한다.

## 2. Endpoint 후보

```text
/ws
```

## 3. Destination 후보

```text
/app/chat/rooms/{roomId}/messages
/topic/chat/rooms/{roomId}
```

## 4. 이벤트 후보

- chat.message.created
- chat.translation.pending
- chat.translation.completed
- chat.translation.failed
- chat.error

## 5. 상세 내용

Payload 상세는 WebSocket 구현 단계에서 보강한다.
