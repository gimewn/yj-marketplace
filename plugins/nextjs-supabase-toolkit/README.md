# nextjs-supabase-toolkit

Next.js + Supabase 프로젝트를 위한 Claude Code 플러그인.
코드 리뷰 에이전트 4종과 CRUD 스캐폴딩·문서 작성 스킬 2종을 제공한다.

## 구성 요소

### 에이전트 (자동 호출)

| 에이전트 | 역할 | 자동 호출 시점 |
|---------|------|-------------|
| `code-reviewer` | 보안·정확성·성능·가독성·컨벤션 종합 리뷰 | 소스 파일 수정 후 |
| `security-reviewer` | RLS·OAuth·비밀키·XSS 보안 전용 | 인증/인가/DB 코드 수정 후 |
| `perf-reviewer` | N+1·컴포넌트 경계·렌더 비용·자산 로딩 | 쿼리·렌더 코드 수정 후 |
| `test-reviewer` | 단위·통합·E2E·회귀 위험 커버리지 | 새 기능·API 추가 후 |

### 스킬 (자동 활성화)

| 스킬 | 역할 | 트리거 |
|------|------|--------|
| `scaffold-crud` | 단일 테이블 CRUD 전체 스캐폴딩 | "CRUD 만들어 주세요", "테이블 추가해 주세요" |
| `write-docs` | docs/ 문서 5종 순서대로 작성 | "문서 만들어줘", "docs 작성해줘" |

## 설치

```bash
# 로컬 플러그인으로 설치
cc --plugin-dir /path/to/nextjs-supabase-toolkit

# 또는 프로젝트 내 .claude/plugins/ 에 복사
cp -r nextjs-supabase-toolkit ~/.claude/plugins/
```

## 사용법

### 코드 리뷰

코드를 수정하면 에이전트가 자동으로 호출된다.
특정 관점만 검토하려면 직접 요청한다.

```
"방금 추가한 API 보안 리뷰 해줘"
→ security-reviewer 에이전트 호출

"성능 리뷰 해줘"
→ perf-reviewer 에이전트 호출

"테스트 커버리지 확인해줘"
→ test-reviewer 에이전트 호출
```

### CRUD 스캐폴딩

```
"comments CRUD 만들어 주세요"
→ DB 마이그레이션 + API Routes + UI 페이지 + 미들웨어 업데이트 자동 생성
```

### 문서 작성

```
"docs 전체 만들어줘"
→ personas.md → user-stories.md → architecture.md → db.md → api.md 순서로 생성
```

## 호환 환경

- **Next.js**: App Router (v14+)
- **Supabase**: supabase-js v2, @supabase/ssr
- **UI**: Tailwind CSS v3/v4, shadcn/ui (base-nova 포함)
- **상태 관리**: Zustand, Redux, 또는 기타
- **테스트**: Vitest + Playwright

## 에이전트 출력 우선순위

모든 에이전트는 발견 사항을 P0/P1/P2로 분류해 보고한다.

| 등급 | 의미 | 조치 |
|------|------|------|
| P0 | 즉시 수정 필수 (보안 취약점, 데이터 손상 가능성) | 릴리즈 차단 |
| P1 | 릴리즈 전 수정 권장 | 강력 권장 |
| P2 | 개선 권장 | 선택 |
