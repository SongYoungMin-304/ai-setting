# 백엔드 개발 노트

**작성일**: 2026-04-20

## ⚠️ 중요 사항

### 게시글 및 댓글 자동 생성 금지

**정책**: 게시글과 댓글을 자동으로 생성하지 말 것

**이유**:
- 프로덕션 환경에서 데이터 오염 방지
- 사용자 테스트 시 일관성 유지
- 실제 사용 패턴에 맞는 데이터만 필요

**적용 범위**:
- 서버 시작 시 시드 데이터 생성 ❌
- API 테스트를 위한 자동 생성 ❌
- 마이그레이션 스크립트의 자동 생성 ❌

**예외**:
- 개발 환경 전용 시드 데이터 (선택사항)
- 테스트 환경의 픽스처

---

## 📝 API 스펙

### 댓글 관련 엔드포인트

#### 댓글 목록 조회
```http
GET /api/v1/posts/{postId}/comments

Response 200:
{
  "comments": [
    {
      "id": 1,
      "postId": 10,
      "content": "좋은 글입니다!",
      "author": {
        "id": 5,
        "username": "user123",
        "profileImage": "https://..."
      },
      "createdAt": "2026-04-20T10:30:00Z",
      "updatedAt": "2026-04-20T10:30:00Z",
      "parentCommentId": null,
      "replies": [...]
    }
  ]
}
```

#### 댓글 작성
```http
POST /api/v1/posts/{postId}/comments

Request:
{
  "content": "댓글 내용"
}

Response 201:
{
  "id": 1,
  "postId": 10,
  "content": "댓글 내용",
  "author": {...},
  "createdAt": "2026-04-20T10:30:00Z",
  "updatedAt": "2026-04-20T10:30:00Z",
  "parentCommentId": null,
  "replies": []
}
```

#### 대댓글 작성
```http
POST /api/v1/comments/{parentCommentId}/replies

Request:
{
  "content": "답글 내용"
}

Response 201:
{
  "id": 2,
  "postId": 10,
  "content": "답글 내용",
  "author": {...},
  "createdAt": "2026-04-20T11:00:00Z",
  "updatedAt": "2026-04-20T11:00:00Z",
  "parentCommentId": 1,
  "replies": []
}
```

#### 댓글 삭제
```http
DELETE /api/v1/comments/{commentId}

Response 204: No Content
```

---

## 🔐 인증 및 권한

### 댓글 작성
- **필요**: Bearer 토큰 (로그인 필수)
- **검증**: Authorization 헤더 확인

### 댓글 삭제
- **필요**: Bearer 토큰
- **권한**: 본인이 작성한 댓글만 삭제 가능
- **검증**: `user.id === comment.author.id`

### 응답 코드
- `201`: 성공적으로 생성됨
- `204`: 성공적으로 삭제됨
- `400`: 잘못된 요청 (유효성 검사 실패)
- `401`: 인증 필요
- `403`: 권한 없음 (다른 사용자의 댓글 삭제 시도)
- `404`: 댓글/게시글을 찾을 수 없음

---

## 📊 데이터 구조

### Comment 엔티티
```typescript
{
  id: number;              // 고유 ID
  postId: number;          // 게시글 ID
  content: string;         // 댓글 내용 (max 500자)
  author: {
    id: number;
    username: string;
    profileImage?: string;
  };
  createdAt: string;       // ISO 8601 형식
  updatedAt: string;       // ISO 8601 형식
  parentCommentId?: number; // null이면 최상위 댓글
  replies: Comment[];      // 재귀 배열
}
```

### 제약 사항
- `content`: 1-500자 사이
- `parentCommentId`: 존재하는 댓글 ID 또는 null
- 최상위 댓글: `parentCommentId = null`
- 대댓글: `parentCommentId = 부모댓글ID`

---

## ✅ 테스트 체크리스트

### 기본 CRUD
- [ ] GET /posts/{postId}/comments → 댓글 목록 조회
- [ ] POST /posts/{postId}/comments → 댓글 작성
- [ ] POST /comments/{id}/replies → 대댓글 작성
- [ ] DELETE /comments/{id} → 댓글 삭제

### 권한 검증
- [ ] 비로그인 상태에서 댓글 작성 시도 → 401
- [ ] 다른 사용자의 댓글 삭제 시도 → 403
- [ ] 본인 댓글 삭제 → 204

### 데이터 검증
- [ ] 빈 댓글 → 400
- [ ] 500자 초과 → 400
- [ ] 존재하지 않는 parentCommentId → 400 또는 404

### 재귀 구조
- [ ] 최대 3단계 깊이까지 replies 포함
- [ ] 대댓글의 replies도 배열로 반환
- [ ] 무한 깊이 지원 (선택사항)

---

## 📅 개발 일정

| 단계 | 상태 | 완료일 |
|------|------|--------|
| API 설계 | ✅ 완료 | 2026-04-20 |
| 데이터베이스 스키마 | ⏳ 예정 | - |
| API 구현 | ⏳ 예정 | - |
| 테스트 | ⏳ 예정 | - |
| 프론트 연동 | ⏳ 예정 | - |

---

## 🔗 관련 문서

- 프론트엔드 구현: `/docs/feature/02-댓글/구현완료.md`
- 설계 문서: `/docs/feature/02-댓글/설계서.md`
- API 스펙: 위 "API 스펙" 섹션 참고

---

**담당자**: Backend Team
**검토자**: Claude AI (Frontend)
