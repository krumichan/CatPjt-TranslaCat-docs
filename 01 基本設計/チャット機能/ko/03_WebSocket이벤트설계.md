# WebSocket 이벤트 설계

## 1. 기본 방침

- WebSocket은 실시간 메시지 송수신에 사용한다.
- 메시지 이력 조회는 REST API로 처리한다.
- 신규 메시지와 번역 완료 이벤트는 WebSocket으로 전달한다.
- 인증 방식은 추후 확정한다.

## 2. 연결 Endpoint

```text
/ws/chat
```

## 3. Subscribe Topic

```text
/topic/chat/rooms/{roomId}
```

채팅방 참여자는 해당 topic을 구독한다.

## 4. Client Send Endpoint

```text
/app/chat/rooms/{roomId}/messages
```

## 5. Event Types

| Event Type | 방향 | 설명 |
|---|---|---|
| message.created | Server -> Client | 새 메시지 생성 |
| translation.pending | Server -> Client | 번역 대기 상태 생성 |
| translation.completed | Server -> Client | 번역 완료 |
| translation.failed | Server -> Client | 번역 실패 |
| message.deleted | Server -> Client | 메시지 삭제 |
| error | Server -> Client | 오류 |

## 6. message.created

```json
{
  "eventType": "message.created",
  "roomId": 1,
  "message": {
    "id": 100,
    "senderType": "USER",
    "senderUserId": 1,
    "senderName": "User A",
    "originalLanguageCode": "KO",
    "originalText": "오늘 뭐 먹을래?",
    "createdAt": "2026-06-20T10:00:00"
  }
}
```

## 7. translation.completed

```json
{
  "eventType": "translation.completed",
  "roomId": 1,
  "messageId": 100,
  "translation": {
    "languageCode": "JA",
    "translatedText": "今日何食べる？",
    "status": "COMPLETED"
  }
}
```

## 8. FE 처리 방침

- `message.created` 수신 시 메시지 목록 하단에 원문 메시지를 추가한다.
- 번역이 아직 없으면 `번역 중...` 또는 숨김 상태로 표시한다.
- `translation.completed` 수신 시 해당 messageId에 번역문을 추가한다.
- 현재 사용자의 표시 대상 언어가 아닌 번역은 UI에 표시하지 않는다.
- WebSocket 끊김 발생 시 재연결을 시도한다.
