# 노션 기반 북리뷰 개발 로드맵

사용자가 노션에서 작성한 책 리뷰를 웹 서비스에서 통합 관리하고 개인 라이브러리로 열람 및 수정할 수 있는 현대적 웹 애플리케이션

## 개요

노션 기반 북리뷰 서비스는 책을 자주 읽으며 리뷰를 남기고 싶은 개인 사용자를 위한 통합 북리뷰 관리 플랫폼입니다. 다음 핵심 기능을 제공합니다:

- **사용자 인증 (F001, F002)**: 이메일과 비밀번호를 통한 안전한 회원가입 및 로그인
- **노션 연동 (F003, F010)**: 노션 공유 데이터베이스와의 자동 동기화
- **리뷰 관리 (F004-F008)**: 리뷰 조회, 상세보기, 작성, 수정, 삭제 기능
- **세션 관리 (F009, F011)**: 로그아웃 및 인증 확인 기반 접근 제어

## 개발 워크플로우

### 1. 작업 계획
- 기존 코드베이스를 학습하고 현재 상태를 파악
- 새로운 작업을 포함하도록 `ROADMAP.md` 업데이트
- 우선순위 작업은 마지막 완료된 작업 다음에 삽입

### 2. 작업 생성
- 기존 코드베이스를 학습하고 현재 상태를 파악
- `/tasks` 디렉토리에 새 작업 파일 생성
- 명명 형식: `XXX-description.md` (예: `001-setup.md`)
- 고수준 명세서, 관련 파일, 수락 기준, 구현 단계 포함
- API/비즈니스 로직 작업 시 "## 테스트 체크리스트" 섹션 필수 포함 (Playwright MCP 테스트 시나리오 작성)

### 3. 작업 구현
- 작업 파일의 명세서를 따름
- 기능과 기능성 구현
- API 연동 및 비즈니스 로직 구현 시 Playwright MCP로 테스트 수행 필수
- 각 단계 후 작업 파일 내 단계 진행 상황 업데이트
- 구현 완료 후 Playwright MCP를 사용한 E2E 테스트 실행
- 테스트 통과 확인 후 다음 단계로 진행

### 4. 로드맵 업데이트
- 로드맵에서 완료된 작업을 ✅로 표시

---

## 개발 단계

### Phase 1: 애플리케이션 골격 구축

애플리케이션의 전체 구조와 라우팅, 기본 타입 정의를 먼저 구성하여 다른 Phase들이 병렬로 진행될 수 있는 기초를 마련합니다.

#### Task 001: 프로젝트 구조 및 라우팅 설정 - 우선순위

Next.js App Router를 기반으로 모든 주요 페이지의 라우트 구조를 생성하고 공통 레이아웃 골격을 구현합니다.

**구현 사항:**
- `/app` 디렉토리 구조 생성: `(auth)` (로그인/회원가입), `(protected)` (인증 필수 페이지), 공개 페이지
- 라우트 그룹별 레이아웃 파일 생성 (`layout.tsx`)
  - Root 레이아웃: 네비게이션 바, 전역 스타일링 기초
  - Auth 레이아웃: 로그인/회원가입 공통 레이아웃
  - Protected 레이아웃: 인증 필수 페이지 공통 레이아웃 및 미들웨어 기초
- 모든 페이지 파일 생성 (빈 껍데기):
  - `/app/page.tsx` (랜딩 페이지)
  - `/app/(auth)/signup/page.tsx` (회원가입)
  - `/app/(auth)/login/page.tsx` (로그인)
  - `/app/(protected)/reviews/page.tsx` (리뷰 목록)
  - `/app/(protected)/reviews/[id]/page.tsx` (리뷰 상세)
  - `/app/(protected)/reviews/create/page.tsx` (리뷰 작성)
  - `/app/(protected)/reviews/[id]/edit/page.tsx` (리뷰 수정)
  - `/app/(protected)/settings/notion/page.tsx` (노션 연동 설정)
- 공통 네비게이션 컴포넌트 골격 (`components/common/Navigation.tsx`)
  - 비로그인 사용자 메뉴: 로그인, 회원가입 링크
  - 로그인 사용자 메뉴: 리뷰 목록, 새 리뷰 작성, 노션 연동, 로그아웃 링크
- `next.config.ts` 및 `tsconfig.json` 초기화 (필요 시 조정)

#### Task 002: 타입 정의 및 인터페이스 설계

TypeScript 인터페이스 및 타입 정의 파일을 생성하여 전체 애플리케이션의 타입 안전성을 보장합니다.

**구현 사항:**
- `/types` 디렉토리 생성 및 타입 파일 구성:
  - `types/user.ts`: User 인터페이스, 회원가입/로그인 요청/응답 타입
  - `types/review.ts`: Review 인터페이스, 리뷰 조회/생성/수정/삭제 요청/응답 타입
  - `types/notion.ts`: Notion 연동 관련 타입 (OAuth 토큰, 데이터베이스 정보)
  - `types/auth.ts`: 인증 관련 타입 (세션, 토큰)
- 데이터베이스 스키마 설계 (마크다운 문서): `docs/database-schema.md`
  - User 테이블 스키마: id, email, passwordHash, notionToken, notionDatabaseId, lastSyncedAt, createdAt, updatedAt
  - Review 테이블 스키마: id, userId, bookTitle, author, publishedDate, content, source, notionPageId, createdAt, updatedAt
  - 관계: User 1:N Review (userId 외래키)
- API 응답 타입 정의:
  - 성공 응답 형식: `{ success: true, data: T, message?: string }`
  - 에러 응답 형식: `{ success: false, error: { code: string, message: string } }`
- 환경 변수 타입 정의: `.env.local` 구조 및 필요한 환경 변수 목록 문서화

---

### Phase 2: UI/UX 완성 (더미 데이터 활용)

모든 페이지의 UI를 더미 데이터로 완성하여 사용자 플로우를 검증합니다. 이 Phase에서는 실제 데이터 연동 없이 하드코딩된 더미 데이터를 사용합니다.

#### Task 003: 공통 컴포넌트 라이브러리 구현 - 우선순위

shadcn/ui를 기반으로 프로젝트에서 반복적으로 사용되는 공통 컴포넌트를 구현합니다.

**구현 사항:**
- shadcn/ui 컴포넌트 설치 및 구성:
  - 필수 컴포넌트: Button, Input, Form, Dialog, Card, Table, Textarea, Select, Alert, Skeleton
  - 아이콘 라이브러리: Lucide React 아이콘 통합
- 프로젝트 커스텀 컴포넌트 구현 (`/components`):
  - `components/ui/`: shadcn/ui 기본 컴포넌트 (설치 후 생성)
  - `components/common/`: 프로젝트 커스텀 컴포넌트
    - `Navigation.tsx` (Phase 1에서 기초 구현, 이제 완성)
    - `Header.tsx` (로고, 사용자 메뉴, 로그아웃 버튼)
    - `Sidebar.tsx` (좌측 네비게이션)
    - `Footer.tsx` (풋터)
  - `components/forms/`: 폼 컴포넌트
    - `SignupForm.tsx` (회원가입 폼)
    - `LoginForm.tsx` (로그인 폼)
    - `ReviewForm.tsx` (리뷰 작성/수정 폼)
    - `NotionConnectForm.tsx` (노션 연동 폼)
  - `components/reviews/`: 리뷰 관련 컴포넌트
    - `ReviewCard.tsx` (리뷰 카드 표시)
    - `ReviewTable.tsx` (리뷰 목록 테이블)
    - `ReviewDetail.tsx` (리뷰 상세 정보)
    - `ReviewActions.tsx` (수정/삭제 액션 버튼)
  - `components/modals/`: 모달 컴포넌트
    - `ConfirmDeleteModal.tsx` (삭제 확인 모달)
    - `ErrorModal.tsx` (에러 메시지 모달)
- Tailwind CSS v4 스타일링:
  - `tailwind.config.ts` 설정 (shadcn/ui와 호환성 유지)
  - 기본 색상, 간격, 타이포그래피 정의
  - 반응형 디자인 클래스 적용
- 디자인 시스템 문서: `docs/design-system.md`
  - 색상 팔레트, 타이포그래피, 간격 규칙
  - 컴포넌트 사용 가이드라인
- 더미 데이터 생성 유틸리티:
  - `/utils/dummy-data.ts`: 테스트용 User, Review 객체 생성 함수
  - 더미 데이터 샘플: 5-10개의 샘플 리뷰 데이터

#### Task 004: 모든 페이지 UI 완성 (더미 데이터 활용)

모든 페이지를 더미 데이터로 완성하여 전체 사용자 플로우를 구성합니다.

**구현 사항:**
- 랜딩 페이지 (`/app/page.tsx`)
  - 서비스 소개 섹션 (제목, 설명, 주요 기능)
  - CTA 버튼: "로그인", "회원가입" (더미 링크)
  - 반응형 레이아웃
- 회원가입 페이지 (`/app/(auth)/signup/page.tsx`)
  - SignupForm 컴포넌트 통합 (React Hook Form + Zod 유효성 검사 로직은 미포함, UI만 구현)
  - 입력 필드: 이메일, 비밀번호, 비밀번호 확인
  - 서버 액션 호출 시뮬레이션 (실제 기능은 Phase 3에서)
  - "로그인 페이지로 이동" 링크
- 로그인 페이지 (`/app/(auth)/login/page.tsx`)
  - LoginForm 컴포넌트 통합
  - 입력 필드: 이메일, 비밀번호
  - 서버 액션 호출 시뮬레이션
  - "회원가입 페이지로 이동" 링크
- 리뷰 목록 페이지 (`/app/(protected)/reviews/page.tsx`)
  - Header와 Navigation 컴포넌트 통합
  - "새 리뷰 작성" 버튼
  - ReviewTable 컴포넌트: 더미 리뷰 데이터 표시 (책 제목, 저자, 작성일, 별점)
  - 각 리뷰마다 "상세보기", "수정", "삭제" 액션 버튼
  - 로그아웃 버튼 (헤더)
  - 페이지네이션 (더미 데이터로 표시)
- 리뷰 상세 페이지 (`/app/(protected)/reviews/[id]/page.tsx`)
  - ReviewDetail 컴포넌트: 더미 리뷰 내용 표시
  - "수정" 버튼 → 리뷰 수정 페이지로 이동
  - "삭제" 버튼 → 삭제 확인 모달
  - "목록으로 돌아가기" 버튼
- 리뷰 작성 페이지 (`/app/(protected)/reviews/create/page.tsx`)
  - ReviewForm 컴포넌트 (새 리뷰 작성 모드)
  - 입력 필드: 책 제목, 저자, 출판일(옵션), 리뷰 내용
  - "저장" 버튼 (서버 액션 호출 시뮬레이션)
  - "취소" 버튼 → 리뷰 목록으로 이동
- 리뷰 수정 페이지 (`/app/(protected)/reviews/[id]/edit/page.tsx`)
  - ReviewForm 컴포넌트 (수정 모드): 더미 데이터로 기존값 표시
  - 입력 필드: 책 제목, 저자, 출판일, 리뷰 내용 (기존값으로 pre-fill)
  - "저장" 버튼
  - "취소" 버튼 → 이전 페이지로 복귀
- 노션 연동 설정 페이지 (`/app/(protected)/settings/notion/page.tsx`)
  - NotionConnectForm 컴포넌트
  - "노션과 연동하기" 버튼 (더미 클릭 핸들러)
  - 연동 상태 표시 (더미: "연동됨", "미연동" 중 표시)
  - 공유 데이터베이스 선택 드롭다운 (더미 옵션)
  - "동기화 시작" 버튼 (더미 로딩 상태)
- 공통 레이아웃 완성:
  - 모든 인증 페이지에 Header, Navigation 컴포넌트 적용
  - 페이지별 타이틀 및 브레드크럼 네비게이션
- 반응형 디자인:
  - 모바일 (< 640px), 태블릿 (640px-1024px), 데스크톱 (> 1024px) 레이아웃
  - 모바일 메뉴 (Sidebar 숨김/표시 토글)
- 에러 및 로딩 상태:
  - `error.tsx` 파일 생성 (에러 바운더리)
  - `loading.tsx` 파일 생성 (로딩 스켈레톤)
  - ErrorModal 컴포넌트로 에러 메시지 표시
- 사용자 플로우 검증:
  - 모든 페이지 간의 네비게이션 확인 (더미 링크로)
  - 사용자 여정 흐름도 대로 페이지 구성 확인
  - 모바일/태블릿/데스크톱에서 UI 확인

---

### Phase 3: 핵심 기능 구현

데이터베이스 연동, API 개발, 인증 시스템을 구현하여 실제 기능을 완성합니다.

#### Task 005: 데이터베이스 및 API 개발 - 우선순위

Supabase PostgreSQL 데이터베이스를 구축하고 RESTful API 엔드포인트를 구현합니다.

**구현 사항:**
- Supabase 프로젝트 생성 및 설정:
  - PostgreSQL 데이터베이스 초기화
  - Row Level Security (RLS) 정책 설정
- 데이터베이스 마이그레이션 스크립트 작성:
  - User 테이블 생성: id (UUID, PK), email (String, Unique, NOT NULL), passwordHash (String), notionToken (String, Encrypted), notionDatabaseId (String), lastSyncedAt (DateTime), createdAt (DateTime), updatedAt (DateTime)
  - Review 테이블 생성: id (UUID, PK), userId (UUID, FK → User.id), bookTitle (String), author (String), publishedDate (Date), content (Text), source (Enum: NOTION, WEB), notionPageId (String), createdAt (DateTime), updatedAt (DateTime)
  - 인덱스 생성: (userId, createdAt), (userId, source)
- Next.js Server Actions 구현 (API 라우트 대신 사용):
  - `/app/actions/auth.ts`: 회원가입, 로그인, 로그아웃, 세션 확인 액션
  - `/app/actions/reviews.ts`: 리뷰 CRUD 액션
  - `/app/actions/notion.ts`: 노션 연동, 동기화 액션
- 더미 데이터 마이그레이션을 실제 API 호출로 교체:
  - Phase 2의 더미 데이터를 실제 Supabase 쿼리로 변경
  - 클라이언트 컴포넌트에서 서버 액션 호출
- Supabase JS 클라이언트 통합:
  - `/lib/supabase.ts`: Supabase 클라이언트 인스턴스
  - 환경 변수 설정: NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_ANON_KEY
- 에러 핸들링 및 유효성 검사:
  - API 응답 에러 처리 (400, 401, 403, 500)
  - 데이터 유효성 검사 (Zod 스키마)
- 테스트 체크리스트:
  - [ ] Supabase에 User 테이블 생성 확인
  - [ ] Supabase에 Review 테이블 생성 확인
  - [ ] 회원가입 서버 액션 동작 확인 (Playwright)
  - [ ] 로그인 서버 액션 동작 확인 (Playwright)
  - [ ] 리뷰 조회 API 동작 확인 (curl 또는 Playwright)
  - [ ] 리뷰 생성 API 동작 확인
  - [ ] 리뷰 수정 API 동작 확인
  - [ ] 리뷰 삭제 API 동작 확인

#### Task 006: 인증 및 권한 시스템 구현

Supabase Auth를 활용하여 안전한 사용자 인증 및 접근 제어를 구현합니다.

**구현 사항:**
- Supabase Auth 설정:
  - 이메일/비밀번호 인증 활성화
  - 환경 변수 설정: SUPABASE_JWT_SECRET (서버 사이드)
- 인증 미들웨어 구현:
  - `/middleware.ts`: 요청 인증 확인 및 보호된 라우트 접근 제어
  - 비로그인 사용자 → 로그인 페이지로 리디렉션
  - 로그인 사용자 → 보호된 라우트 접근 허용
- 세션 관리:
  - Supabase 세션 쿠키 활용
  - `/lib/auth.ts`: 세션 생성, 검증, 삭제 유틸리티
  - `useAuth()` 훅 구현: 현재 사용자 정보 조회
- NextAuth.js 또는 Supabase Auth 통합 (선택):
  - 옵션 1: Supabase Auth 직접 사용 (권장)
  - 옵션 2: NextAuth.js + Supabase 어댑터
- 회원가입 기능 구현 (`/app/actions/auth.ts`):
  - 이메일 유효성 검사 (Zod)
  - 비밀번호 강도 검증 (최소 8자, 대소문자/숫자/특수문자 포함)
  - 중복 이메일 확인
  - User 레코드 생성 및 인증 토큰 발급
- 로그인 기능 구현:
  - 이메일 형식 검증
  - 비밀번호 확인 (bcrypt 또는 Supabase 내장)
  - 인증 토큰 발급 및 쿠키에 저장
  - 로그인 후 노션 연동 확인 (연동 안 됨 → 노션 연동 페이지로 리디렉션)
- 로그아웃 기능 구현:
  - 현재 세션 삭제
  - 쿠키 삭제
  - 로그인 페이지로 리디렉션
- 인증 확인 (F011):
  - 미들웨어에서 모든 보호된 라우트 접근 제어
  - 유효하지 않은 토큰 → 로그인 페이지로 리디렉션
- 보안 설정:
  - HTTPS 기본 (개발 환경 제외)
  - CSRF 토큰 구현 (Next.js 15 기본)
  - 민감한 정보(비밀번호) 클라이언트로 전송 금지
  - 환경 변수 보호 (NEXT_PUBLIC_ 접두사 확인)
- 테스트 체크리스트:
  - [ ] 회원가입 후 인증 토큰 발급 확인 (Playwright)
  - [ ] 로그인 성공 후 세션 생성 확인
  - [ ] 잘못된 비밀번호로 로그인 실패 확인
  - [ ] 비로그인 상태에서 보호된 라우트 접근 차단 확인
  - [ ] 로그아웃 후 세션 삭제 확인
  - [ ] 유효하지 않은 토큰으로 API 호출 차단 확인

#### Task 007: 노션 API 연동 및 동기화 구현

Notion API를 통해 사용자의 노션 공유 데이터베이스와 리뷰 데이터를 동기화합니다.

**구현 사항:**
- Notion API 클라이언트 설정:
  - `@notionhq/client` 라이브러리 설치
  - `/lib/notion.ts`: Notion 클라이언트 인스턴스 및 유틸리티
- Notion OAuth 인증 구현:
  - Notion 앱 생성 및 OAuth 설정
  - Notion OAuth 콜백 처리 (`/app/api/auth/notion/callback`)
  - 사용자 notionToken 저장 (암호화)
- 노션 데이터베이스 자동 감지:
  - 사용자의 Notion 계정에서 공유된 데이터베이스 목록 조회
  - 북리뷰 데이터베이스 식별 (필드 구조 기반)
- 노션 → 웹 동기화:
  - `/app/actions/notion.ts`에 `syncNotionReviews()` 액션 구현
  - Notion 데이터베이스에서 리뷰 페이지 조회
  - 각 페이지를 Review 레코드로 변환 (bookTitle, author, content 등)
  - 새로운 리뷰 생성, 기존 리뷰 업데이트 (notionPageId 기반)
  - 웹에서 삭제된 리뷰는 Notion에서 삭제하지 않음 (한 방향 동기화)
- 자동 동기화 스케줄 (F010):
  - 사용자 로그인 후 자동 동기화 트리거
  - 정기 동기화: 30분마다 자동 실행 (예약 작업 또는 클라이언트 사이드 폴링)
  - `lastSyncedAt` 필드 업데이트
- 노션 데이터 필드 매핑:
  - Notion 페이지 → Review 모델 매핑 규칙 문서화
  - 지원하는 필드: 제목, 저자, 출판일, 본문/설명
- 오류 처리:
  - Notion API 오류 (401, 403, 429) 처리
  - 네트워크 오류 재시도 로직
  - 사용자에게 동기화 실패 알림
- 테스트 체크리스트:
  - [ ] Notion OAuth 인증 완료 확인 (Playwright)
  - [ ] 노션 공유 데이터베이스 감지 확인
  - [ ] 노션 리뷰를 웹에 동기화 확인
  - [ ] 새로운 노션 리뷰 추가 후 동기화 확인
  - [ ] 기존 노션 리뷰 수정 후 동기화 확인
  - [ ] 자동 동기화 작동 확인

#### Task 008: 리뷰 CRUD 기능 완성 및 통합 테스트

리뷰 조회, 생성, 수정, 삭제 기능을 완성하고 전체 사용자 플로우에 대한 E2E 테스트를 수행합니다.

**구현 사항:**
- 리뷰 조회 (F004):
  - `/app/actions/reviews.ts`에 `getReviews()` 액션 구현
  - 현재 사용자의 모든 리뷰 조회
  - 페이지네이션 지원 (10개씩)
  - 정렬 옵션: 최신순, 오래된순
  - 리뷰 목록 페이지에서 테이블 표시
- 리뷰 상세 조회 (F005):
  - `/app/actions/reviews.ts`에 `getReviewById()` 액션 구현
  - 특정 리뷰 상세 정보 조회
  - 리뷰 상세 페이지에서 표시
- 리뷰 생성 (F006):
  - `/app/actions/reviews.ts`에 `createReview()` 액션 구현
  - 입력 필드 검증 (Zod): bookTitle (필수), author (필수), publishedDate (옵션), content (필수)
  - Review 레코드 생성 (userId, source='WEB')
  - 생성 후 리뷰 목록 페이지로 리디렉션
- 리뷰 수정 (F007):
  - `/app/actions/reviews.ts`에 `updateReview()` 액션 구현
  - 리뷰 소유권 확인 (userId 일치)
  - 필드 업데이트 (bookTitle, author, publishedDate, content)
  - updatedAt 필드 업데이트
  - 수정 후 리뷰 상세 페이지로 리디렉션
- 리뷰 삭제 (F008):
  - `/app/actions/reviews.ts`에 `deleteReview()` 액션 구현
  - 리뷰 소유권 확인
  - Review 레코드 삭제 (source='WEB'인 경우만)
  - Notion에서 생성된 리뷰는 웹에서 삭제 불가 (또는 soft delete)
  - 삭제 후 리뷰 목록 페이지로 리디렉션
- 프론트엔드 통합:
  - ReviewForm 컴포넌트에서 실제 서버 액션 호출
  - 로딩 상태 표시 (pending 상태)
  - 에러 메시지 표시
  - 성공 메시지 표시
- 권한 검증:
  - 모든 리뷰 CRUD 액션에서 사용자 인증 확인
  - 다른 사용자의 리뷰는 조회/수정/삭제 불가
- 테스트 체크리스트 (Playwright E2E):
  - [ ] 로그인 → 리뷰 목록 페이지 이동 확인
  - [ ] 새 리뷰 작성 → 리뷰 목록에서 표시 확인
  - [ ] 리뷰 상세 페이지에서 전체 내용 표시 확인
  - [ ] 리뷰 수정 → 변경사항 저장 확인
  - [ ] 리뷰 삭제 → 삭제 확인 모달 표시 및 삭제 완료 확인
  - [ ] 다른 사용자의 리뷰 수정 시도 → 접근 거부 확인
  - [ ] 노션에서 가져온 리뷰와 웹에서 작성한 리뷰 혼합 표시 확인
  - [ ] 리뷰 목록 페이지네이션 동작 확인

---

### Phase 4: 고급 기능 및 최적화

실시간 기능, 성능 최적화, 배포 파이프라인을 구축합니다.

#### Task 009: 부가 기능 및 사용자 경험 향상

추가적인 사용자 편의 기능과 실시간 업데이트 기능을 구현합니다.

**구현 사항:**
- 실시간 동기화 알림:
  - 자동 동기화 중 로딩 표시
  - 동기화 완료 후 알림 (토스트 메시지)
- 리뷰 검색 및 필터링:
  - 리뷰 목록에서 검색 기능 (bookTitle, author)
  - 필터링: 출처 (Notion, Web), 작성일 범위
- 북마크 기능 (선택사항):
  - 자주 읽는 리뷰 북마크
- 리뷰 내보내기 (선택사항):
  - 리뷰를 Markdown, PDF로 내보내기
- 다크 모드 지원:
  - Tailwind CSS 다크 모드 설정
  - 사용자 선호도 저장 (localStorage)

#### Task 010: 성능 최적화 및 배포 준비

성능 최적화, 모니터링, 배포 파이프라인을 구축합니다.

**구현 사항:**
- 성능 최적화:
  - 이미지 최적화 (Next.js Image 컴포넌트)
  - 코드 스플리팅 및 동적 임포트
  - 캐싱 전략 (ISR, 동적 라우트 캐싱)
  - Lighthouse 점수 개선 (90 이상 목표)
- 테스트 코드 작성:
  - 유닛 테스트: 유틸리티 함수 (Jest)
  - 통합 테스트: API 엔드포인트 (Playwright)
  - E2E 테스트: 전체 사용자 플로우 (Playwright)
- CI/CD 파이프라인:
  - GitHub Actions 설정
  - PR 생성 시 자동 린트, 타입 체크, 테스트 실행
  - main 브랜치에 병합 시 자동 배포
- 모니터링 및 로깅:
  - Sentry 또는 LogRocket 연동 (에러 모니터링)
  - 사용자 분석: Google Analytics 또는 Mixpanel
- 환경 변수 관리:
  - `.env.local` (로컬), `.env.production` (프로덕션) 설정
  - Vercel 환경 변수 설정
- Vercel 배포:
  - 프로젝트를 Vercel에 연결
  - 자동 배포 설정
  - 도메인 설정
  - SSL 인증서 자동 갱신

---

## 개발 환경 설정 체크리스트

### 필수 설정
- [ ] Node.js 20+ 설치 확인
- [ ] npm/yarn/pnpm 설정 확인
- [ ] 프로젝트 의존성 설치: `npm install`
- [ ] 개발 서버 실행: `npm run dev`
- [ ] ESLint 및 Prettier 설정 확인: `npm run lint`
- [ ] 빌드 확인: `npm run build`

### Supabase 설정
- [ ] Supabase 계정 생성
- [ ] Supabase 프로젝트 생성
- [ ] 환경 변수 설정 (`.env.local`):
  - `NEXT_PUBLIC_SUPABASE_URL`
  - `NEXT_PUBLIC_SUPABASE_ANON_KEY`
  - `SUPABASE_SERVICE_ROLE_KEY` (선택사항, 서버 사이드)

### Notion API 설정
- [ ] Notion 개발자 계정 생성
- [ ] Notion 앱 생성 및 OAuth 설정
- [ ] 환경 변수 설정 (`.env.local`):
  - `NEXT_PUBLIC_NOTION_CLIENT_ID`
  - `NOTION_CLIENT_SECRET`
  - `NOTION_OAUTH_REDIRECT_URI`

### Vercel 배포 (선택사항)
- [ ] Vercel 계정 생성
- [ ] GitHub 계정 연결
- [ ] 프로젝트를 Vercel에 연결
- [ ] 환경 변수 설정 (Vercel 대시보드)

---

## 기술 스택 요약

| 분류 | 기술 |
|------|------|
| **프레임워크** | Next.js 15.5.3, React 19.1.0 |
| **언어** | TypeScript 5.6+ |
| **스타일링** | TailwindCSS v4, shadcn/ui |
| **폼** | React Hook Form 7.x, Zod |
| **인증** | Supabase Auth |
| **데이터베이스** | Supabase (PostgreSQL) |
| **API 통합** | Notion API (@notionhq/client) |
| **캐싱** | node-cache (Notion 동기화 데이터) |
| **배포** | Vercel |
| **개발 도구** | ESLint, Prettier, Husky, lint-staged |

---

## 우선순위 및 의존성

```
Phase 1: 애플리케이션 골격 구축
├── Task 001: 프로젝트 구조 및 라우팅 설정 (필수 - 모든 작업의 기초)
├── Task 002: 타입 정의 및 인터페이스 설계 (필수 - 전체 타입 안전성)
│
Phase 2: UI/UX 완성 (Phase 1 완료 후)
├── Task 003: 공통 컴포넌트 라이브러리 구현 (필수 - Phase 2의 기초)
│   └── Task 004: 모든 페이지 UI 완성 (Task 003에 의존)
│
Phase 3: 핵심 기능 구현 (Phase 2 완료 후)
├── Task 005: 데이터베이스 및 API 개발 (필수 - Task 006, 007, 008의 기초)
├── Task 006: 인증 및 권한 시스템 구현 (Task 005에 의존)
├── Task 007: 노션 API 연동 및 동기화 구현 (Task 005, 006에 의존)
└── Task 008: 리뷰 CRUD 기능 완성 및 통합 테스트 (Task 005, 006, 007에 의존)
│
Phase 4: 고급 기능 및 최적화 (Phase 3 완료 후)
├── Task 009: 부가 기능 및 사용자 경험 향상 (Task 008에 의존)
└── Task 010: 성능 최적화 및 배포 준비 (전체에 의존)
```

---

## 참고 문서

- [PRD (Product Requirements Document)](./PRD.md)
- [프로젝트 구조](./guides/project-structure.md)
- [스타일링 가이드](./guides/styling-guide.md)
- [컴포넌트 패턴](./guides/component-patterns.md)
- [Next.js 15 전문 가이드](./guides/nextjs-15.md)
- [폼 처리 완전 가이드](./guides/forms-react-hook-form.md)
- [데이터베이스 스키마](./database-schema.md)
- [디자인 시스템](./design-system.md)

---

마지막 업데이트: 2026-03-02
