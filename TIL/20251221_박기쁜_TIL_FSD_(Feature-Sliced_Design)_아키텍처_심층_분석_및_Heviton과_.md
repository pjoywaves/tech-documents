---
template: "TIL"
title: "FSD (Feature-Sliced Design) 아키텍처 심층 분석 및 Heviton과 비교"
created_at: "2025-12-21 22:40"
created_by:
  name: "gbpark"
  email: "gbpark@herit.net"
participants: []
tags:
  - "FSD"
  - "프론트엔드 아키텍처"
  - "Feature-Sliced Design"
  - "모듈화"
  - "Heviton"
category: "TIL"
status: "draft"
visibility: "internal"
related_docs: []
custom: {}
---

# TIL (Today I Learned)

## 📚 학습 주제
프론트엔드 아키텍처: Feature-Sliced Design (FSD) 이해 및 Heviton과의 비교

---

## 🔑 핵심 내용
- **Feature-Sliced Design(FSD)은 애플리케이션 기능에 초점을 맞춰 구조화하는 프론트엔드 아키텍처 방법론**입니다. 비즈니스 요구사항 변화에도 안정적인 유지보수를 목표로 합니다.
- **FSD는 7가지 레이어(App, Pages, Widgets, Features, Entities, Shared, Processes - deprecated)로 구성**되며, **상위 레이어는 하위 레이어만 참조 가능한 엄격한 단방향 의존성 규칙**을 가집니다.
- **Entities(비즈니스 데이터), Features(사용자 액션/기능), Widgets(복합 UI 블록)의 역할을 명확히 구분**하여 코드의 관심사를 분리하고 재사용성을 높입니다.
- **Heviton(Feature-first) 아키텍처와 비교**했을 때, FSD는 Widgets 레이어의 명시적인 존재, 엄격한 의존성 규칙, 그리고 Entity와 Feature의 명확한 분리에서 차이를 보입니다.
- FSD의 **엄격한 규칙은 초기 학습 곡선은 높지만, 장기적으로 대규모 프로젝트의 일관성과 유지보수성을 크게 향상**시킵니다.

---

## 📝 상세 내용

### 💡 Feature-Sliced Design(FSD) 소개
FSD는 프론트엔드 프로젝트를 구조화하는 아키텍처 방법론으로, 비즈니스 요구사항이 계속 변해도 프로젝트를 이해하기 쉽고 안정적으로 유지할 수 있도록 돕습니다. 기술적 레이어보다는 **애플리케이션 기능**에 초점을 맞춰 구조화하는 것이 특징입니다.

핵심 원칙은 **단방향 의존성**입니다:
> **상위 레이어는 하위 레이어만 참조 가능하며, 역방향 참조는 엄격히 금지됩니다.**

```
App → Pages → Widgets → Features → Entities → Shared
 ⬇      ⬇       ⬇         ⬇          ⬇
(아래로만 참조 가능, 위로는 불가능)
```

### 🧱 FSD 7개 레이어 구조
FSD는 애플리케이션의 규모와 복잡도에 따라 다음과 같은 레이어로 구성됩니다. `Processes` 레이어는 현재 사용되지 않습니다 (deprecated).

| 레이어 | 역할 | 예시 |
|--------|------|------|
| **App** | 앱 실행 기반 | 라우팅, 진입점, 글로벌 스타일, Provider |
| **~~Processes~~** | (deprecated) | 복잡한 페이지 간 시나리오 |
| **Pages** | 페이지 단위 | 전체 페이지 또는 큰 페이지 파트 |
| **Widgets** | 복잡한 UI 블록 | 헤더, 네비게이션, 검색 필터, 댓글 위젯 |
| **Features** | 비즈니스 기능 | 로그인, 장바구니 추가, 좋아요 |
| **Entities** | 비즈니스 엔티티 | user, product, order, post |
| **Shared** | 공통 유틸리티 | Button, Input, API client, 유틸 함수 |

### 🔍 Entities, Features, Widgets 차이점
FSD에서 가장 중요한 개념적 구분은 Entities, Features, Widgets입니다. 이 세 가지는 애플리케이션의 핵심 구성 요소를 나타내며, 각각의 역할과 책임이 명확히 분리됩니다.

| 구분 | **Entities** | **Features** | **Widgets** |
|------|--------------|--------------|-------------|
| **개념** | 비즈니스 데이터 객체 | 사용자 액션/기능 | 독립적인 UI 덩어리 |
| **질문** | "무엇을 다루나?" | "무엇을 하나?" | "무엇을 보여주나?" |
| **예시** | User, Product, Post | 로그인, 장바구니 추가 | Header, Sidebar, CommentWidget |
| **재사용** | 여러 기능에서 사용 | 특정 기능에서 사용 | 여러 페이지에서 사용 |
| **비즈니스 가치** | 데이터 중심 | 액션/행동 중심 | UI/UX 중심 |

#### 📦 Entities (엔티티)
앱의 **핵심 비즈니스 객체**를 정의합니다. "이 앱이 다루는 핵심 데이터는 무엇인가?"에 대한 답변입니다.
- **특징:**
    - 여러 기능(Features)에서 **공통으로 사용**됩니다.
    - "명사"로 표현됩니다 (User, Product, Order...).
    - 데이터 구조와 표현 방법을 정의합니다.
- **구조 예시:**
```
📂 entities
  📂 user          # 사용자
    📁 ui          # UserCard, UserAvatar 컴포넌트
    📁 model       # User 타입, 상태관리
    📁 api         # getUserById, updateUser
  
  📂 product       # 상품
    📁 ui          # ProductCard, ProductImage
    📁 model       # Product 타입, 장바구니 상태
    📁 api         # getProducts, getProductDetail
  
  📂 post          # 게시글
    📁 ui          # PostCard, PostPreview
    📁 model       # Post 타입
    📁 api         # getPosts, createPost
```

#### 🚀 Features (기능)
**사용자에게 가치를 주는 액션**을 정의합니다. "사용자가 무엇을 할 수 있나?"에 대한 답변입니다.
- **특징:**
    - "동사"로 표현됩니다 (로그인하기, 추가하기, 좋아요 누르기...).
    - Entities를 **사용**하며, 비즈니스 로직을 포함합니다.
- **구조 예시:**
```
📂 features
  📂 auth                    # 인증 기능
    📁 ui
      LoginForm.vue          # 로그인 폼
      SignupForm.vue         # 회원가입 폼
    📁 model
      useAuth.ts             # 인증 상태 관리
    📁 api
      login.ts               # 로그인 API 호출
  
  📂 add-to-cart            # 장바구니 추가 기능
    📁 ui
      AddToCartButton.vue    # 장바구니 담기 버튼
    📁 model
      useAddToCart.ts        # 장바구니 추가 로직
  
  📂 product-like           # 상품 좋아요 기능
    📁 ui
      LikeButton.vue         # 좋아요 버튼
    📁 model
      useLike.ts             # 좋아요 상태 관리
```

#### 🧩 Widgets (위젯)
페이지에서 **독립적으로 동작하는 복합적인 UI 블록**을 정의합니다. "페이지에서 독립적으로 동작하는 큰 UI 블록은?"에 대한 답변입니다.
- **특징:**
    - **여러 Features를 조합**하여 완성된 UI 블록을 만듭니다.
    - 페이지의 일부이지만 **자급자족**하며, 완전한 사용 사례(use case)를 제공합니다.
- **구조 예시:**
```
📂 widgets
  📂 header               # 헤더
    📁 ui
      Header.vue          # 로고 + 네비게이션 + 검색 + 장바구니
    📁 model
      useHeaderMenu.ts
  
  📂 product-filter       # 상품 필터 위젯
    📁 ui
      FilterSidebar.vue   # 카테고리, 가격, 브랜드 필터
    📁 model
      useFilter.ts
  
  📂 comment-section      # 댓글 섹션
    📁 ui
      CommentList.vue     # 댓글 목록 + 작성 폼
      CommentItem.vue
    📁 model
      useComments.ts
```

#### 🛠️ FSD 코드 예시: 레이어 간의 조합
다음은 FSD 레이어들이 어떻게 조합되어 최종 페이지를 구성하는지 보여주는 예시입니다.

```
📂 entities
  📂 product
    📁 ui
      ProductCard.vue        # 상품 카드 (단순 표시)
      ProductImage.vue       # 상품 이미지
    📁 model
      types.ts               # Product 타입 정의
    📁 api
      getProduct.ts          # 상품 조회

📂 features
  📂 add-to-cart
    📁 ui
      AddToCartButton.vue    # "장바구니 담기" 버튼
    📁 model
      useAddToCart.ts        # 장바구니 추가 로직
  
  📂 product-like
    📁 ui
      LikeButton.vue         # "좋아요" 버튼
    📁 model
      useLike.ts

📂 widgets
  📂 product-card-with-actions
    📁 ui
      ProductCardFull.vue    # ProductCard + AddToCartButton + LikeButton

📂 pages
  📂 product-detail
    📁 ui
      ProductDetailPage.vue  # 전체 페이지 조합
```

**세부 컴포넌트 코드 예시:**

```vue
<!-- entities/product/ui/ProductCard.vue -->
<!-- 순수하게 상품 정보만 표시 -->
<template>
  <div>
    <ProductImage :src="product.image" />
    <h3>{{ product.name }}</h3>
    <p>{{ product.price }}원</p>
  </div>
</template>

<script setup>
import ProductImage from './ProductImage.vue';
// Product 타입 정의가 있다고 가정
const props = defineProps({
  product: Object, // { id: string, name: string, price: number, image: string }
});
</script>
```

```vue
<!-- features/add-to-cart/ui/AddToCartButton.vue -->
<!-- "장바구니 담기" 기능 -->
<template>
  <button @click="handleAddToCart">
    장바구니 담기
  </button>
</template>

<script setup>
import { useAddToCart } from '@/features/add-to-cart/model/useAddToCart'; // 가상의 훅 임포트
const props = defineProps({
  productId: String,
});
const { addToCart } = useAddToCart();
const handleAddToCart = () => addToCart(props.productId);
</script>
```

```vue
<!-- widgets/product-card-with-actions/ui/ProductCardFull.vue -->
<!-- Entity + Features 조합 -->
<template>
  <div>
    <ProductCard :product="product" />
    <AddToCartButton :productId="product.id" />
    <LikeButton :productId="product.id" />
  </div>
</template>

<script setup>
import { ProductCard } from '@/entities/product';
import { AddToCartButton } from '@/features/add-to-cart';
import { LikeButton } from '@/features/product-like'; // 가상의 임포트
const props = defineProps({
  product: Object, // { id: string, name: string, price: number, image: string }
});
</script>
```

```vue
<!-- pages/product-detail/ui/ProductDetailPage.vue -->
<!-- 전체 페이지 -->
<template>
  <div>
    <Header />
    <ProductCardFull :product="product" />
    <CommentSection :productId="product.id" />
  </div>
</template>

<script setup>
import { Header } from '@/widgets/header'; // 가상의 임포트
import { ProductCardFull } from '@/widgets/product-card-with-actions';
import { CommentSection } from '@/widgets/comment-section'; // 가상의 임포트
const props = defineProps({
  product: Object, // 실제로는 라우터 파람 등으로 상품 정보를 가져옴
});
</script>
```

### 🔄 FSD와 다른 아키텍처 방법론 비교
- **MVC (Model-View-Controller)**: 기술적 레이어로 분리 (Model, View, Controller) vs. **FSD**: 비즈니스 도메인으로 분리 (User, Product, Order...).
- **Atomic Design**: UI 컴포넌트의 크기(Atom → Molecule → Organism...)로 분리 vs. **FSD**: 비즈니스 의미(Entity → Feature → Widget...)로 분리.
- **Component-Based**: 컴포넌트 재사용 중심 vs. **FSD**: 비즈니스 로직과 컴포넌트의 구조화를 함께 다룸.

### 🆚 FSD와 Heviton 아키텍처 비교
Heviton은 "Feature-first"라는 접근 방식을 사용하는 프로젝트 구조로, FSD와 유사하면서도 몇 가지 중요한 차이점을 가집니다.

#### Heviton vs FSD - 핵심 구조 비교

| 구분 | **Heviton (Feature-first)** | **FSD** |
|------|------------------------------|---------|
| **최상위 조직** | 기능 중심 (`/src/features`) | 레이어 중심 (7 Layers) |
| **라우팅** | `/app` (Expo Router, 파일 기반) | `pages/` 레이어 |
| **공통 UI** | `/src/ui` (27개 컴포넌트) | `shared/ui/` |
| **상태 관리** | `/src/stores` (Zustand) | 각 레이어의 `model/` 세그먼트 |
| **API 레이어** | `/src/lib/api` | `shared/api/` |
| **유틸리티** | `/src/lib` | `shared/lib/` |

#### Heviton과 FSD의 구조 예시

**Heviton 구조 예시:**
```
heviton-app-frontend/
├── app/                    # Expo Router (파일 기반 라우팅)
│   ├── (auth)/            # 인증 화면
│   ├── (app)/(tabs)/      # 메인 앱 (탭)
│   └── _layout.tsx
├── src/
│   ├── features/          # 기능별 모듈 ⭐ 핵심
│   │   ├── auth/
│   │   │   ├── components/
│   │   │   ├── hooks/
│   │   │   ├── types.ts
│   │   │   └── index.ts
│   │   ├── plant/
│   │   ├── menu/
│   │   └── user/
│   ├── ui/                # 재사용 UI 컴포넌트
│   ├── lib/               # 라이브러리 추상화
│   │   ├── api/
│   │   ├── query/
│   │   ├── navigation/
│   │   └── utils/
│   ├── stores/            # 글로벌 상태 (Zustand)
│   ├── theme/             # 디자인 토큰
│   └── viz/               # 차트
```

**FSD 구조 예시:**
```
src/
├── app/                   # 앱 전역 설정
│   ├── providers/
│   └── styles/
├── pages/                 # 페이지 (라우트별)
│   ├── dashboard/
│   ├── login/
│   └── settings/
├── widgets/               # 복합 UI 블록
│   ├── header/
│   ├── sidebar/
│   └── plant-card/
├── features/              # 비즈니스 기능
│   ├── auth/
│   │   ├── ui/
│   │   ├── model/
│   │   └── api/
│   ├── plant-select/
│   └── alert-notification/
├── entities/              # 비즈니스 엔티티
│   ├── user/
│   │   ├── ui/
│   │   ├── model/
│   │   └── api/
│   ├── plant/
│   └── energy/
└── shared/                # 공통 요소
    ├── ui/
    ├── api/
    ├── lib/
    └── config/
```

#### 각 항목별 상세 비교

-   **라우팅 방식 비교:**

| 구분 | **Heviton** | **FSD** |
|------|-------------|---------|
| **위치** | `/app` (별도 폴더) | `pages/` 레이어 |
| **방식** | Expo Router (파일 기반) | 레이어 내 페이지 슬라이스 |
| **구조** | `(auth)/login.tsx` | `pages/login/ui/LoginPage.vue` |

-   **Entity vs Feature 분리 비교:**
    -   **Heviton의 `/src/features`**: `features/plant` (발전소 데이터), `features/user` (사용자 데이터) 처럼 **Entity와 Feature가 혼재**되어 있습니다.
    -   **FSD의 분리**: `entities/plant`, `entities/user` (데이터)와 `features/auth`, `features/plant-select` (액션)으로 **명확하게 분리**됩니다.

-   **공통 UI 분리 비교:**

| 구분 | **Heviton** | **FSD** |
|------|-------------|---------|
| **위치** | `/src/ui` (최상위) | `shared/ui/` (레이어 내) |
| **접근** | `@/ui/button` | `@/shared/ui/button` |
    -   둘 다 공통 UI를 별도로 분리한다는 점에서 유사합니다.

-   **상태 관리 방식 비교:**
    -   **Heviton**: `/src/stores` 폴더에 글로벌 상태(Zustand)를 관리하며, `features/*/hooks`에서 TanStack Query를 사용합니다. 글로벌 상태와 서버 상태가 분리되어 있습니다.
    -   **FSD**: 각 레이어/슬라이스의 `model/` 세그먼트 내부에 모든 상태(Zustand, TanStack Query 등)를 캡슐화합니다.

-   **API 레이어 비교:**
    -   **Heviton**: `src/lib/api`에 공통 API 클라이언트를 두며, 각 feature의 `hooks/`에서 API 호출 훅을 관리합니다.
    -   **FSD**: `shared/api`에 공통 API 클라이언트를 두고, 각 feature의 `api/`에서 해당 feature 관련 API만 관리합니다.
    -   공통 클라이언트를 분리하고 각 feature에서 사용하는 점은 유사합니다.

-   **재사용 컴포넌트 구조 비교:**

| 구분 | **Heviton** | **FSD** |
|------|-------------|---------|
| **재사용 UI** | `/src/ui` | `shared/ui/` |
| **Feature 전용** | `features/*/components` | `features/*/ui/` |
| **화면** | `/app/*.tsx` | `pages/*/ui/` |
| **복합 UI** | ❌ 없음 | `widgets/*/ui/` ✅ |
    -   **핵심 차이**: FSD는 **Widgets 레이어**가 복합 UI 블록을 위한 명시적인 존재합니다.

-   **의존성 규칙 비교:**
    -   **Heviton**: `app/` → `features/` → `ui/` → `lib/` 와 같은 일반적인 흐름은 있으나, **규칙이 명시적이지 않아** 개발자 판단에 의존합니다.
    -   **FSD**: `Pages` → `Widgets` → `Features` → `Entities` → `Shared` 와 같은 **엄격한 단방향 의존성을 강제**하며, 린트 등으로 위반 시 에러를 발생시킵니다.

-   **실제 프로젝트 구조 비교 (Heviton vs FSD 예시):**
    -   **Heviton 구조:** `features/plant` 하나에 화면, 기능, 데이터 관련 로직이 모두 포함될 수 있습니다.
    ```
    app/(app)/(tabs)/dashboard/index.tsx  # 화면
      ↓
    features/plant/                       # 발전소 기능
      components/
        dashboard/
          plant-card.tsx                  # 발전소 카드
          summary.tsx                     # 요약 위젯
      hooks/
        use-plant-list.ts                 # API 호출
    ```
    -   **FSD 구조:** Entity(데이터), Feature(액션), Widget(복합 UI)로 **명확히 분리**됩니다.
    ```
    pages/dashboard/
      ui/DashboardPage.vue                # 화면

    widgets/
      plant-summary/                      # 요약 위젯 (복합)
        ui/PlantSummary.vue

    features/
      plant-select/                       # 발전소 선택 기능
        ui/PlantSelector.vue

    entities/
      plant/                              # 발전소 엔티티
        ui/PlantCard.vue                  # 단순 카드
        model/plantStore.ts
        api/getPlants.ts
    ```

#### 공통점
-   **기능 중심 조직**: 둘 다 도메인/기능별로 코드를 그룹화하여 모듈화를 추구합니다.
-   **공통 UI 분리**: `ui/` 또는 `shared/ui/`와 같은 형태로 재사용 가능한 UI 컴포넌트를 분리합니다.
-   **API 추상화**: `lib/api/` 또는 `shared/api/`를 통해 공통 API 클라이언트 및 관련 로직을 추상화합니다.
-   **타입 안전성**: TypeScript Strict 모드를 활용하여 견고한 코드 작성을 지향합니다.
-   **Public API 패턴**: `index.ts` 파일을 사용하여 각 모듈의 외부 노출 영역(Public API)을 제어합니다.

#### 핵심 차이점 요약

| 구분 | **Heviton (Feature-first)** | **FSD** |
|------|------------------------------|---------|
| **구조 원칙** | Feature 중심, 자유로운 구조 | 7 Layer + Slice + Segment (엄격) |
| **의존성 규칙** | 암묵적 (개발자 판단) | 명시적 (린트로 강제) |
| **Entity vs Feature** | 혼재 (`features/plant`, `features/user`) | 명확히 분리 (`entities/`, `features/`) |
| **Widgets** | ❌ 개념 없음 | ✅ 명시적 레이어 존재 |
| **라우팅** | `/app` 별도 폴더 | `pages/` 레이어 |
| **상태 관리** | 글로벌 분리 (`/stores`) | 각 레이어 내부 (`model/`) |
| **학습 곡선** | 낮음 (직관적) | 높음 (개념 학습 필요) |
| **적용 난이도** | 쉬움 | 중간 (규칙 많음) |

---

## 💡 새롭게 알게 된 점
- **FSD의 엄격한 레이어 의존성 규칙의 중요성**: 상위 레이어가 하위 레이어만 참조할 수 있다는 규칙이 코드의 응집도를 높이고 결합도를 낮춰 장기적인 유지보수성을 크게 향상시킨다는 것을 깨달았습니다. 린트 룰을 통해 이를 강제하는 것이 중요하다는 점을 인지했습니다.
- **Entities, Features, Widgets 간의 명확한 역할 구분**: 기존에는 이 세 가지 개념이 혼재되기 쉬웠으나, FSD에서는 각각 "무엇을 다루나?", "무엇을 하나?", "무엇을 보여주나?"라는 질문으로 역할을 명확히 구분하여 코드의 관심사를 효과적으로 분리할 수 있음을 알게 되었습니다.
- **Widgets 레이어의 가치**: 단순히 UI 컴포넌트의 집합이 아니라, 여러 Features를 조합하여 독립적이고 완성된 사용 사례를 제공하는 중요한 계층임을 이해했습니다. 이는 페이지를 조립하는 데 있어 유연성과 재사용성을 제공합니다.
- **Heviton과 FSD 비교를 통한 FSD의 강점**: Heviton처럼 Feature-first 접근 방식이 직관적일 수 있지만, Entity와 Feature의 혼재 및 느슨한 의존성 규칙으로 인해 프로젝트가 커질수록 유지보수에 어려움이 생길 수 있다는 점을 파악했습니다. FSD의 명확한 분리와 규칙은 이러한 문제를 예방하는 데 효과적입니다.
- **DRY(Don't Repeat Yourself) 원칙과 FSD의 균형**: `Shared` 레이어는 진짜 공통 코드를 위한 것이며, `Features/Entities`는 각 기능에 특화된 로직을 담아 불필요한 추상화 없이 로컬 커스터마이제이션을 허용함으로써 DRY 원칙과 유연성 사이의 균형을 추구한다는 점을 알게 되었습니다.

---

## 🤔 궁금한 점 / 추가 학습 필요
- FSD를 실제 대규모 프로젝트에 적용할 때 초기 설정(린트, 플러그인 등)에 대한 가이드라인이나 템플릿 프로젝트를 더 자세히 살펴보고 싶습니다.
- 레거시 프로젝트에 FSD를 점진적으로 도입하는 전략에 대해 추가적인 자료를 찾아보고 싶습니다.
- FSD가 SSR/SSG 환경에서 효율적으로 작동하는 방식에 대한 더 심층적인 이해가 필요합니다.
- FSD의 "slice"와 "segment" 개념이 실제 팀 개발 시 어떻게 코드 리뷰나 커밋 전략에 영향을 미치는지 궁금합니다.

---

## 🔗 참고 자료
* https://feature-sliced.github.io/documentation/docs/get-started/overview
* https://feature-sliced.design/
* https://velog.io/@teo/fsd
* https://lapidix.dev/posts/fsd-ddd-clean-architecture
