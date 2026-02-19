---
template: "TIL"
title: "Next.js 핵심 개념: 렌더링, 라우팅, 최적화"
created_at: "2026-02-19 11:12"
created_by:
  name: "gbpark"
  email: "gbpark@herit.net"
participants: []
tags:
  - "Next.js"
  - "렌더링"
  - "Pre-rendering"
  - "ISR"
  - "Page Router"
category: "TIL"
status: "draft"
visibility: "internal"
related_docs: []
custom: {}
---

# TIL (Today I Learned)

## 📚 학습 주제
웹 렌더링 방식 (CSR, Pre-rendering), Next.js의 주요 기능 (Page Router, SSG, ISR, SEO), 프레임워크와 라이브러리 개념 비교

---

## 🔑 핵심 내용
-   **프레임워크 vs 라이브러리**: 프레임워크는 개발의 주도권이 프레임워크에 있어 정해진 구조 안에서 개발하며, 라이브러리는 개발자에게 주도권이 있어 필요한 기능을 선택적으로 활용합니다.
-   **CSR의 초기 성능 한계**: Client-Side Rendering (CSR)은 초기 진입 시 빈 HTML을 받아 JavaScript 번들을 다운로드하고 렌더링하기까지 시간이 소요되어 First Contentful Paint (FCP)가 느릴 수 있습니다.
-   **Next.js의 Pre-rendering**: Next.js는 React의 CSR 단점을 보완하기 위해 사전 렌더링(Pre-rendering) 방식을 채택합니다. 서버에서 HTML을 미리 생성하여 클라이언트에 전송하고, 이후 JavaScript를 통해 상호작용 가능한 상태로 만듭니다 (Hydration).
-   **Next.js Page Router**: 파일 시스템 기반의 직관적인 라우팅을 제공하며, 정적 라우팅뿐만 아니라 동적 라우팅, Catch-all Segment, Optional Catch-all Segment를 지원합니다.
-   **ISR (Incremental Static Regeneration)**: SSG 방식으로 생성된 페이지의 데이터가 오래될 수 있는 단점을 보완하기 위해, 일정 시간(revalidate) 또는 요청 시점에 페이지를 증분적으로 재생성하는 기능입니다.

---

## 📝 상세 내용
### 🌟 프레임워크 vs 라이브러리
개발에 사용되는 도구의 주도권과 자유도에 따라 다음과 같이 구분할 수 있습니다.

| 특징         | 프레임워크 (Framework)                                     | 라이브러리 (Library)                                          |
| :----------- | :--------------------------------------------------------- | :------------------------------------------------------------ |
| **주도권**   | 프레임워크 ➡️ "제어 역전(IoC)"                                | 개발자                                                        |
| **자유도**   | 낮음 (정해진 구조와 규칙을 따름)                           | 높음 (필요한 기능만 선택하여 사용)                            |
| **제공 기능** | 개발에 필요한 대부분의 기능 제공                           | 특정 기능만 제공, 필요시 다른 라이브러리 또는 직접 개발 필요 |
| **예시**     | Spring, Angular, Vue, Next.js (프레임워크 성격이 강함)     | React (UI 라이브러리), jQuery                                 |

### 🌐 웹 렌더링 방식 이해
웹 페이지가 사용자에게 보여지는 과정을 이해하는 것은 중요합니다.

#### 1. Client-Side Rendering (CSR)
브라우저(클라이언트)가 대부분의 렌더링 작업을 수행하는 방식입니다.

**💬 처리 과정:**
1.  사용자가 클라이언트(브라우저)를 통해 화면을 요청합니다.
2.  서버는 **빈 껍데기 `index.html`** 파일을 클라이언트에게 전달합니다.
3.  클라이언트는 빈 화면을 렌더링합니다.
4.  서버에서 JavaScript 번들 파일을 클라이언트에게 전송합니다.
5.  클라이언트는 전송받은 JavaScript를 실행하여 화면을 렌더링합니다.
6.  비로소 사용자는 요청한 화면을 볼 수 있습니다.

**👍 장점:**
-   초기 진입 이후에는 JavaScript 번들을 한 번에 받기 때문에, 페이지 전환 시 서버에 다시 요청할 필요 없이 빠르고 부드러운 화면 전환을 제공합니다.

**👎 단점:**
-   초기 진입 시 빈 화면을 렌더링하고 JavaScript 번들을 다운로드 및 실행하는 데 시간이 오래 걸립니다. 이를 **First Contentful Paint (FCP)**가 느리다고 표현합니다.
-   **FCP (First Contentful Paint)**: 요청 시작 시점으로부터 콘텐츠가 화면에 처음 나타나는 데 걸리는 시간. FCP 비율이 높아질수록 사용자의 이탈률 또한 높아질 수 있습니다.

#### 2. Next.js와 사전 렌더링 (Pre-rendering)
Next.js는 React의 CSR 단점을 보완하기 위해 사전 렌더링 (Pre-rendering)을 채택합니다. 이는 서버에서 미리 렌더링된 HTML을 클라이언트에게 제공하는 방식입니다.

**💬 처리 과정:**
1.  사용자가 클라이언트(브라우저)를 통해 화면을 요청합니다.
2.  서버에서 JavaScript를 실행하여 해당 페이지의 HTML을 렌더링합니다.
3.  완전히 렌더링된 HTML을 클라이언트에게 전달합니다.
4.  사용자는 클라이언트를 통해 **완성된 화면을 즉시 보게 됩니다.** (단, 아직 상호작용은 불가합니다.)
5.  서버에서 현재 페이지에 필요한 JavaScript 번들을 클라이언트에게 보내줍니다.
6.  클라이언트는 JavaScript를 실행하여 HTML에 연결합니다. 이를 **수화 (Hydration)**라고 합니다. (메마른 땅에 JS의 축복을 내리는 비유)
7.  이제 화면과의 상호작용이 가능해집니다. (Time To Interactive 달성)

**✨ Pre-Fetching (사전 가져오기):**
-   사용자가 특정 링크를 클릭하기 전에 해당 페이지에 필요한 리소스를 미리 다운로드하여 페이지 로딩 속도를 향상시키는 기법입니다. (예: `Link` 컴포넌트 사용 시)

#### 3. 렌더링의 두 가지 의미
"렌더링"이라는 단어는 문맥에 따라 두 가지 의미로 사용될 수 있습니다.

1.  **JavaScript 실행(렌더링)**: JavaScript 코드 (예: React 컴포넌트)를 실제 브라우저가 이해할 수 있는 HTML 문자열로 변환하는 과정.
2.  **화면에 렌더링**: 생성된 HTML 코드를 브라우저가 파싱하고 스타일을 적용하여 실제 사용자 화면에 시각적으로 그려내는 작업.

### 🚀 Next.js 주요 기능

#### 1. Page Router
Next.js의 `pages` 폴더 구조를 기반으로 하는 라우팅 방식입니다. 파일 또는 폴더 이름이 곧 URL 경로가 됩니다.

**🛣️ 파일명 기반 라우팅 예시:**
-   `pages/items.tsx` ===> `{protocol}://{host}:{port}/items`
-   `pages/product/index.tsx` ===> `{protocol}://{host}:{port}/product`
-   `pages/product/[id].tsx` ===> `{protocol}://{host}:{port}/product/1` (동적 라우팅)
-   `pages/book/[...id].tsx` ===> `{protocol}://{host}:{port}/book/1/123/233/2223` (Catch-all Segment: 여러 개의 ID 값을 받을 수 있음)
-   `pages/book/[[...id]].tsx` ===> `{protocol}://{host}:{port}/book` 또는 `{protocol}://{host}:{port}/book/1/2` (Optional Catch-all Segment: ID 값이 없거나 많거나 모두 처리)
-   `pages/404.tsx` ===> 제공하지 않는 경로로 접근 시 표시되는 커스텀 에러 페이지

#### 2. Static Site Generation (SSG)
빌드 시점에 HTML 파일을 미리 생성하여 제공하는 방식입니다.
모든 요청에 대해 동일한 정적 파일을 제공하므로 매우 빠르며 CDN 캐싱에 유리합니다.

#### 3. Incremental Static Regeneration (ISR)
SSG 방식으로 생성된 정적 페이지의 단점(데이터가 오래될 수 있음)을 보완하는 기술입니다.
일정 시간을 주기로 (시간 기반) 또는 특정 요청 시점(요청 기반)에 페이지를 다시 생성하여 최신 데이터를 반영할 수 있습니다.

**⏳ 시간 기반 ISR 예시:**
```javascript
export const getStaticProps = async () => {
  console.log("getStaticProps ====== "); // 빌드 시점 또는 재검증 시점 호출

  const [allBooks, recoBooks] = await Promise.all([
    fetchBooks(),
    fetchRandomBooks(),
  ]);
  return {
    props: {
      allBooks,
      recoBooks,
    },
    revalidate: 3, // 유통기한(재검증 주기): 3초마다 페이지 재생성 시도
  };
};
```
-   `revalidate` 옵션을 사용하여 페이지의 "유통기한"을 설정할 수 있습니다. 설정된 시간이 지나면 다음 사용자 요청 시 백그라운드에서 페이지를 다시 생성합니다.

**🔄 요청 기반 ISR (On-Demand ISR):**
-   사용자의 데이터 수정/삭제와 같은 특정 액션 발생 시, 해당 페이지를 즉시 다시 생성하도록 트리거하는 방식입니다.

#### 4. SEO (검색 엔진 최적화)
Next.js는 `next/head` 컴포넌트를 통해 페이지별 `<head>` 태그 내용을 쉽게 관리하여 검색 엔진 최적화에 도움을 줍니다.

```javascript
import Head from 'next/head'

export default function Component(){
	return (
		<>
			<Head>
				<title>타이틀</title>
				<meta property="og:image" content=""/>
				{/* 기타 메타 태그, CSS, 스크립트 등을 추가 */}
			</Head>
			{/* 페이지 콘텐츠 */}
		</>
	)
}
```

### 🚧 Page Router의 단점
-   페이지별 레이아웃 설정이 번거롭습니다.
-   데이터 페칭 로직이 페이지 컴포넌트에 집중되는 경향이 있습니다.
-   불필요한 컴포넌트들도 초기 JavaScript Bundle에 포함될 수 있어 번들 크기가 커질 위험이 있습니다.

---

## 💡 새롭게 알게 된 점
-   **렌더링의 이중적 의미**: JavaScript 코드를 HTML로 변환하는 것과 HTML을 화면에 그리는 것이 서로 다른 과정임을 명확히 구분하게 되었습니다.
-   **Hydration의 중요성**: Pre-rendering에서 서버가 완성된 HTML을 보내주더라도, 클라이언트 측에서 JavaScript가 HTML에 연결(수화)되어야 비로소 상호작용이 가능해진다는 점을 깨달았습니다. 이는 사용자 경험에 중요한 과정입니다.
-   **FCP와 사용자 이탈률**: CSR의 초기 FCP 지연이 사용자 이탈률에 직접적인 영향을 미칠 수 있다는 점을 인지하게 되어, 초기 로딩 성능의 중요성을 다시 한번 생각하게 되었습니다.
-   **ISR의 유연성**: SSG의 정적이라는 한계를 `revalidate` 옵션을 통해 동적으로 극복할 수 있는 ISR의 개념이 매우 효율적이라고 느꼈습니다. 특히 On-Demand ISR은 데이터 변경이 잦은 서비스에 유용할 것으로 보입니다.
-   **Next.js 라우팅의 다양성**: `[...id]`와 `[[...id]]`와 같은 Catch-all Segment 및 Optional Catch-all Segment가 있어 복잡한 URL 패턴을 유연하게 처리할 수 있다는 점이 인상 깊었습니다.

---

## 🤔 궁금한 점 / 추가 학습 필요
-   **Pre-Fetching의 구체적인 작동 방식**: Next.js에서 `Link` 컴포넌트가 Pre-fetching을 어떻게 수행하는지, 그리고 개발자가 Pre-fetching 전략을 어떻게 제어할 수 있는지 더 알아보고 싶습니다.
-   **On-Demand ISR 구현 상세**: `revalidate`를 통한 시간 기반 ISR 외에, On-Demand ISR이 서버 API 등을 통해 어떻게 트리거되고 구현되는지 실제 예시를 통해 학습하고 싶습니다.
-   **App Router와의 비교**: Page Router의 단점들이 Next.js 13부터 도입된 App Router에서 어떻게 개선되었는지 비교 학습이 필요합니다.

---

## 🔗 참고 자료
-   [작성 필요]