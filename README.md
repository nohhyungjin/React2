# 202130113 노형진
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


