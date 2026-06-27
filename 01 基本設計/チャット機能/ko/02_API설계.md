# 채팅 기능 API 설계

## 1. 문서 목적

본 문서는 TranslaCat 채팅 기능 Phase 1 개발에 필요한 REST API를 정의한다.

Phase 1에서는 REST API와 WebSocket의 역할을 분리한다.

REST API는 채팅방 생성, 채팅방 목록/상세 조회, 멤버 조회, 메시지 이력 조회, 사용자별 언어 설정 조회/변경/초기화, WebSocket 장애 시 보조 메시지 저장 경로를 담당한다.

실시간 메시지 송수신, 신규 메시지 수신, 번역 완료/실패 알림은 WebSocket 이벤트에서 처리한다.

## 2. 공통 규칙

### 2.1 Base URL

```txt
/api/v1/chat
```

### 2.2 인증

모든 API는 로그인 사용자를 전제로 한다.

```txt
Authorization: Bearer {accessToken}
```

### 2.3 공통 응답 구조

모든 API 응답은 공통 응답 래퍼를 사용한다.

```json
{
  "data": {}
}
```

실제 응답 필드는 `data` 내부에 위치한다.

### 2.4 권한 기준

* 채팅방 접근 권한은 `chat_room_member`를 기준으로 확인한다.
* `active = true`이고 `deleted_at is null`인 멤버만 접근 가능한 멤버로 본다.
* 채팅방 멤버가 아닌 사용자는 채팅방 상세, 멤버 목록, 메시지 목록을 조회할 수 없다.
* 채팅방 멤버가 아닌 사용자는 메시지를 저장할 수 없다.

### 2.5 Phase 1 API 설계 기준

* Phase 1 채팅방 생성 요청은 `memberUserIds`를 사용한다.
* `memberUserIds`는 내부 사용자 ID 기반의 Phase 1 임시 구조이다.
* Phase 1.5에서는 publicId 기반 사용자 검색 및 초대 요청으로 전환한다.
* Phase 1 채팅방 생성은 기본적으로 `sourceType = MANUAL`로 생성한다.
* 메시지 목록 조회는 cursor 기반으로 처리한다.
* offset 기반 페이지네이션은 사용하지 않는다.
* 메시지 원문 저장과 번역 처리는 분리한다.
* 번역 결과는 메시지 응답의 `translations`에 포함한다.

## 3. API 목록

| 구분           | Method | URL                                                    | Phase 1    |
| ------------ | ------ | ------------------------------------------------------ | ---------- |
| 채팅방 생성       | POST   | `/api/v1/chat/rooms`                                   | 포함         |
| 내 채팅방 목록 조회  | GET    | `/api/v1/chat/rooms`                                   | 포함         |
| 채팅방 상세 조회    | GET    | `/api/v1/chat/rooms/{chatRoomId}`                      | 포함         |
| 채팅방 멤버 목록 조회 | GET    | `/api/v1/chat/rooms/{chatRoomId}/members`              | 포함         |
| 내 언어 설정 조회   | GET    | `/api/v1/chat/rooms/{chatRoomId}/members/me/language`  | 포함         |
| 내 언어 설정 변경   | PATCH  | `/api/v1/chat/rooms/{chatRoomId}/members/me/language`  | 포함         |
| 내 언어 설정 초기화  | DELETE | `/api/v1/chat/rooms/{chatRoomId}/members/me/language`  | 포함         |
| 메시지 목록 조회    | GET    | `/api/v1/chat/rooms/{chatRoomId}/messages`             | 포함         |
| REST 메시지 저장  | POST   | `/api/v1/chat/rooms/{chatRoomId}/messages`             | 포함         |
| 채팅방 멤버 추가    | POST   | `/api/v1/chat/rooms/{chatRoomId}/members`              | Phase 1 제외 |
| 채팅방 퇴실       | DELETE | `/api/v1/chat/rooms/{chatRoomId}/members/me`           | Phase 1 제외 |
| 메시지 삭제       | DELETE | `/api/v1/chat/rooms/{chatRoomId}/messages/{messageId}` | Phase 1 제외 |

## 4. 채팅방 생성

### 4.1 기본 정보

```txt
POST /api/v1/chat/rooms
```

1:1 채팅방 또는 그룹 채팅방을 생성한다.

Phase 1에서는 `memberUserIds`를 사용한다. 이 값은 내부 사용자 ID 기반의 임시 구조이며, Phase 1.5에서 publicId 기반 요청으로 전환한다.

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

| 필드            | 타입       | 필수 | 설명                 |
| ------------- | -------- | -: | ------------------ |
| roomType      | string   |  Y | DIRECT, GROUP      |
| name          | string   |  N | 채팅방 이름. GROUP에서 사용 |
| description   | string   |  N | 채팅방 설명             |
| memberUserIds | number[] |  Y | 초대 대상 사용자 ID 목록    |

### 4.4 처리 기준

* DIRECT 생성 시 `memberUserIds`는 1명이어야 한다.
* GROUP 생성 시 `memberUserIds`는 1명 이상이어야 한다.
* 로그인 사용자는 자동으로 OWNER 멤버가 된다.
* 생성 대상 사용자는 MEMBER 멤버가 된다.
* Phase 1 생성 채팅방은 `sourceType = MANUAL`을 사용한다.
* 같은 사용자 조합의 DIRECT 방이 이미 존재하고 `sourceType = MANUAL`이면 기존 방을 반환할 수 있다.
* OPEN roomType 생성은 Phase 1에서 허용하지 않는다.

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

## 5. 내 채팅방 목록 조회

### 5.1 기본 정보

```txt
GET /api/v1/chat/rooms
```

로그인 사용자가 참여 중인 채팅방 목록을 조회한다.

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

| 필드                      | 타입          | 설명                       |
| ----------------------- | ----------- | ------------------------ |
| chatRooms               | array       | 채팅방 목록                   |
| chatRooms[].id          | number      | 채팅방 ID                   |
| chatRooms[].roomType    | string      | DIRECT, GROUP, OPEN      |
| chatRooms[].sourceType  | string      | MANUAL, FRIEND, OPEN, AI |
| chatRooms[].name        | string/null | 채팅방 이름                   |
| chatRooms[].description | string/null | 채팅방 설명                   |
| chatRooms[].ownerId     | number/null | 채팅방 소유자 ID               |
| chatRooms[].memberCount | number      | active 멤버 수              |
| chatRooms[].createdAt   | string      | 생성일시                     |
| chatRooms[].updatedAt   | string      | 수정일시                     |

## 6. 채팅방 상세 조회

### 6.1 기본 정보

```txt
GET /api/v1/chat/rooms/{chatRoomId}
```

로그인 사용자가 참여 중인 특정 채팅방의 상세 정보를 조회한다.

### 6.2 Path parameters

| 파라미터       | 타입     | 설명     |
| ---------- | ------ | ------ |
| chatRoomId | number | 채팅방 ID |

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

### 6.4 권한 조건

* 로그인 사용자가 해당 채팅방의 active 멤버여야 한다.
* 멤버가 아닌 경우 접근을 차단한다.

## 7. 채팅방 멤버 목록 조회

### 7.1 기본 정보

```txt
GET /api/v1/chat/rooms/{chatRoomId}/members
```

채팅방의 active 멤버 목록을 조회한다.

### 7.2 Response

```json
{
  "data": {
    "members": [
      {
        "id": 1,
        "chatRoomId": 1,
        "userId": 1,
        "name": "사용자A",
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

### 7.3 권한 조건

* 로그인 사용자가 해당 채팅방의 active 멤버여야 한다.

## 8. 내 채팅방 언어 설정 조회

### 8.1 기본 정보

```txt
GET /api/v1/chat/rooms/{chatRoomId}/members/me/language
```

로그인 사용자의 해당 채팅방 언어 설정을 조회한다.

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

## 9. 내 채팅방 언어 설정 변경

### 9.1 기본 정보

```txt
PATCH /api/v1/chat/rooms/{chatRoomId}/members/me/language
```

로그인 사용자의 해당 채팅방 한정 언어 설정을 변경한다.

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

| 필드                      | 타입      | 필수 | 설명       |
| ----------------------- | ------- | -: | -------- |
| originalLanguageCode    | string  |  Y | 원문 기준 언어 |
| translationLanguageCode | string  |  Y | 수신 번역 언어 |
| showOriginal            | boolean |  Y | 원문 표시 여부 |
| showTranslation         | boolean |  Y | 번역 표시 여부 |

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

## 10. 내 채팅방 언어 설정 초기화

### 10.1 기본 정보

```txt
DELETE /api/v1/chat/rooms/{chatRoomId}/members/me/language
```

로그인 사용자의 해당 채팅방 언어 설정을 초기화하고 기본 언어 설정을 따르도록 되돌린다.

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

## 11. 메시지 목록 조회

### 11.1 기본 정보

```txt
GET /api/v1/chat/rooms/{chatRoomId}/messages
```

채팅방의 최신 메시지 또는 cursorId 기준 이전 메시지를 조회한다.

### 11.2 Query parameters

| 파라미터     | 타입     | 필수 | 설명                  |
| -------- | ------ | -: | ------------------- |
| cursorId | number |  N | 이전 메시지 조회 기준 메시지 ID |

### 11.3 요청 예시

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
        "senderName": "사용자A",
        "senderEmail": "user-a@example.com",
        "senderType": "USER",
        "messageType": "TEXT",
        "content": "안녕하세요",
        "status": "SENT",
        "translations": [
          {
            "id": 1001,
            "languageCode": "ja",
            "translatedContent": "こんにちは",
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

### 11.5 처리 기준

* `cursorId`가 없으면 최신 메시지 목록을 조회한다.
* `cursorId`가 있으면 해당 ID보다 이전 메시지를 조회한다.
* 응답에는 다음 조회에 사용할 `nextCursorId`를 포함한다.
* 더 조회할 메시지가 있으면 `hasNext = true`를 반환한다.
* 번역 결과는 `translations` 배열에 포함한다.
* 번역이 없으면 `translations = []`를 반환한다.

## 12. REST 기반 메시지 저장

### 12.1 기본 정보

```txt
POST /api/v1/chat/rooms/{chatRoomId}/messages
```

채팅방에 텍스트 메시지 원문을 저장한다.

Phase 1의 기본 메시지 송신 경로는 WebSocket이지만, REST 메시지 저장 API는 WebSocket 구현 전 테스트 및 WebSocket 장애 시 fallback 경로로 사용할 수 있다.

### 12.2 Request

```json
{
  "content": "안녕하세요"
}
```

### 12.3 Response

```json
{
  "data": {
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

### 12.4 처리 기준

* 로그인 사용자가 채팅방 active 멤버여야 한다.
* 메시지 원문을 먼저 저장한다.
* 번역 처리는 메시지 저장과 분리한다.
* 번역 요청/결과 반영은 비동기 처리한다.
* WebSocket 사용 가능 시 FE는 WebSocket 송신을 우선 사용한다.

## 13. Phase 1 제외 API

### 13.1 채팅방 멤버 추가

```txt
POST /api/v1/chat/rooms/{chatRoomId}/members
```

Phase 1에서는 제외한다.

Phase 1.5에서 친구 목록 기반 그룹 초대 또는 publicId 기반 초대 요청으로 재설계한다.

### 13.2 채팅방 퇴실

```txt
DELETE /api/v1/chat/rooms/{chatRoomId}/members/me
```

Phase 1에서는 제외한다.

Phase 1.5에서 친구 그룹 채팅 및 초대 수락/거절 정책과 함께 재검토한다.

### 13.3 메시지 삭제

```txt
DELETE /api/v1/chat/rooms/{chatRoomId}/messages/{messageId}
```

Phase 1에서는 제외한다.

`ChatMessageStatus = DELETED` 구조는 존재하지만, 사용자 삭제 API는 Phase 1.5 또는 Phase 2에서 재검토한다.

## 14. REST API와 WebSocket 역할 구분

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

## 15. 비고

Phase 1의 API 설계는 기본 채팅 MVP를 완성하기 위한 최소 범위로 정의한다.

친구 추가, 친구 목록, publicId 기반 사용자 검색, 그룹 초대, 채팅방 퇴실, 메시지 삭제, 오픈 채팅, AI 참여자 관련 API는 Phase 1.5 또는 Phase 2에서 별도 설계한다.
