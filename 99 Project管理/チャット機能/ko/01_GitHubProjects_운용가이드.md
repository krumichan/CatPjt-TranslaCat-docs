# GitHub Projects 운용 가이드

## 1. 목적

TranslaCat 채팅 기능 개발은 FE/BE/docs repo가 함께 움직이므로, 작업 관리는 GitHub Projects에서 통합 관리한다.

## 2. Project 이름

```text
TranslaCat Chatting Development
```

## 3. 상태 컬럼

| 컬럼 | 의미 |
|---|---|
| Backlog | 언젠가 해야 할 후보 작업 |
| Todo | 이번 개발 범위에 포함되어 있고 곧 착수할 작업 |
| In Progress | 현재 작업 중인 작업 |
| Review | 구현 완료 후 확인/리뷰/테스트 중인 작업 |
| Done | 완료된 작업 |
| Hold | 보류 또는 차단된 작업 |

## 4. Custom Fields

| Field | 값 |
|---|---|
| Phase | 1차, 2차 |
| Area | BE, FE, Docs, QA, Infra, AI, Common |
| Priority | High, Medium, Low |
| Type | Feature, Docs, Test, Research, Refactor, Bug |
| Target Repo | BE, FE, Docs, AI |

## 5. Issue 생성 위치

| 작업 종류 | Issue 생성 위치 |
|---|---|
| BE 작업 | CatPjt-TranslaCat-be |
| FE 작업 | CatPjt-TranslaCat-fe |
| 문서/기획 작업 | CatPjt-TranslaCat-docs |
| 공통/운영/설계 판단 | CatPjt-TranslaCat-docs |

## 6. Issue 제목 규칙

```text
[BE] 채팅방 생성 API 구현
[FE] 채팅방 목록 화면 구현
[DOCS] DB 설계 작성
[QA] WebSocket 메시지 송수신 확인
[ARCH] Redis 도입 여부 검토
```

## 7. Epic 운영 방식

docs repo에 Epic Issue를 만든다.

```text
[EPIC] 1차 채팅 MVP 개발
[EPIC] 2차 채팅 확장 개발
```

각 BE/FE/Docs 작업 Issue는 Epic에 연결한다.

## 8. 인수인계 규칙

작업을 이어갈 때 아래 3개 URL을 전달한다.

```text
1. https://github.com/krumichan/CatPjt-TranslaCat-docs
2. https://github.com/users/krumichan/projects/2
3. 현재 작업 중인 Issue URL
```

## 9. 작업 종료 시 갱신 규칙

작업이 완료될 때마다 다음을 갱신한다.

- GitHub Project 상태
- 관련 Issue 체크리스트
- docs repo의 현재상황 문서
- 필요 시 API/DB/WebSocket 설계 문서
