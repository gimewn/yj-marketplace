---
name: scaffold-crud
description: Next.js + Supabase 프로젝트에서 단일 테이블 CRUD(DB 마이그레이션·API Routes·UI·RLS)를 한 번에 스캐폴딩한다. 'CRUD 만들어 주십시오', '단일 테이블 추가해 주십시오', 'API + 페이지 만들어 주십시오' 같은 요청에 자동 호출한다.
allowed-tools:
  - Read
  - Write
  - Edit
  - mcp__plugin_supabase_supabase__apply_migration
  - mcp__plugin_supabase_supabase__generate_typescript_types
---

# scaffold-crud

## 사용 시점

다음 표현이 포함된 요청에서 자동으로 호출한다.

- "CRUD 만들어 주십시오 / 주세요"
- "단일 테이블 추가해 주십시오 / 주세요"
- "API + 페이지 만들어 주십시오" (리소스 이름이 명시된 경우)

리소스 이름이 `$ARGUMENTS`로 전달된다. 이름이 비어 있으면 먼저 확인한다.

## 진행 순서

1. **기존 패턴 파악** — `CLAUDE.md`, `src/lib/database.types.ts`, 기존 API 라우트를 읽어 프로젝트 컨벤션을 파악한다.
2. **DB 마이그레이션** — `apply_migration`으로 테이블 생성 + RLS 활성화 + 4개 정책(SELECT·INSERT·UPDATE·DELETE) 적용.
3. **타입 동기화** — `generate_typescript_types`로 `src/lib/database.types.ts`를 재생성한다.
4. **API Routes 생성**
   - `src/app/api/<resource>/route.ts` — GET(목록·필터), POST(생성)
   - `src/app/api/<resource>/[id]/route.ts` — GET(단건), PATCH(수정), DELETE(삭제)
5. **UI 페이지 생성** — `src/app/<resource>/page.tsx` — 목록, 인라인 수정, 삭제 버튼
6. **미들웨어 업데이트** — `src/middleware.ts`의 `matcher`에 `/api/<resource>` 와 `/api/<resource>/:path*` 추가

## 컨벤션

| 항목 | 규칙 |
|------|------|
| 파일 경로 | `src/app/api/<resource>/route.ts` |
| 인증 | 각 핸들러 첫 줄에 `supabase.auth.getUser()` 확인, 미인증 시 401 반환 |
| created_by | 클라이언트에서 받지 않고 서버에서 `user.id`로 자동 설정 |
| 에러 코드 | PGRST116 → 404, 그 외 → 500 |
| 응답 형식 | GET 목록 → 배열, POST → 201 + 단건, PATCH → 200 + 단건, DELETE → 204 no body |
| UI 라이브러리 | 프로젝트 CLAUDE.md 기준 (Tailwind v4 + shadcn 등) |
| 상태 관리 | `useState` + fetch. 전역 상태는 프로젝트 컨벤션 따름 |

## 주의사항

- **RLS 필수**: `alter table <resource> enable row level security`를 마이그레이션에 반드시 포함한다. 누락 시 anon 키로 전체 행이 노출된다.
- **테이블명 plural**: Supabase 컨벤션은 복수형(`comments`, `tasks`). 단수로 만들지 않는다.
- **PostgREST join**: `select('*, other_table(col)')`은 직접 FK가 없으면 500. `auth.users`는 public 스키마가 아니므로 join 불가 — 별도 `profiles` 테이블을 두고 API 레벨에서 merge한다.
- **타입 재생성**: 마이그레이션 후 반드시 `generate_typescript_types`를 실행해 `database.types.ts`를 최신 상태로 유지한다.
- **미들웨어**: 새 API 경로를 `matcher`에 추가하지 않으면 인증 없이 접근 가능해진다.
