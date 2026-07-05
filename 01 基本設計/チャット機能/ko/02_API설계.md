# 채팅 기능 REST API 설계

## 1. 목적

Phase 1 기본 채팅 API와 Phase 1.5 프로필/친구/차단/친구 채팅 API를 통합 정리한다.

## 2. 공통 규칙

Base URL:

```text
/api/v1
```

공개 API가 아닌 경우 JWT 로그인 사용자를 기준으로 처리한다.

공통 응답 예시:

```json
{
  "resultCode": 200,
  "message": "success",
  "body": {},
  "guid": null,
  "createDate": "2026-07-05 18:00:00"
}
```

실제 응답 데이터는 `body`에 포함한다.

## 3. API 목록

### 프로필/사용자
- `GET /api/v1/users/me/profile`
- `PATCH /api/v1/users/me/profile`
- `GET /api/v1/users/search?publicId={publicId}`

### 친구 요청
- `POST /api/v1/friend-requests`
- `GET /api/v1/friend-requests/received`
- `GET /api/v1/friend-requests/sent`
- `PATCH /api/v1/friend-requests/{requestId}/accept`
- `PATCH /api/v1/friend-requests/{requestId}/reject`
- `PATCH /api/v1/friend-requests/{requestId}/cancel`

### 친구
- `GET /api/v1/friends`
- `DELETE /api/v1/friends/{friendUserId}`

### 차단
- `POST /api/v1/blocks`
- `GET /api/v1/blocks`
- `DELETE /api/v1/blocks/{blockedUserId}`

### 친구 채팅
- `POST /api/v1/chat/friends/{friendUserId}/direct-room`
- `POST /api/v1/chat/friends/group-rooms`

### 기존 채팅
- `POST /api/v1/chat/rooms`
- `GET /api/v1/chat/rooms`
- `GET /api/v1/chat/rooms/{roomId}`
- `GET /api/v1/chat/rooms/{roomId}/messages`
- `POST /api/v1/chat/rooms/{roomId}/messages`

메시지/WebSocket/번역/언어 설정 세부 흐름은 Phase 1 설계를 유지한다.

## 4. 내 프로필 조회

```http
GET /api/v1/users/me/profile
```

예시:

```json
{
  "resultCode": 200,
  "message": "success",
  "body": {
    "userId": 1,
    "publicId": "TC-H6VR-9KQD",
    "nickname": "krumi",
    "profileImageUrl": null,
    "bio": "안녕하세요"
  },
  "guid": null,
  "createDate": "2026-07-05 18:00:00"
}
```

## 5. 내 프로필 수정

```http
PATCH /api/v1/users/me/profile
```

Request:

```json
{
  "nickname": "krumi",
  "profileImageUrl": "https://example.com/profile.png",
  "bio": "안녕하세요"
}
```

정책:
- publicId 수정 불가
- nickname validation 적용
- profileImageUrl/bio는 정책 범위에서 null 가능
- 파일 업로드는 후속 이슈

## 6. publicId 사용자 검색

```http
GET /api/v1/users/search?publicId=TC-PFSN-CLNA
```

대표 정보:
- userId
- publicId
- nickname
- profileImageUrl
- 친구 관계 상태
- 차단 관련 액션 가능 여부

정책:
- 자기 자신 요청 방지
- 이미 친구인 경우 중복 요청 방지
- PENDING 중복 방지
- 차단 관계 요청 제한

## 7. 친구 요청 전송

```http
POST /api/v1/friend-requests
```

```json
{
  "receiverPublicId": "TC-PFSN-CLNA"
}
```

조건:
- 대상 존재
- 자기 자신 아님
- 이미 친구 아님
- 동일 PENDING 없음
- 양방향 차단 관계 없음

성공 시 상태는 `PENDING`.

## 8. 받은/보낸 요청

```http
GET /api/v1/friend-requests/received
GET /api/v1/friend-requests/sent
```

FE 알림 센터에서:
- 받은 요청: 수락/거절
- 보낸 요청: 취소

## 9. 요청 상태 변경

수락:

```http
PATCH /api/v1/friend-requests/{requestId}/accept
```

- 수신자만 가능
- PENDING만 가능
- 성공 시 친구 관계 생성
- 중복 관계 생성 금지

거절:

```http
PATCH /api/v1/friend-requests/{requestId}/reject
```

- 수신자만 가능
- PENDING만 가능
- `REJECTED` 전환

취소:

```http
PATCH /api/v1/friend-requests/{requestId}/cancel
```

- 발신자만 가능
- PENDING만 가능
- `CANCELED` 전환

## 10. 친구 목록/삭제

```http
GET /api/v1/friends
```

친구 카드 대표 정보:
- friend userId
- publicId
- nickname
- profileImageUrl
- 친구 관계 정보

사용처:
- 목록 표시
- 검색 필터
- FRIEND DIRECT
- FRIEND GROUP 멤버 선택

삭제:

```http
DELETE /api/v1/friends/{friendUserId}
```

- 친구 관계 제거
- 기존 방/메시지는 삭제하지 않음
- 신규 친구 채팅 시작 가능 여부는 현재 관계 기준으로 검증

## 11. 차단

차단:

```http
POST /api/v1/blocks
```

```json
{
  "blockedPublicId": "TC-3567-W4EZ"
}
```

정책:
- 자기 자신 차단 불가
- 중복 차단 불가
- 친구 요청 제한
- FRIEND DIRECT 신규 시작 제한
- FRIEND GROUP 선택/생성 제한
- 차단 후 친구 목록에서 제외

목록:

```http
GET /api/v1/blocks
```

해제:

```http
DELETE /api/v1/blocks/{blockedUserId}
```

해제 후 이전 친구 관계는 자동 복구하지 않는 것을 기본으로 한다.

## 12. FRIEND DIRECT

```http
POST /api/v1/chat/friends/{friendUserId}/direct-room
```

Request body 없음.

처리:
1. 인증 사용자 확인
2. 대상 사용자 확인
3. 친구 관계 확인
4. 양방향 차단 확인
5. 동일 사용자 조합의 `DIRECT + FRIEND` 조회
6. 있으면 기존 방 반환
7. 없으면 신규 생성
8. 양 사용자를 멤버로 등록
9. 방 응답 반환

정책:
- `roomType = DIRECT`
- `sourceType = FRIEND`
- MANUAL DIRECT는 재사용하지 않음

대표 응답:

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
  "name": "주말 일본어 공부방",
  "description": "친구들과 일본어를 공부하는 방",
  "memberUserIds": [2, 3]
}
```

처리:
1. 인증 사용자 확인
2. 멤버 목록 중복/유효성 검증
3. 모두 현재 친구인지 확인
4. 양방향 차단 확인
5. `GROUP + FRIEND` 방 생성
6. 생성자를 OWNER로 추가
7. 선택 친구를 MEMBER로 추가
8. 방 응답 반환

정책:
- memberUserIds에 자기 자신 미포함
- 모든 대상은 현재 친구
- 차단 관계 사용자 포함 불가
- Phase 1.5에서는 동일 멤버 조합 그룹 중복 생성 허용

## 14. 기존 MANUAL API

```http
POST /api/v1/chat/rooms
```

유지 정책:
- API/기능 삭제 안 함
- 기존 MANUAL 방 목록/상세/메시지 유지
- 일반 사용자 UI에서는 내부 userId 직접 입력 생성 흐름 비노출
- 정상 사용자 진입점은 친구 목록 기반 API

## 15. FE 상태/오류 처리

구분:
- Loading
- Empty
- Validation Error
- API Error
- Retry 가능 상태
- 인증 만료
- 관계/권한 검증 실패

채팅방 이동 후 방 상세, 메시지, 언어 설정, WebSocket 흐름이 정상 연결되어야 한다.

## 16. Phase 2 확장

기존 채팅 구조를 유지하며 sourceType과 정책을 확장한다.

- OPEN
- AI
- 가상 프로필
- AI 멤버
- 공개방 검색
- 운영/신고 기능
