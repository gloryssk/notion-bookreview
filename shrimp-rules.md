# 노션 기반 북리뷰 AI 개발 가이드

**프로젝트:** 노션 기반 북리뷰 서비스 (Notion Book Review Web App)
**기술 스택:** Next.js 15.5.3, React 19, TypeScript 5, TailwindCSS v4, shadcn/ui
**데이터베이스:** Supabase (PostgreSQL) + Notion API
**배포 플랫폼:** Vercel

---

## 📋 프로젝트 개요

사용자가 노션에서 작성한 책 리뷰를 웹 서비스에서 통합 관리하고 개인 라이브러리로 열람 및 수정할 수 있는 현대적 웹 애플리케이션입니다.

**핵심 기능 (11가지):**
- F001: 회원가입 (이메일/비밀번호)
- F002: 로그인 (이메일/비밀번호)
- F003: 노션 API 연동 (OAuth)
- F004: 리뷰 목록 조회
- F005: 리뷰 상세 조회
- F006: 리뷰 작성 (웹)
- F007: 리뷰 수정
- F008: 리뷰 삭제
- F009: 로그아웃
- F010: 노션 자동 동기화 (30분 주기)
- F011: 인증 확인 (보호 페이지 접근 제어)

**개발 단계:**
- Phase 1: 애플리케이션 골격 구축 (라우팅, 타입 정의)
- Phase 2: UI/UX 완성 (더미 데이터)
- Phase 3: 핵심 기능 구현 (API, 인증, 동기화)
- Phase 4: 고급 기능 및 최적화

---

## 🏗️ 아키텍처 규칙 (필수)

### 1. Next.js App Router (필수)

**필수 규칙:**
- `src/app/` 디렉토리만 사용 - Pages Router 절대 금지
- App Router 파일 규칙 준수:
  - `page.tsx` - 해당 경로의 페이지
  - `layout.tsx` - 레이아웃 (자식 페이지 감싸기)
  - `loading.tsx` - 로딩 UI (필요시)
  - `error.tsx` - 에러 바운더리 (필수)
  - `not-found.tsx` - 404 페이지 (필수)

**라우팅 그룹 구조:**
```
src/app/
├── page.tsx                    # 랜딩 페이지 (/)
├── layout.tsx                  # 루트 레이아웃
├── (auth)/                     # 인증 페이지 그룹
│   ├── login/page.tsx         # 로그인
│   ├── signup/page.tsx        # 회원가입
│   └── layout.tsx             # 인증 레이아웃
├── (protected)/               # 보호된 페이지 그룹
│   ├── reviews/page.tsx       # 리뷰 목록
│   ├── reviews/[id]/page.tsx  # 리뷰 상세
│   ├── reviews/[id]/edit/page.tsx  # 리뷰 수정
│   ├── reviews/create/page.tsx     # 리뷰 작성
│   ├── settings/notion/page.tsx    # 노션 연동
│   └── layout.tsx             # 보호된 페이지 레이아웃
└── api/                       # Server Actions (라우트 핸들러 최소화)
```

### 2. Server Components 우선 (필수)

**규칙:**
- 모든 컴포넌트는 기본적으로 Server Component
- `'use client'` 지시문은 **상호작용이 필수인 경우에만** 사용
- 상호작용이 있는 컴포넌트를 작은 단위로 분리해 클라이언트 컴포넌트 최소화

**예시:**
```typescript
// ✅ Server Component (기본)
export default async function ReviewListPage() {
  const reviews = await getReviews()
  return (
    <div>
      <h1>리뷰 목록</h1>
      {/* 클라이언트 컴포넌트 분리 */}
      <ReviewTable reviews={reviews} />
    </div>
  )
}

// ✅ 클라이언트 컴포넌트 (최소한)
'use client'
export function ReviewTable({ reviews }: { reviews: Review[] }) {
  const [selectedId, setSelectedId] = useState<string | null>(null)
  return <table>...</table>
}
```

### 3. Request APIs (Next.js 15.5.3 신규)

**필수 규칙:**
- `params`와 `searchParams`는 **반드시 Promise로 처리**
- 동기식 접근 금지 (deprecated)

**올바른 사용:**
```typescript
export default async function Page({
  params,
  searchParams
}: {
  params: Promise<{ id: string }>
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>
}) {
  const { id } = await params          // ✅ 필수: await
  const query = await searchParams     // ✅ 필수: await
  const cookieStore = await cookies()
  const headersList = await headers()

  // ...
}
```

---

## 📁 파일 구조 및 네이밍 규칙

### 디렉토리 구조 (필수)

```
src/
├── app/                    # Next.js App Router
├── components/             # React 컴포넌트
│   ├── ui/                # shadcn/ui 컴포넌트
│   ├── layout/            # 레이아웃 컴포넌트
│   ├── navigation/        # 네비게이션 컴포넌트
│   ├── sections/          # 페이지 섹션 컴포넌트
│   ├── forms/             # 폼 컴포넌트
│   ├── reviews/           # 리뷰 관련 컴포넌트
│   ├── modals/            # 모달 컴포넌트
│   └── providers/         # Context 프로바이더
├── lib/                    # 유틸리티 및 설정
│   ├── auth.ts            # 인증 관련 유틸
│   ├── supabase.ts        # Supabase 클라이언트
│   ├── notion.ts          # Notion API 유틸
│   ├── utils.ts           # 공통 유틸리티
│   └── env.ts             # 환경 변수 타입
├── types/                  # TypeScript 타입 정의
│   ├── user.ts            # 사용자 타입
│   ├── review.ts          # 리뷰 타입
│   ├── notion.ts          # Notion 타입
│   └── auth.ts            # 인증 타입
└── actions/               # Server Actions
    ├── auth.ts            # 인증 액션
    ├── reviews.ts         # 리뷰 CRUD 액션
    └── notion.ts          # 노션 동기화 액션
```

### 파일명 규칙

| 파일 타입 | 규칙 | 예시 |
|-----------|------|------|
| **컴포넌트** | PascalCase (.tsx) | `ReviewCard.tsx`, `LoginForm.tsx` |
| **페이지** | page.tsx | `src/app/reviews/page.tsx` |
| **유틸/라이브러리** | camelCase (.ts) | `utils.ts`, `auth.ts` |
| **타입 정의** | camelCase (.ts) | `user.ts`, `review.ts` |
| **Server Actions** | camelCase (.ts) | `auth.ts`, `reviews.ts` |
| **스타일** | module.css 또는 Tailwind | Tailwind CSS 우선 사용 |

---

## 🧩 컴포넌트 작성 표준

### 1. 컴포넌트 분류 (필수)

**컴포넌트는 5가지 폴더에 분류:**

1. **ui/** - shadcn/ui 기반 순수 UI 컴포넌트
   - 비즈니스 로직 없음
   - 모든 동작은 props로 제어
   - 재사용 가능성 최대

2. **layout/** - 페이지 구조 컴포넌트
   - Header, Footer, Container, Sidebar
   - 전체 페이지 레이아웃 담당

3. **navigation/** - 네비게이션 컴포넌트
   - MainNav, MobileNav, Breadcrumb
   - 페이지 간 이동 담당

4. **sections/** - 페이지 섹션 컴포넌트
   - Hero, Features, CTA
   - 특정 페이지의 블록 단위 컴포넌트

5. **forms/** / **reviews/** / **modals/** - 기능별 컴포넌트
   - LoginForm, ReviewCard, ConfirmModal
   - 특정 기능을 구현하는 컴포넌트

### 2. 컴포넌트 작성 원칙

**단일 책임 원칙 준수:**
- 각 컴포넌트는 **한 가지 명확한 책임만** 담당
- 책임이 증가하면 세분화

**Props 타입 정의 필수:**
```typescript
// ✅ 명시적 props 타입
interface ReviewCardProps {
  review: Review
  onEdit: (id: string) => void
  onDelete: (id: string) => Promise<void>
}

export function ReviewCard({ review, onEdit, onDelete }: ReviewCardProps) {
  // ...
}

// ❌ any 타입 금지
export function ReviewCard(props: any) { }
```

**컴포지션 우선:**
```typescript
// ✅ 컴포지션
<Card>
  <CardHeader>
    <CardTitle>제목</CardTitle>
  </CardHeader>
  <CardContent>내용</CardContent>
</Card>

// ❌ 과도한 props
<Card title="제목" content="내용" footer="..." />
```

---

## 🔤 네이밍 규칙 (필수)

### 변수, 함수, 유틸

**camelCase 필수:**
```typescript
// ✅ 올바른 예
const userId = "123"
const isAuthenticated = true
const handleSubmit = () => {}
const getReviewById = async (id: string) => {}
const parseNotionData = (data: unknown) => {}

// ❌ 금지
const user_id = "123"
const UserID = "123"
const handlesubmit = () => {}
const get_review_by_id = () => {}
```

### 컴포넌트명

**PascalCase 필수:**
```typescript
// ✅ 올바른 예
export function ReviewCard() {}
export function LoginForm() {}
export function UserAvatar() {}

// ❌ 금지
export function reviewCard() {}
export function login_form() {}
export const UserAvatar = () => {}  // 화살표 함수보다 function 선호
```

### 이벤트 핸들러, 콜백

**on/handle 접두사 사용:**
```typescript
// ✅ 올바른 예
<button onClick={handleDelete}>삭제</button>
<input onChange={handleInputChange} />
const onSuccess = () => {}
const onError = (error: Error) => {}

// ❌ 금지
<button onClick={delete}>삭제</button>
<button onClick={deleteReview}>삭제</button>
const success = () => {}
```

### API/Server Action 함수

**동사로 시작 (getXxx, createXxx, updateXxx, deleteXxx):**
```typescript
// ✅ 올바른 예
async function getReviews(userId: string): Promise<Review[]> {}
async function createReview(data: CreateReviewRequest): Promise<Review> {}
async function updateReview(id: string, data: UpdateReviewRequest): Promise<Review> {}
async function deleteReview(id: string): Promise<void> {}

// ❌ 금지
async function reviews() {}
async function reviewCreate() {}
async function review_delete() {}
```

---

## 📝 타입 정의 표준

### 1. 타입 파일 조직 (필수)

**types/ 디렉토리에 중앙화:**
- `types/user.ts` - User, SignUp/LogIn Request/Response
- `types/review.ts` - Review, Create/Update/Delete Request
- `types/notion.ts` - Notion OAuth, Database Info
- `types/auth.ts` - Session, Token, Auth State

### 2. 타입 작성 규칙

**명시적 타입 정의 필수 (any 금지):**
```typescript
// ✅ 올바른 예
interface User {
  id: string
  email: string
  createdAt: Date
}

type CreateUserRequest = {
  email: string
  password: string
  passwordConfirm: string
}

// ❌ 금지
type User = any
interface ApiResponse {
  data: any
  error: any
}
```

**API 응답 타입 표준화:**
```typescript
// ✅ 표준 응답 타입
interface ApiSuccess<T> {
  success: true
  data: T
  message?: string
}

interface ApiError {
  success: false
  error: {
    code: string
    message: string
  }
}

type ApiResponse<T> = ApiSuccess<T> | ApiError
```

**데이터베이스 모델:**
```typescript
// ✅ Review 타입 표준
interface Review {
  id: string
  userId: string
  bookTitle: string
  author: string
  publishedDate: Date | null
  content: string
  source: 'NOTION' | 'WEB'     // Enum 타입
  notionPageId: string | null
  createdAt: Date
  updatedAt: Date
}

interface CreateReviewRequest {
  bookTitle: string
  author: string
  publishedDate?: Date
  content: string
}
```

---

## ⚡ Server Actions 표준

### 1. 파일 위치 (필수)

**src/actions/ 디렉토리에 작성:**
```
src/actions/
├── auth.ts      # signUp, logIn, logOut, verifySession
├── reviews.ts   # getReviews, getReviewById, createReview, updateReview, deleteReview
└── notion.ts    # syncNotionReviews, getNotionDatabases
```

### 2. 작성 규칙

**'use server' 지시문 필수:**
```typescript
// ✅ 올바른 예
'use server'

import { createClient } from '@/lib/supabase'

export async function getReviews(userId: string): Promise<Review[]> {
  const supabase = createClient()
  const { data, error } = await supabase
    .from('reviews')
    .select('*')
    .eq('userId', userId)
    .order('createdAt', { ascending: false })

  if (error) throw new Error(error.message)
  return data
}

// ❌ 금지
export async function getReviews() {  // 인증 확인 없음
  return fetch('/api/reviews')  // 클라이언트 사이드 fetch 사용
}
```

**인증 확인 필수:**
```typescript
// ✅ 모든 Server Action에서 사용자 확인
export async function createReview(data: CreateReviewRequest): Promise<Review> {
  const { user } = await getCurrentUser()  // 인증 확인
  if (!user) throw new Error('Unauthorized')

  // 사용자 관련 데이터만 조회/생성
  const review = await supabaseClient
    .from('reviews')
    .insert([{ ...data, userId: user.id }])

  return review
}
```

---

## 📋 폼 처리 표준

### 1. React Hook Form + Zod (필수)

**검증 스키마는 Server Action 파일에 정의:**
```typescript
// ✅ src/actions/reviews.ts
import { z } from 'zod'

const CreateReviewSchema = z.object({
  bookTitle: z.string().min(1, '책 제목을 입력하세요'),
  author: z.string().min(1, '저자명을 입력하세요'),
  publishedDate: z.date().optional(),
  content: z.string().min(1, '리뷰 내용을 입력하세요'),
})

export async function createReview(formData: unknown) {
  const validated = CreateReviewSchema.parse(formData)
  // ...
}
```

**클라이언트 폼 컴포넌트:**
```typescript
// ✅ src/components/forms/ReviewForm.tsx
'use client'

import { useFormState } from 'react-dom'
import { createReview } from '@/actions/reviews'

export function ReviewForm() {
  const [state, formAction] = useFormState(createReview, null)

  return (
    <form action={formAction}>
      <input name="bookTitle" type="text" required />
      <input name="author" type="text" required />
      <textarea name="content" required />
      <button type="submit">저장</button>
      {state?.error && <div>{state.error}</div>}
    </form>
  )
}
```

---

## 🧪 테스트 규칙

### 1. 테스트 필수 항목

**다음 항목에 대해 Playwright E2E 테스트 필수:**
- 인증 플로우 (회원가입, 로그인, 로그아웃)
- 리뷰 CRUD 작업
- 노션 동기화 기능
- 보호 페이지 접근 제어

**테스트 위치:**
```
project-root/
└── e2e/
    ├── auth.spec.ts       # 인증 테스트
    ├── reviews.spec.ts    # 리뷰 CRUD 테스트
    └── notion-sync.spec.ts # 노션 동기화 테스트
```

---

## 🚫 절대 금지 사항

### 금지된 패턴

| 금지 사항 | 이유 |
|----------|------|
| **Pages Router 사용** | App Router만 사용 |
| **any 타입** | 반드시 명시적 타입 정의 |
| **클라이언트에서 직접 fetch** | Server Actions 또는 API 라우트 사용 |
| **환경 변수 하드코딩** | `.env.local` 또는 `lib/env.ts`에서 정의 |
| **비로그인 사용자 데이터 접근** | 모든 액션에서 인증 확인 필수 |
| **직접 데이터베이스 쿼리 (클라이언트)** | Server Actions에서만 DB 접근 |
| **Notion 토큰 클라이언트 저장** | 서버 사이드에서만 관리 |
| **비밀번호 평문 저장** | Supabase Auth 사용 또는 bcrypt 암호화 |
| **에러 메시지 숨김** | 사용자에게 명확한 에러 표시 |
| **Tailwind 클래스 중복** | `cn()` 유틸 사용 (tailwind-merge) |

### 금지된 코드 패턴

```typescript
// ❌ 절대 금지
any 타입 사용
const data: any = await fetch(...)

// ❌ 클라이언트에서 인증 정보 저장
localStorage.setItem('authToken', token)

// ❌ 환경 변수 하드코딩
const apiUrl = 'https://api.example.com'

// ❌ 비로그인 사용자 데이터 접근
export async function getReviews() {  // user 확인 없음
  return await supabase.from('reviews').select()
}

// ❌ 클라이언트에서 Supabase 접근
'use client'
const supabase = createClient()  // 서버 전용
const { data } = await supabase.from('reviews').select()
```

---

## 🔑 핵심 파일 상호작용 규칙

### 1. 인증 흐름

**필수 파일:**
- `lib/auth.ts` - 인증 유틸 (getCurrentUser, verifyToken)
- `actions/auth.ts` - Server Actions (signUp, logIn, logOut)
- `middleware.ts` - 라우트 보호
- `types/auth.ts` - 인증 타입

**수정 시 주의사항:**
- `middleware.ts` 수정 → `/app/(protected)` 레이아웃도 확인
- `actions/auth.ts` 수정 → 모든 인증 페이지 테스트 필수

### 2. 데이터 흐름

**Server Action → 클라이언트 컴포넌트:**
```
actions/reviews.ts (Server Action)
    ↓
컴포넌트 (useFormState 또는 useAction)
    ↓
UI 업데이트
```

**금지: 직접 fetch**
```typescript
// ❌ 금지
'use client'
const [data, setData] = useState()
useEffect(() => {
  fetch('/api/reviews').then(...)
}, [])
```

### 3. 노션 동기화

**필수 파일:**
- `lib/notion.ts` - Notion API 클라이언트
- `actions/notion.ts` - 동기화 Server Action
- `types/notion.ts` - Notion 타입

**동기화 트리거:**
- 로그인 후 (if notionToken exists)
- 30분 주기 (정기 동기화)
- 수동 트리거 (사용자 요청)

### 4. Supabase 접근

**서버 전용 (actions/, lib/):**
```typescript
// ✅ 서버 액션에서만 사용
'use server'
const supabase = createClient()
```

**클라이언트 제외:**
```typescript
// ❌ 클라이언트 컴포넌트에서 사용 금지
'use client'
const supabase = createClient()  // 금지
```

---

## 🏗️ 개발 단계별 수정 규칙

### Phase 1: 라우팅 및 타입 정의

**수정 대상:**
- `src/app/` 폴더 생성 및 라우팅 구조
- `src/types/` 파일 작성 및 타입 정의
- `middleware.ts` 인증 미들웨어

**주의사항:**
- 빈 페이지 파일도 생성 (나중 구현용)
- 타입 정의는 최대한 상세히

### Phase 2: UI/UX 개발

**수정 대상:**
- `src/components/` 컴포넌트 작성
- `src/app/*/page.tsx` 페이지 구현 (더미 데이터)
- 스타일링 (Tailwind CSS)

**주의사항:**
- Server Components로 구성
- 아직 데이터 연동 없음
- 반응형 디자인 필수

### Phase 3: 기능 구현

**수정 대상:**
- `src/actions/` Server Actions 작성
- `src/lib/` Supabase, Notion 클라이언트
- 컴포넌트에 실제 액션 연동
- 테스트 파일 작성

**주의사항:**
- 모든 액션에 인증 확인 필수
- 에러 처리 및 토스트 메시지
- Playwright 테스트 함께 작성

---

## 🔧 개발 도구 및 명령어

### 필수 명령어

```bash
npm run dev              # 개발 서버 실행 (Turbopack)
npm run build            # 프로덕션 빌드
npm run lint             # ESLint 점검
npm run lint:fix         # ESLint 자동 수정
npm run format           # Prettier 포맷팅
npm run format:check     # Prettier 점검
npm run typecheck        # TypeScript 타입 체크
npm run check-all        # 린트 + 타입 체크 + 포맷 점검 (필수)
npm run start            # 프로덕션 서버 실행
```

### Pre-commit Hook

**Husky + lint-staged 설정됨:**
- 커밋 전 자동으로 `npm run check-all` 실행
- 린트/포맷 오류 시 커밋 차단

**강제 적용:**
```bash
npm run check-all  # 반드시 통과해야 함
```

---

## 📊 데이터베이스 스키마

### User 테이블

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  passwordHash TEXT,
  notionToken TEXT,                -- 암호화됨
  notionDatabaseId TEXT,
  lastSyncedAt TIMESTAMP,
  createdAt TIMESTAMP DEFAULT NOW(),
  updatedAt TIMESTAMP DEFAULT NOW()
);
```

### Review 테이블

```sql
CREATE TABLE reviews (
  id UUID PRIMARY KEY,
  userId UUID NOT NULL REFERENCES users(id),
  bookTitle TEXT NOT NULL,
  author TEXT NOT NULL,
  publishedDate DATE,
  content TEXT NOT NULL,
  source TEXT NOT NULL,             -- 'NOTION' 또는 'WEB'
  notionPageId TEXT,
  createdAt TIMESTAMP DEFAULT NOW(),
  updatedAt TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_reviews_user_created ON reviews(userId, createdAt);
CREATE INDEX idx_reviews_user_source ON reviews(userId, source);
```

---

## 🎯 AI 의사결정 기준

### 컴포넌트 분류 판단

1. **순수 UI 컴포넌트?** → `components/ui/`
2. **페이지 레이아웃?** → `components/layout/`
3. **페이지 섹션?** → `components/sections/`
4. **폼 컴포넌트?** → `components/forms/`
5. **특정 기능?** → `components/{feature}/` (reviews/, modals/ 등)

### Server vs Client 컴포넌트 판단

1. **상호작용 필요?**
   - YES → `'use client'` (최소 범위)
   - NO → Server Component

2. **서버 데이터 필요?**
   - YES → Server Component
   - NO → 클라이언트 컴포넌트 (필요시)

### 파일 위치 판단

| 파일 타입 | 위치 | 예시 |
|----------|------|------|
| API 호출 | `actions/` | `actions/reviews.ts` |
| 클라이언트 상태 | `'use client'` 컴포넌트 | ReviewTable |
| 유틸 함수 | `lib/` | `lib/utils.ts` |
| 타입 정의 | `types/` | `types/review.ts` |
| 컴포넌트 | `components/` | `components/reviews/ReviewCard.tsx` |

---

## ✅ 코드 리뷰 체크리스트

**모든 PR/커밋 전에 확인:**

- [ ] TypeScript: `npm run typecheck` 통과
- [ ] Lint: `npm run lint` 통과 (0 error, 0 warning)
- [ ] Format: `npm run format:check` 통과
- [ ] Build: `npm run build` 성공
- [ ] Server Action에 인증 확인
- [ ] 폼 입력값 Zod 검증
- [ ] 에러 메시지 사용자에게 표시
- [ ] 보호 페이지 `middleware.ts` 등록
- [ ] 새 라우트 추가 시 레이아웃 파일 생성
- [ ] Playwright 테스트 작성 (API 변경 시)
- [ ] 주석: 한국어로 각 기능 설명
- [ ] 환경 변수: `.env.local`에서 정의, 코드에 하드코딩 금지

---

마지막 업데이트: 2026-03-03
