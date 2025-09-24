# 202130113 노형진
## 2025-09-24 5주차
#### 6. Linking Between Pages

* **기본 내비게이션 방식**: `<Link>` 컴포넌트
* **특징**: 프리페칭 + 클라이언트 내비게이션

```tsx
import Link from 'next/link'

export default function PostList({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.slug}>
          <Link href={`/blog/${post.slug}`}>{post.title}</Link>
        </li>
      ))}
    </ul>
  )
}
```

---

#### 7. Route Props Helpers

* **PageProps**: 페이지 컴포넌트 props (`params`, `searchParams`)
* **LayoutProps**: 레이아웃 컴포넌트 props (`children`, named slots)

```tsx
// app/blog/[slug]/page.tsx
export default async function Page(props: PageProps<'/blog/[slug]'>) {
  const { slug } = props.params
  return <h1>Blog post: {slug}</h1>
}
```

---


#####  React vs Next.js 라우팅 비교

| 구분                | **React (CRA 기준)**                                | **Next.js**                                            |
| ----------------- | ------------------------------------------------- | ------------------------------------------------------ |
| **라우팅 기본 방식**     | 내장 라우터 없음 → `react-router-dom` 등 외부 라이브러리 필요      | **파일 시스템 기반 라우팅** (폴더/파일 구조 = URL 구조)                  |
| **설정 방법**         | `BrowserRouter`, `Routes`, `Route` 컴포넌트로 직접 정의    | `app` 디렉토리 안에 `page.tsx` 파일 추가 시 자동 라우팅                |
| **동적 라우트**        | `<Route path="/users/:id">` 같은 패턴 사용              | `[id]` 폴더명 사용 (`/users/[id]/page.tsx`)                 |
| **중첩 라우트**        | `<Route>` 안에 `<Route>` 중첩                         | 폴더 중첩으로 구현 (`/dashboard/settings`)                     |
| **Layout 구성**     | 별도 컴포넌트 만들어 각 페이지에서 불러와야 함                        | `layout.tsx` 파일로 라우트별 레이아웃 자동 적용, 중첩 지원                |
| **링크 이동**         | `<Link to="/about">About</Link>`                  | `<Link href="/about">About</Link>` (프리페칭 자동 지원)        |
| **코드 분할**         | `React.lazy`, `Suspense` 직접 설정 필요                 | 라우트 단위 코드 스플리팅 자동                                      |
| **SSR/SSG**       | 기본 지원 없음 (추가 라이브러리 필요)                            | 내장 지원 (`getServerSideProps`, `generateStaticParams` 등) |
| **검색 파라미터**       | `useParams`, `useSearchParams` (react-router-dom) | `searchParams` prop (서버) / `useSearchParams` (클라이언트)   |
| **라우트 보호 (Auth)** | `PrivateRoute` 같은 커스텀 HOC 작성                      | Middleware 또는 layout 단에서 제어 가능                         |

---

##### Next.js Pages Router vs App Router 비교

| 구분           | **Pages Router (`/pages`)**                                                                  | **App Router (`/app`)**                                                 |
| ------------ | -------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| **도입 시기**    | Next.js 초기부터 존재 (Next.js 12까지 기본)                                                            | Next.js 13에서 도입 (13\~14에서 안정화)                                          |
| **라우팅 방식**   | **파일 기반 라우팅**: `pages/index.tsx → /`, `pages/blog/[id].tsx → /blog/:id`                      | **폴더 기반 라우팅**: `app/page.tsx → /`, `app/blog/[id]/page.tsx → /blog/:id` |
| **렌더링 모델**   | 기본적으로 **Client-Side Rendering(CSR)** 중심, SSR/SSG는 `getServerSideProps`, `getStaticProps`로 처리 | **Server Components 기본 지원** → 서버에서 실행되는 컴포넌트와 클라이언트 컴포넌트를 명확히 구분        |
| **데이터 패칭**   | `getServerSideProps`, `getStaticProps`, `getInitialProps`                                    | `async function` 직접 사용 가능, `fetch()`가 서버에서 자동 캐싱/최적화                    |
| **레이아웃**     | 별도의 `_app.tsx`, `_document.tsx` 파일로 전체 레이아웃 구성                                               | **`layout.tsx` 파일**로 라우트별, 중첩 레이아웃을 자동 지원                               |
| **중첩 라우팅**   | 지원 없음 → 각 페이지마다 공통 UI를 직접 포함해야 함                                                             | 지원 → 상위 `layout.tsx`가 하위 라우트들을 감싸는 구조                                   |
| **코드 스플리팅**  | 페이지 단위 자동 분리                                                                                 | 페이지 + 레이아웃 단위까지 세분화된 자동 코드 스플리팅                                         |
| **라우트 그룹**   | 지원 없음                                                                                        | `(folder)` 형식으로 URL에 영향 없이 구조화 가능                                       |
| **에러/로딩 처리** | 글로벌 `_error.tsx`만 가능                                                                         | **`error.tsx`, `loading.tsx`, `not-found.tsx`** 등 라우트 단위 처리 가능          |
| **점진적 채택**   | Next.js 13 이상에서도 계속 사용 가능                                                                    | 권장 방식 (Next.js 팀의 미래 지향적 기본)                                            |


---

### Next.js Linking & Navigating

#### 1. 네비게이션 동작 원리

* **Server Rendering**

  * 기본적으로 페이지와 레이아웃은 **React Server Component**로 동작
  * 서버에서 **Server Component Payload**를 생성해 클라이언트로 전송
  * 두 가지 방식

    * **Static Rendering (Prerendering)** → 빌드 타임/재검증 시 캐싱
    * **Dynamic Rendering** → 요청 시마다 서버에서 생성
  * 단점: 클라이언트는 서버 응답을 기다려야 함

* **Prefetching**

  * `<Link>` 컴포넌트 사용 시 뷰포트에 들어오면 자동으로 **백그라운드 로딩**
  * **Static Route** → 전체 미리 불러옴
  * **Dynamic Route** → 스킵하거나 부분적(prefetch + `loading.tsx`)

* **Streaming**

  * 서버가 **준비된 부분부터 점진적으로 전송**
  * 사용자 입장에서 로딩 UI(스켈레톤)를 빠르게 보여주어 즉각 반응성을 제공
  * `loading.tsx`를 만들어서 활용
  * Next.js가 내부적으로 `<Suspense>` 처리 → 로딩 UI 먼저 보여주고, 이후 실제 콘텐츠 교체

* **Client-side transitions**

  * `<Link>`를 통한 네비게이션은 클라이언트에서 빠르게 처리
  * 서버 응답을 기다리는 지연을 최소화

---

##### 장점

* **빠른 탐색 경험**
  → 프리패칭 + 클라이언트 전환 덕분에 즉각적 반응
* **느린 네트워크에서도 개선된 UX**
  → Streaming으로 부분 로딩 제공
* **SEO 및 초기 렌더링 보장**
  → SSR로 초기 HTML 생성
* **Core Web Vitals 향상**
  → TTFB, FCP, TTI 개선

---

## 2025-09-17 4주차

### Layouts & Pages

#### 1. Pages

* **Page 정의**: 특정 라우트에서 렌더링되는 UI.
* **생성 방법**: `app` 디렉토리에 `page.tsx` 생성 후 React 컴포넌트 default export.

```tsx
// app/page.tsx
export default function Page() {
  return <h1>Hello Next.js!</h1>
}
```

---

#### 2. Layouts

* **Layout 정의**: 여러 페이지에서 공유되는 UI.
* **특징**: 네비게이션 시 상태 유지, 인터랙션 유지, 리렌더링 없음.
* **Root Layout**: `app/layout.tsx`는 필수이며 `<html>`, `<body>` 포함.

```tsx
// app/layout.tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <main>{children}</main>
      </body>
    </html>
  )
}
```

* **Nested Layouts**: 폴더별로 `layout.tsx` 추가 시 상위 → 하위로 중첩

---

#### 3. Nested Routes

* **폴더 = URL 세그먼트**
* **page.tsx = 해당 세그먼트의 UI**
* 예시: `/blog` → `app/blog/page.tsx`

```tsx
// app/blog/page.tsx
export default async function Page() {
  const posts = await getPosts()
  return (
    <ul>
      {posts.map((post) => (
        <Post key={post.id} post={post} />
      ))}
    </ul>
  )
}
```

* `/blog/[slug]` → 동적 라우트 (폴더명에 대괄호 사용)

```tsx
// app/blog/[slug]/page.tsx
export default function Page() {
  return <h1>Hello, Blog Post Page!</h1>
}
```

---

#### 4. Dynamic Segments

* **형식**: `[slug]`, `[id]` 등
* **데이터 기반으로 여러 페이지 자동 생성**

```tsx
// app/blog/[slug]/page.tsx
export default async function BlogPostPage({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug)
  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </div>
  )
}
```

---

##### slug란?

* **slug 정의**: URL에서 특정 자원을 구분하기 위해 사용되는 문자열
  예: `/blog/nextjs-routing-guide` → `nextjs-routing-guide`가 slug
* **용도**: DB의 id나 title을 기반으로 URL-friendly 문자열을 생성해 **가독성 좋고 SEO 친화적**인 경로 제공

---

##### slug 생성 예시

* 데이터 (DB)

  ```json
  {
    "id": 1,
    "title": "Next.js Routing Guide",
    "content": "..."
  }
  ```
* slug 변환: `"Next.js Routing Guide"` → `"nextjs-routing-guide"`
* 최종 URL: `/blog/nextjs-routing-guide`

---

##### getStaticPaths / generateStaticParams와 slug

Next.js에서 slug 기반 페이지를 미리 생성하려면 사용:

```tsx
// app/blog/[slug]/page.tsx
import { getPosts, getPost } from '@/lib/posts'

export async function generateStaticParams() {
  const posts = await getPosts()
  return posts.map((post) => ({ slug: post.slug }))
}

export default async function BlogPostPage({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug)
  return <h1>{post.title}</h1>
}
```

---

##### slug 장점

* **SEO 최적화**: URL에 의미 있는 키워드 포함 가능
* **가독성**: 숫자 ID보다 직관적 (예: `/blog/42` vs `/blog/nextjs-routing-guide`)
* **데이터 매핑**: DB 쿼리 시 `slug`를 key로 사용 가능


---

#### 5. Search Params

* **서버 컴포넌트**: `searchParams` prop 사용 (동적 렌더링)
* **클라이언트 컴포넌트**: `useSearchParams` hook 사용

```tsx
// Server Component
export default async function Page({ searchParams }: { searchParams: { [key: string]: string } }) {
  const filters = searchParams.filters
}
```

---

## 2025-09-10 3주차
### 프로젝트 구조
#### 폴더 및 파일 규칙
##### 1. 최상위 폴더

| 폴더       | 용도                         |
| -------- | -------------------------- |
| `app`    | App Router (페이지 및 레이아웃 관리) |
| `pages`  | Pages Router (기존 페이지 라우팅)  |
| `public` | 정적 자원 제공 (이미지, 폰트 등)       |
| `src`    | 선택적 애플리케이션 소스 폴더           |

---

##### 2. 최상위 파일

| 파일                   | 용도                                                          |
| -------------------- | ----------------------------------------------------------- |
| `next.config.js`     | Next.js 설정                                                  |
| `package.json`       | 의존성, 스크립트                                                   |
| `instrumentation.ts` | OpenTelemetry 설정                                            |
| `middleware.ts`      | 요청 미들웨어                                                     |
| `.env*`              | 환경 변수 (`.env.local`, `.env.development`, `.env.production`) |
| `.eslintrc.json`     | ESLint 설정                                                   |
| `.gitignore`         | Git 무시 파일                                                   |
| `tsconfig.json`      | TypeScript 설정                                               |
| `jsconfig.json`      | JavaScript 설정                                               |
| `next-env.d.ts`      | TypeScript 선언 파일                                            |

---

##### 3. 라우팅 파일

| 파일             | 확장자           | 용도                        |
| -------------- | ------------- | ------------------------- |
| `layout`       | .js/.jsx/.tsx | 공통 UI 레이아웃 (헤더, 네비, 푸터 등) |
| `page`         | .js/.jsx/.tsx | 페이지 내용, 공개 라우트            |
| `loading`      | .js/.jsx/.tsx | 로딩 스켈레톤 UI                |
| `not-found`    | .js/.jsx/.tsx | 404 페이지                   |
| `error`        | .js/.jsx/.tsx | 라우트 에러 처리                 |
| `global-error` | .js/.jsx/.tsx | 전역 에러 처리                  |
| `route`        | .js/.ts       | API 엔드포인트                 |
| `template`     | .js/.jsx/.tsx | 재렌더링 레이아웃                 |
| `default`      | .js/.jsx/.tsx | 병렬 라우트 기본 페이지             |

---

##### 4. 중첩 및 동적 라우트

* **폴더 구조** = URL 구조, `layout`은 하위 라우트 감싸기
* **공개 라우트**: `page.js` 또는 `route.js` 존재 시
* **동적 라우트**:

  * `[param]` : 단일 매개변수
  * `[...param]` : catch-all
  * `[[...param]]` : optional catch-all

예시:

* `/blog/[slug]/page.tsx` → `/blog/my-first-post`
* `/shop/[...slug]/page.tsx` → `/shop/clothing/shoes`

---

##### 5. Route Groups & Private 폴더

| 유형          | 표시         | 설명                      |
| ----------- | ---------- | ----------------------- |
| Route Group | `(folder)` | URL에는 포함되지 않고 레이아웃/공유용  |
| Private 폴더  | `_folder`  | 라우팅에 포함되지 않음, UI/유틸 보관용 |

---

##### 6. 병렬 및 인터셉트 라우트

| 패턴               | 의미         | 사용 사례      |
| ---------------- | ---------- | ---------- |
| `@folder`        | Named slot | 사이드바 + 메인  |
| `(.)folder`      | 동일 레벨 인터셉트 | 모달 미리보기    |
| `(..)folder`     | 상위 레벨 인터셉트 | 상위 자식 오버레이 |
| `(..)(..)folder` | 두 단계 인터셉트  | 깊은 중첩 오버레이 |
| `(...)folder`    | 루트 인터셉트    | 임의 라우트 렌더링 |

---

##### 7. 메타데이터 파일

###### 앱 아이콘

* `favicon.ico` : 파비콘
* `icon.*` : 앱 아이콘 (직접/생성 가능)
* `apple-icon.*` : Apple 아이콘 (직접/생성 가능)

###### Open Graph & Twitter 이미지

* `opengraph-image.*`, `twitter-image.*` : OG/Twitter용 이미지 (직접/생성 가능)

###### SEO

* `sitemap.xml/js` : 사이트맵
* `robots.txt/js` : 크롤러 설정

---
#### 프로젝트 구성

##### 1. 컴포넌트 계층 (Component Hierarchy)

Next.js는 특정 파일의 컴포넌트를 **정해진 계층 구조**로 렌더링합니다:

1. `layout.js` → 공통 레이아웃
2. `template.js` → 재렌더링 레이아웃
3. `error.js` → React 에러 경계
4. `loading.js` → React suspense 경계 (로딩 UI)
5. `not-found.js` → 404 페이지
6. `page.js` 또는 중첩된 `layout.js` → 실제 페이지 콘텐츠

* **중첩 라우트**: 자식 라우트의 컴포넌트는 부모 레이아웃 안에 재귀적으로 렌더링됨.

---

##### 2. Colocation (파일 동위 배치)

* `app` 디렉토리 안에서 폴더 구조 = 라우트 구조
* **공개 라우트 조건**: `page.js` 또는 `route.js` 존재 시만 접근 가능
* **장점**: route segment 안에 관련 파일을 안전하게 배치 가능 (라우트로 자동 노출되지 않음)
* 선택 사항: `app` 밖에 프로젝트 파일을 둬도 됨

---

##### 3. Private 폴더

* `_folderName`으로 폴더 이름 시작 → 라우팅 제외
* **용도**:

  * UI 로직과 라우팅 로직 분리
  * 내부 파일 일관성 유지
  * 코드 에디터에서 정리/그룹화
  * 미래 Next.js 파일 규칙 충돌 방지
* 참고:

  * 앱 밖에서도 `_` 접두사로 "비공개" 표시 가능
  * URL 세그먼트 시작 시 `%5F` 사용 가능 (`%5FfolderName`)

---

##### 4. Route Groups

* `(folderName)` → URL에는 포함되지 않고 **조직화 용도**
* **장점**:

  * 사이트 섹션, 의도, 팀 단위 라우트 그룹화 (예: 마케팅, 관리자 페이지)
  * 같은 레벨에서 여러 중첩 레이아웃 가능
  * 특정 라우트 그룹에만 레이아웃 적용 가능

---

##### 5. `src` 폴더

* 선택적 폴더로, 애플리케이션 코드(`app` 포함)를 루트 설정 파일과 분리
* 루트에는 주로 프로젝트 설정/구성 파일을 둠

---
정리하면, Next.js에서 **프로젝트 구성 전략과 라우트 그룹 활용**은 다음과 같이 요약할 수 있습니다.

---
#### 예제

##### 1. 프로젝트 파일 배치 전략

1. **`app` 외부에 프로젝트 파일 배치**

   * 앱 디렉토리는 순수 라우팅용으로 사용
   * 전역 공유 코드와 컴포넌트를 루트 폴더에 둠

2. **`app` 최상위 폴더에 프로젝트 파일 배치**

   * 모든 애플리케이션 코드와 공유 폴더를 `app` 최상위에 배치
   * 라우팅 구조와 함께 관리 가능

3. **기능(feature) 또는 라우트별 파일 분리**

   * 전역적으로 공유되는 코드는 `app` 루트에 배치
   * 특정 라우트에서만 사용되는 코드는 해당 route segment 폴더에 배치

---

##### 2. Route Groups 활용

* `(folder)` → URL에는 포함되지 않음, 조직화 용도
* **장점**:

  * 관련 라우트를 하나로 묶어 관리
  * 같은 URL 레벨에서 **여러 레이아웃** 적용 가능
  * 특정 라우트만 선택적으로 레이아웃 적용 가능 (opt-in layout)

###### 예시:

* `(shop)/cart/page.js` → `cart`만 `(shop)` 레이아웃 공유
* `(overview)/loading.js` → 대시보드의 overview 페이지에만 로딩 스켈레톤 적용

---

##### 3. 다중 루트 레이아웃

* 최상위 `layout.js` 제거 후 각 **Route Group마다 layout.js** 생성
* 목적: 애플리케이션 섹션별 완전히 다른 UI/경험 제공
* 주의: `<html>`과 `<body>` 태그를 각 루트 레이아웃에 추가해야 함

###### 예시:

* `(marketing)` → 마케팅 섹션 전용 레이아웃
* `(shop)` → 쇼핑 섹션 전용 레이아웃

---



## 2025-09-03 2주차

**부트스트랩(Bootstrap)**

* 오픈 소스 프론트엔드 프레임워크
* HTML, CSS, JavaScript 기반
* 반응형 레이아웃 지원
* 그리드 시스템 제공
* 버튼, 카드, 네비게이션 바 등 UI 컴포넌트 제공
* 일관된 디자인 구현 가능
* 커스터마이징 용이
* 빠른 프로토타이핑 적합

---

**Core Web Vitals**

* 구글이 정의한 웹사이트 사용자 경험 측정 지표
* 페이지 속도, 반응성, 시각적 안정성 평가

주요 항목

* **LCP (Largest Contentful Paint)**: 주요 콘텐츠가 로드되는 속도

* **FID (First Input Delay)**: 사용자 입력에 대한 첫 반응 속도

* **CLS (Cumulative Layout Shift)**: 예기치 않은 레이아웃 이동 정도

* 검색 순위에 직접적 영향

* 사용자 만족도와 이탈률에 큰 관련

---

**Next.js `src` 디렉토리**

* 루트에 `src/` 생성 가능
* `src/pages`, `src/app` 자동 인식
* 설정 필요 없음
* 소스 코드와 설정 파일 분리
* 프로젝트 구조 깔끔하게 유지

---

`eslintrc.json`

* 기존 방식 (레거시)
* JSON, YAML, JS 형식 지원
* 계층적 설정 가능 (프로젝트 루트 + 하위 폴더)
* 오래된 플러그인/가이드에서 주로 사용

`eslint.config.mjs`

* 새로운 방식 (Flat Config)
* ES Module 기반 (`.mjs`, `.js`)
* 배열 형태로 설정 → 직관적
* 계층적 머지 대신 명시적 구성
* 최신 ESLint 버전에서 권장

---

### `eslint.config.mjs`와 Next.js

* Next.js 13 이후 → ESLint 기본 지원
* 새로운 Flat Config(`eslint.config.mjs`) 방식과 호환
* 프로젝트 루트에 `eslint.config.mjs` 작성 가능
* Next.js 플러그인(`next/core-web-vitals`) 불러와 적용
* 목적: 최신 ESLint 규칙을 Next.js 환경에 맞게 적용

---

### Hard Link

* 같은 파일에 대한 또 다른 이름
* 동일한 inode 번호 공유
* 원본 삭제해도 다른 링크는 파일 유지
* 같은 파일 시스템 내에서만 생성 가능

### Symbolic Link (Soft Link)

* 원본 파일 경로를 가리키는 별도 파일
* 다른 inode 사용
* 원본 삭제 시 링크 깨짐
* 다른 파일 시스템에도 생성 가능

---


## 2025-08-27 1주차
### Getting Started

Next.js는 **React 기반의 풀스택 웹 프레임워크**로, Vercel에서 개발·유지하고 있음  
React를 단순히 프론트엔드 라이브러리로 사용하는 것과 달리, **서버 사이드 렌더링(SSR), 정적 사이트 생성(SSG), API 라우트, 이미지 최적화 등**을 지원하여 프로덕션 수준의 웹 애플리케이션을 빠르고 효율적으로 개발할 수 있음

---

#### Next.js의 주요 특징

##### 1. **서버 사이드 렌더링 (SSR)**

* React 기본은 클라이언트 사이드 렌더링(CSR)이지만, Next.js는 서버에서 HTML을 미리 만들어 전달할 수 있음
* 이를 통해 \*\*검색 엔진 최적화(SEO)\*\*가 좋아지고, 초기 로딩 속도도 빨라짐

##### 2. **정적 사이트 생성 (SSG)**

* 빌드 시점에 미리 HTML 파일을 생성해두고, 클라이언트 요청 시 바로 제공하는 방식
* 블로그, 문서 사이트처럼 **변경이 적고 빠른 응답**이 필요한 경우 적합

##### 3. **하이브리드 렌더링**

* 한 프로젝트 안에서 **SSR + SSG + CSR**을 필요에 맞게 혼합해서 사용가능
* 예: 상품 목록은 SSG로, 상품 상세는 SSR로, 장바구니는 CSR로 처리

##### 4. **파일 기반 라우팅**

* `pages/` 폴더 안에 파일을 만들면 자동으로 라우팅이 설정
* 예: `pages/about.js` → `/about`

##### 5. **API Routes**

* `pages/api/` 디렉토리에 파일을 만들면 서버리스 API 엔드포인트가 됨
* 프론트엔드와 백엔드를 같은 프로젝트에서 다룰 수 있음

##### 6. **이미지 최적화**

* `<Image>` 컴포넌트를 사용하면 자동으로 **최적화된 이미지 사이즈, WebP 변환, Lazy Loading**을 제공

##### 7. **스타일링 자유도**

* CSS, Sass, CSS-in-JS(Styled Components, Emotion), Tailwind CSS 등 자유롭게 선택 가능

##### 8. **App Router (13버전 이상)**

* `pages/` 대신 `app/` 디렉토리를 이용한 **새로운 라우팅 시스템**을 지원
* 서버 컴포넌트, 레이아웃(Layout), 병렬 라우팅 등 기능 제공
* 서버-클라이언트 경계가 명확해져서 데이터 패칭이 훨씬 깔끔해짐

---

#### Next.js Installation

##### 1. 시스템 요구사항

* Node.js 18.18 이상
* macOS, Windows(WSL 포함), Linux 지원

---

##### 2. 새 프로젝트 만들기 (빠른 시작)

```bash
npx create-next-app@latest my-app --yes
cd my-app
npm run dev
```

* 기본 설정: **TypeScript, Tailwind, App Router, Turbopack, @/* import alias*\*

실행 후 브라우저에서 [http://localhost:3000](http://localhost:3000) 접속

---

##### 3. 설치 시 CLI 옵션 (선택 가능)

* 프로젝트 이름
* TypeScript 여부
* Linter (ESLint / Biome / None)
* Tailwind 사용 여부
* `src/` 디렉토리 사용 여부
* App Router 사용 여부
* Turbopack 사용 여부
* import alias 설정

---

##### 4. 수동 설치 (원하는 설정 직접 구성할 때)

```bash
npm i next@latest react@latest react-dom@latest
```

`package.json`에 scripts 추가:

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "eslint",
    "lint:fix": "eslint --fix"
  }
}
```
<img src="img/스크린샷 2025-08-27 120634.png" alt="수동 설치하는 중">

---

##### 5. 기본 폴더 구조

* `app/layout.tsx` → **루트 레이아웃 (html, body 포함 필수)**
* `app/page.tsx` → **홈 페이지**

예시:

```tsx
// app/layout.tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  )
}

// app/page.tsx
export default function Page() {
  return <h1>Hello, Next.js!</h1>
}
```

---

##### 6. 정적 자산 (이미지, 폰트 등)

* `public/` 폴더 생성 → `/` 경로로 접근 가능
  예: `/public/profile.png` → 코드에서 `/profile.png`로 사용

---

##### 7. 개발 서버 실행

```bash
npm run dev
```

* 브라우저에서 결과 확인
* 코드 수정 후 저장 → 자동 반영

---

##### 8. TypeScript 설정

* `.ts` 또는 `.tsx` 파일 저장 → 자동으로 `tsconfig.json` 생성 & 필요한 패키지 설치됨

---

##### 9. Linting

* **ESLint**

  ```json
  {
    "scripts": {
      "lint": "eslint",
      "lint:fix": "eslint --fix"
    }
  }
  ```

* **Biome**

  ```json
  {
    "scripts": {
      "lint": "biome check",
      "format": "biome format --write"
    }
  }
  ```

---

##### 10. 절대 경로 & 모듈 별칭

* `tsconfig.json` 또는 `jsconfig.json` 수정

```json
{
  "compilerOptions": {
    "baseUrl": "src/",
    "paths": {
      "@/styles/*": ["styles/*"],
      "@/components/*": ["components/*"]
    }
  }
}
```

`../../../components/button` 대신 `@/components/button` 으로 import 가능

---


