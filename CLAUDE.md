# 고민상담소 — 익명 게시판

## 프로젝트 개요

완전 익명 고민 게시판. 로그인 없이 누구나 글/댓글/답글/이미지 업로드 가능.
단일 HTML 파일(`index.html`) + Supabase 백엔드로 구성.

## 파일 구조

```
anonymous-board/
└── index.html   # 전체 앱 (HTML + CSS + JS 통합)
```

## 기술 스택

- **프론트엔드**: 바닐라 JS + HTML/CSS (프레임워크 없음)
- **백엔드**: Supabase (PostgreSQL + Storage)
- **Supabase SDK**: `@supabase/supabase-js@2` (CDN)
- **폰트**: Noto Sans KR (Google Fonts)

## Supabase 연결 정보

```js
SUPABASE_URL = 'https://dnpockkrhuufppuxoyyl.supabase.co'
SUPABASE_KEY = 'sb_publishable_Tu478jr1UIHstLcRkYJE4g_zLpL7vn4'  // anon 키
```

## DB 스키마

### posts 테이블

| 컬럼 | 타입 | 설명 |
|------|------|------|
| id | int8 (PK) | 자동 증가 |
| category | text | 연애/직장/인간관계/기타 |
| content | text | 본문 (최대 500자) |
| likes | int4 | 공감 수 |
| image_url | text | 첨부 이미지 URL (nullable) |
| created_at | timestamptz | 작성 시각 |

### comments 테이블

| 컬럼 | 타입 | 설명 |
|------|------|------|
| id | int8 (PK) | 자동 증가 |
| post_id | int8 (FK) | posts.id 참조 |
| content | text | 댓글 본문 (최대 200자) |
| parent_id | int8 | 답글인 경우 부모 comment.id, 최상위면 null |
| created_at | timestamptz | 작성 시각 |

## Supabase RLS 정책

아래 SQL을 Supabase SQL Editor에서 실행해야 앱이 동작함.

```sql
-- posts RLS
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;

CREATE POLICY "posts_select_anon" ON posts FOR SELECT TO anon USING (true);
CREATE POLICY "posts_insert_anon" ON posts FOR INSERT TO anon WITH CHECK (true);
CREATE POLICY "posts_update_likes_anon" ON posts FOR UPDATE TO anon USING (true) WITH CHECK (true);
REVOKE UPDATE ON posts FROM anon;
GRANT UPDATE (likes) ON posts TO anon;  -- likes 컬럼만 수정 허용

-- comments RLS
ALTER TABLE comments ENABLE ROW LEVEL SECURITY;

CREATE POLICY "comments_select_anon" ON comments FOR SELECT TO anon USING (true);
CREATE POLICY "comments_insert_anon" ON comments FOR INSERT TO anon WITH CHECK (true);
```

## Supabase Storage

버킷 이름: `post-images` (public)

```sql
-- 버킷 생성
INSERT INTO storage.buckets (id, name, public, file_size_limit, allowed_mime_types)
VALUES ('post-images', 'post-images', true, 5242880, ARRAY['image/jpeg','image/png','image/gif','image/webp'])
ON CONFLICT (id) DO NOTHING;

-- Storage RLS
CREATE POLICY "post_images_insert_anon" ON storage.objects FOR INSERT TO anon
  WITH CHECK (bucket_id = 'post-images');

CREATE POLICY "post_images_select_anon" ON storage.objects FOR SELECT TO anon
  USING (bucket_id = 'post-images');
```

## 구현된 기능

- [x] 카테고리 선택 후 익명 게시글 작성 (연애/직장/인간관계/기타)
- [x] 게시글 목록 (카테고리 필터, 페이지네이션 15개씩)
- [x] 공감(좋아요) — localStorage로 중복 방지
- [x] 댓글 작성 (최대 200자)
- [x] 1단계 답글 (parent_id로 구현)
- [x] 이미지 업로드 (Supabase Storage, 5MB 제한)
- [x] 이미지 라이트박스 (클릭 시 전체화면)
- [x] 글 작성 60초 쿨다운 (localStorage)
- [x] 모바일 대응

## UI 규칙

- 다크 테마 고정 (`--bg: #0f0f11`)
- 카테고리별 색상: 연애(핑크) / 직장(파랑) / 인간관계(초록) / 기타(노랑)
- 액센트 컬러: `#a78bfa` (보라)

## 작업 이력 요약

1. 초기 익명 게시판 구현 (posts + comments + 공감)
2. 1단계 답글 기능 추가
3. 게시글 작성 시 이미지 업로드 기능 추가
4. 모바일 이미지 업로드 버튼 대응 (iOS Safari input[type=file] 호환성 수정)
