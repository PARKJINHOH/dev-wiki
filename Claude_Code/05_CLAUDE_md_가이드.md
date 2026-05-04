# 05. CLAUDE.md 가이드

> Claude가 프로젝트를 이해하는 기준 문서인 CLAUDE.md를 잘 작성하는 방법을 설명합니다.

---

## CLAUDE.md란?

**CLAUDE.md**는 Claude Code가 프로젝트를 시작할 때 **자동으로 읽는 마크다운 파일**입니다. 프로젝트의 컨텍스트, 규칙, 패턴을 Claude에게 "영구적으로" 전달하는 역할을 합니다.

- 프로젝트 루트에 위치 (`my-project/CLAUDE.md`)
- Claude가 세션을 시작할 때 자동으로 읽음
- 직접 작성하거나 `/init`으로 자동 생성 가능
- 프로젝트 규칙, 컨벤션, 주의사항 등을 담음

---

## 왜 중요한가?

CLAUDE.md가 없으면 Claude는 매번 프로젝트를 처음 보는 것처럼 시작합니다. 반면 잘 작성된 CLAUDE.md가 있으면:

- 프로젝트 기술 스택을 다시 설명하지 않아도 됨
- 코딩 컨벤션을 매번 말하지 않아도 Claude가 맞춰서 작업
- 하지 말아야 할 것들을 미리 명시해서 실수 방지
- 자주 쓰는 명령어를 기억해서 빠르게 활용

---

## 자동 생성

```bash
/init
```

Claude가 프로젝트를 분석해서 기본 CLAUDE.md를 생성합니다. 이후 필요한 내용을 추가로 편집하세요.

---

## 위치별 역할

CLAUDE.md는 여러 위치에 존재할 수 있으며, Claude Code는 계층적으로 모두 읽습니다.

| 위치 | 범위 |
|---|---|
| `~/.claude/CLAUDE.md` | 모든 프로젝트에 적용 (글로벌) |
| `{project}/CLAUDE.md` | 해당 프로젝트에만 적용 |
| `{project}/src/CLAUDE.md` | 해당 디렉토리 내 작업 시 추가 적용 |

```
~/.claude/CLAUDE.md              ← 전역 (모든 프로젝트에 적용)
my-project/CLAUDE.md             ← 프로젝트 루트
my-project/src/CLAUDE.md         ← 특정 디렉토리 (해당 디렉토리 작업 시 추가 로드)
```

서브디렉토리의 CLAUDE.md는 해당 디렉토리 파일을 작업할 때 추가로 적용됩니다.

---

## 작성 가이드

### 반드시 포함해야 할 내용

**1. 프로젝트 개요**
```markdown
## 프로젝트 개요
사내 인사관리 시스템. React + TypeScript 프론트엔드, Node.js + PostgreSQL 백엔드.
```

**2. 기술 스택**
```markdown
## 기술 스택
- Frontend: React 18, TypeScript 5, Vite, Tailwind CSS
- Backend: Node.js 20, Express, Prisma ORM
- DB: PostgreSQL 15
- Test: Vitest, Testing Library
```

**3. 자주 쓰는 명령어**
```markdown
## 명령어
- `npm run dev`: 개발 서버 (포트 3000)
- `npm run build`: 프로덕션 빌드
- `npm test`: 테스트 실행
- `npm run db:migrate`: DB 마이그레이션 실행
- `npm run lint`: 린트 검사
```

**4. 코딩 규칙**
```markdown
## 코딩 규칙
- 함수명: camelCase (`getUserById`)
- 컴포넌트명: PascalCase (`UserCard`)
- 상수: UPPER_SNAKE_CASE (`MAX_RETRY_COUNT`)
- 파일명: kebab-case (`user-card.tsx`)
- 타입 정의는 `types/` 폴더에 분리
- async/await 사용, Promise 체이닝 지양
```

**5. 금지 사항**
```markdown
## 주의사항
- `any` 타입 사용 금지
- `console.log` 커밋 금지 (디버깅 후 제거)
- 테스트 없이 PR 올리지 않기
- 환경변수 직접 코드에 넣지 않기 (.env 사용)
- `main` 브랜치에 직접 푸시 금지
```

---

### 선택적으로 추가하면 좋은 내용

**아키텍처 설명**
```markdown
## 아키텍처
- `src/api/`: API 라우터 (Express 라우터 파일)
- `src/services/`: 비즈니스 로직 (DB 접근 없음)
- `src/repositories/`: DB 접근 레이어 (Prisma만 사용)
- `src/components/`: 재사용 가능한 UI 컴포넌트
- `src/pages/`: 페이지 단위 컴포넌트
```

**자주 쓰는 패턴**
```markdown
## Patterns
- API 응답은 항상 `{ data, error }` 구조 반환
- DB 접근은 반드시 `src/lib/db.ts`의 트랜잭션 헬퍼 사용
- 환경변수는 `src/config/env.ts`에서만 접근
```

**과거 실수 / 주의사항 (Gotchas)**
```markdown
## Gotchas
- `userId`와 `user_id`가 혼재함 — API는 camelCase, DB는 snake_case
- `next.config.js`의 rewrites 수정 시 반드시 QA팀 확인 필요
- `src/middleware.ts`는 Edge Runtime — Node.js API 사용 불가
```

**테스트 정책**
```markdown
## 테스트
- 유닛 테스트: `*.test.ts` (서비스, 유틸 함수)
- 통합 테스트: `*.spec.ts` (API 엔드포인트)
- 커버리지 목표: 80% 이상
- 테스트 DB는 Docker로 띄움: `docker-compose up -d test-db`
```

**브랜치 전략**
```markdown
## Git 브랜치 전략
- `main`: 프로덕션 배포 브랜치
- `develop`: 개발 통합 브랜치
- 기능 브랜치: `feat/기능명`
- 버그픽스: `fix/버그설명`
- PR은 반드시 `develop` 대상으로 생성
```

---

## 효율적으로 활용하는 팁

### 1. `/init` 명령어로 자동 생성
```
/init
```
Claude Code가 코드베이스를 분석해서 CLAUDE.md 초안을 만들어줍니다. 이후 팀 규칙에 맞게 편집하세요.

### 2. 반복 요청은 CLAUDE.md로 옮기기

Claude에게 같은 지시를 두 번 이상 했다면 → CLAUDE.md에 넣을 신호입니다.

```
"앞으로 항상 에러 핸들링을 포함해줘" → CLAUDE.md에 기록
"console.log 쓰지 마라고 했잖아"      → CLAUDE.md 주의사항에 추가
```

### 3. 팀 전체에서 공유

CLAUDE.md를 git에 커밋하면 팀 전체가 동일한 Claude 동작을 얻습니다. 온보딩 문서로도 활용 가능합니다.

```bash
git add CLAUDE.md
git commit -m "docs: Claude Code 프로젝트 가이드 추가"
```

### 4. 서브디렉토리별 전문화

영역별로 다른 규칙이 필요하다면 서브디렉토리에 별도 CLAUDE.md를 둡니다.

```
/backend/CLAUDE.md   → "FastAPI 패턴, Pydantic 스키마 규칙"
/frontend/CLAUDE.md  → "React 컴포넌트 패턴, Tailwind 클래스 순서"
```

### 5. `@CLAUDE.md`로 명시적 참조

대화 중에 `@CLAUDE.md`를 언급하면 Claude가 해당 내용을 다시 참조합니다. 규칙이 지켜지지 않는다고 느낄 때 유용합니다.

```
"@CLAUDE.md에 정의된 API 응답 형식을 따라서 다시 작성해줘"
```

---

## 좋은 CLAUDE.md vs 나쁜 CLAUDE.md

### 나쁜 예

```markdown
# 프로젝트

웹 애플리케이션입니다. React를 씁니다.
```

→ 너무 적은 정보. Claude가 매번 탐색해야 함.

### 좋은 예

```markdown
# 고객관리 시스템 (CRM)

영업팀을 위한 내부 CRM 시스템. B2B 고객사, 계약, 영업 기회를 관리합니다.

## 기술 스택
- React 18 + TypeScript + Vite
- Zustand (전역 상태), React Query (서버 상태)
- Express + Prisma + PostgreSQL
- Vitest + MSW (API 모킹)

## 핵심 개념
- Customer: 고객사 (회사 단위)
- Contact: 고객사 담당자
- Deal: 영업 기회/계약
- Pipeline: 영업 단계 (Lead → Qualified → Proposal → Won/Lost)

## 개발 명령어
- `npm run dev`: http://localhost:5173
- `npm run server`: API 서버 http://localhost:4000
- `npm test`: 테스트 전체 실행
- `npm run db:seed`: 테스트 데이터 초기화

## 규칙
- 모든 API 응답은 `{ data, error, meta }` 형태
- 날짜는 UTC ISO 8601 형식으로 저장
- 페이지네이션: `?page=1&limit=20` 쿼리 파라미터
- `any` 타입 절대 사용 금지
- 모든 에러는 `AppError` 클래스를 통해 throw
```

→ Claude가 프로젝트 맥락을 바로 이해하고 올바른 방식으로 코드를 작성할 수 있음.

---

## 유지관리 팁

- 프로젝트가 변경될 때 CLAUDE.md도 함께 업데이트하세요
- 팀 컨벤션이 바뀌면 즉시 반영하세요
- Claude가 실수를 반복하면 CLAUDE.md에 금지 사항으로 추가하세요
- 너무 길어지면 핵심만 남기고 상세한 내용은 별도 문서에 링크로 연결하세요
- CLAUDE.md 자체가 너무 길어지면 Claude의 응답 품질이 저하될 수 있으므로 간결하게 유지하세요
