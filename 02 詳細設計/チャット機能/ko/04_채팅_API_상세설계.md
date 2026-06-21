# 채팅 API 상세설계

## 1. 목적

본 문서는 채팅 기능 REST API 상세설계 기준을 정리한다.

## 2. 1차 API 후보

```text
POST   /api/v1/chat/rooms
GET    /api/v1/chat/rooms
GET    /api/v1/chat/rooms/{roomId}
GET    /api/v1/chat/rooms/{roomId}/members
PATCH  /api/v1/chat/rooms/{roomId}/members/me/language-settings
GET    /api/v1/chat/rooms/{roomId}/messages
```

## 3. 상세 내용

Request/Response DTO는 API 구현 단계에서 보강한다.
