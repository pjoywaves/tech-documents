---
template: "TIL"
title: "Atomic Design 학습 및 적용 고민"
created_at: "2025-11-06 10:58"
created_by:
  name: "박기쁜"
  email: "gbpark@herit.net"
participants: []
tags:
  - "Atomic Design"
  - "컴포넌트"
  - "UI 아키텍처"
  - "프론트엔드"
  - "디자인 시스템"
category: "TIL"
status: "draft"
visibility: "internal"
related_docs: []
custom: {}
---

# TIL (Today I Learned)

## 📚 학습 주제
Atomic Design에 대한 이해 및 다양한 프론트엔드 아키텍처 패턴 비교 분석

---

## 🔑 핵심 내용
*   Atomic Design의 개념과 구성 요소 (Atoms, Molecules, Organisms, Templates, Pages) 이해
*   Atomic Design을 따르는 이유와 따르지 않는 이유 분석
*   Component-Driven Development (CDD), Container-Presenter 패턴, MVVM 등 다양한 프론트엔드 아키텍처 패턴 학습
*   각 패턴의 장단점 비교 및 실제 프로젝트 적용 가능성 검토
*   디자이너, 개발자, 사용자 간 멘탈 모델의 중요성 인지

---

## 📝 상세 내용

### 1. Atomic Design 이란? ⚛️

컴포넌트 제작 시 발생하는 문제점들을 해결하기 위한 방법론

*   **문제점:**
    *   컴포넌트 파편화: UI 불일치, 명칭 혼란
    *   명확한 설계 기준 부재: 재사용성 저하, 코드 리뷰 어려움

*   **Atomic Design:**
    *   디자인 시스템을 만드는 방법론
    *   화학적 관점에서 영감을 얻음 (Atoms -> Molecules -> Organisms -> Templates -> Pages)
    *   각 단계별로 추상적인 것에서 구체적인 것으로 발전

### 1.1. Atomic Design 구성 요소

| 구성 요소  | 설명                                                                                      | 예시                                                                                                |
| :--------- | :---------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------- |
| Atoms      | 더 이상 분해할 수 없는 기본 컴포넌트                                                      | label, input, button, 폰트, 색상                                                                 |
| Molecules  | Atoms의 결합, 한 가지 기능을 수행                                                          | input + button (form 전송)                                                                         |
| Organisms  | Molecules의 결합, 특정 영역과 컨텍스트를 가짐                                               | header (logo, navigation, search form 포함)                                                          |
| Templates  | Organisms, Molecules로 구성, 페이지 스켈레톤                                                 | 실제 콘텐츠가 없는 와이어프레임                                                                     |
| Pages      | Template의 인스턴스, 실제 콘텐츠                                                            | 장바구니 페이지 (사용자가 담은 상품 유/무)                                                          |

### 1.2. Atomic Design 적용 시 어려운 점

*   단위를 나누는 기준의 주관성 (특히 Molecule과 Organism 구분)
    *   Molecule: SRP에 따른 1가지 책임, 컨텍스트 없이 UI적인 요소 (Input, TextBadge)
    *   Organism: 서비스에서 Layout을 기준으로 나눌 수 있는 영역, 컨텍스트 존재 (CardViewItem, CategoryMore)
*   약간의 다른 Organism이 있을 때 중복 컴포넌트 발생 또는 불필요한 Props 증가

### 1.3. Atomic Design 원활한 적용을 위한 도구

*   스토리북
*   디자이너가 작성한 피그마 컴포넌트
*   UI 모델링 (Figjam)
*   재사용성을 높이기 위해 마진/패딩 스타일은 Atoms에 직접 정의하지 않고 외부에서 주입

    ```jsx
    <Comment style={{margin: '20px 40px', flex: 1}}/>
    ```

### 2. Atomic Design을 따르는 이유 vs 따르지 않는 이유 ⚖️

| 구분          | 이유                                                                                                                                                                                                 |
| :------------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 따르는 사람들 | 잘 정리되는 폴더 구조, 컴포넌트들의 선명한 목적, 재사용성                                                                                                                                              |
| 따르지 않는 사람들 | 복잡하게 느껴짐, Molecule과 Organism의 모호함, Template와 Pages를 데이터 유무로 분리하지만 Props Drilling 발생, Page -> Atom으로 데이터 주입 시 깊은 Props 전달의 귀찮음 발생 (상태 관리 라이브러리 사용 필요) |

### 3. 다른 프론트엔드 아키텍처 패턴 🏗️

#### 3.1. Component-Driven Development (CDD)

*   UI를 페이지 단위가 아니라 컴포넌트 단위로 먼저 개발하고 조립 (Storybook 접근 방식)
*   컴포넌트 중심의 협업 및 테스트 강조

```
src/
 ├─ components/
 │   ├─ atoms/
 │   │   ├─ Button.vue
 │   │   ├─ TextField.vue
 │   │   └─ Icon.vue
 │   │
 │   ├─ molecules/
 │   │   ├─ SearchBar.vue
 │   │   ├─ Card.vue
 │   │   └─ Modal.vue
 │   │
 │   ├─ organisms/
 │   │   ├─ Header.vue
 │   │   ├─ Sidebar.vue
 │   │   └─ UserProfileForm.vue
 │   │
 │   ├─ templates/
 │   │   ├─ DashboardLayout.vue
 │   │   └─ AuthLayout.vue
 │   │
 │   └─ pages/
 │       ├─ LoginPage.vue
 │       └─ DashboardPage.vue
 │
 ├─ stories/                   # Storybook 문서 (컴포넌트별)
 │   ├─ Button.stories.js
 │   ├─ Card.stories.js
 │   └─ Header.stories.js
 │
 ├─ styles/                    # 공통 스타일, 토큰
 │   ├─ _variables.scss
 │   └─ _mixins.scss
 │
 ├─ utils/
 │   └─ formatters.js
 └─ main.js
```

#### 3.2. Container-Presenter Pattern / Smart-Dumb Component Pattern

*   UI (Presenter, Dumb)와 비즈니스 로직 (Container, Smart) 분리
*   관심사 분리 (SoC - Separation of Concerns)

```
src/
 ├─ components/
 │   ├─ user/
 │   │   ├─ UserListContainer.vue      # 데이터 fetch, 상태 관리
 │   │   ├─ UserListPresenter.vue      # 화면 표시(UI)
 │   │   └─ index.js
 │   │
 │   ├─ dashboard/
 │   │   ├─ EnergyChartContainer.vue
 │   │   ├─ EnergyChartPresenter.vue
 │   │   └─ SummaryContainer.vue
 │
 ├─ api/
 │   ├─ userApi.js
 │   └─ dashboardApi.js
 │
 ├─ store/
 │   ├─ userStore.js
 │   └─ dashboardStore.js
 │
 └─ main.js
```

#### 3.3. MVVM (Model-View-ViewModel)

*   Model: 데이터 및 비즈니스 로직 (API, DB)
*   View: 사용자에게 보여지는 화면
*   ViewModel: View와 Model 사이를 연결하고 반응형 데이터 상태 관리

```
src/
 ├─ models/
 │   ├─ UserModel.js             # 데이터 구조, Entity 정의
 │   ├─ EnergyModel.js
 │   └─ AlarmModel.js
 │
 ├─ views/
 │   ├─ UserView.vue             # 사용자 화면(View)
 │   ├─ DashboardView.vue
 │   └─ AlarmView.vue
 │
 ├─ viewmodels/
 │   ├─ useUserViewModel.js      # ViewModel (상태 + 로직)
 │   ├─ useDashboardViewModel.js
 │   └─ useAlarmViewModel.js
 │
 ├─ api/
 │   ├─ userApi.js
 │   ├─ dashboardApi.js
 │   └─ alarmApi.js
 │
 ├─ store/
 │   ├─ userStore.js
 │   └─ alarmStore.js
 │
 └─ main.js
```

#### 3.4. Feature-Based Structure (기능 단위 구조)

*   기능별로 폴더를 쪼개고, 각 폴더 안에 UI + 상태 + 비즈니스 로직을 한데 묶음

```
src/
 ├─ features/
 │   ├─ user/
 │   │   ├─ components/
 │   │   ├─ hooks/
 │   │   ├─ api/
 │   │   └─ index.js
 │   ├─ dashboard/
 │   └─ auth/
```

#### 3.5. Domain-Driven UI (백엔드의 DDD 기반)

*   도메인(user, building, energy 등) 단위로 컴포넌트, 스토어, API를 분리
*   각 도메인 내부에서 UI, State, Logic, Repository, Model을 독립적으로 분리

```
src/
 ├─ domains/
 │   ├─ user/
 │   │   ├─ components/
 │   │   │   ├─ UserList.vue
 │   │   │   └─ UserCard.vue
 │   │   ├─ store/
 │   │   │   └─ userStore.js
 │   │   ├─ api/
 │   │   │   └─ userApi.js
 │   │   ├─ models/
 │   │   │   └─ UserModel.js
 │   │   ├─ services/
 │   │   │   └─ UserService.js      # 비즈니스 로직 (Repository와 View 연결)
 │   │   └─ index.js
 │   │
 │   ├─ energy/
 │   │   ├─ components/
 │   │   │   ├─ EnergyChart.vue
 │   │   │   └─ EnergySummary.vue
 │   │   ├─ store/
 │   │   │   └─ energyStore.js
 │   │   ├─ api/
 │   │   │   └─ energyApi.js
 │   │   ├─ models/
 │   │   │   └─ EnergyModel.js
 │   │   ├─ services/
 │   │   │   └─ EnergyService.js
 │   │   └─ index.js
 │
 ├─ shared/                       # 공통 모듈
 │   ├─ components/
 │   │   ├─ Button.vue
 │   │   └─ Card.vue
 │   ├─ utils/
 │   │   └─ formatter.js
 │   ├─ hooks/
 │   │   └─ useFetch.js
 │   ├─ constants/
 │   │   └─ colors.js
 │   └─ styles/
 │       └─ variables.scss
 │
 ├─ router/
 │   └─ index.js
 │
 ├─ store/
 │   └─ rootStore.js              # 각 domain store 병합
 │
 └─ main.js
```

### 4. 어떤 디자인을 채택해야 할까? 🤔

*   Atomic Design을 재해석하여 프로젝트에 맞는 구조를 세울 필요가 있다.
*   HZEMS (개발 중인 프로젝트) 기준으로 MVVM과 유사한 패턴을 따르되, Atomic Design의 개념 (Atom, Molecule, Organism)을 활용하여 폴더 구조 및 컴포넌트 네이밍 개선 가능
*   기존 컴포넌트 개발 시 구조 미정립, Props 남발 등의 문제점을 해결 가능

---

## 💡 새롭게 알게 된 점

*   **멘탈 모델:** 사용자가 시스템에 대해 형성하는 이해와 믿음. 디자이너와 사용자 간 멘탈 모델 격차를 줄이는 것이 중요.
    *   예: 뒤로가기 버튼 = 취소 기능, 검색창 = 어떤 정보든 제공
*   컴포넌트 구조/폴더에 대한 사전 고민의 중요성
*   **Prop Drilling:** 컴포넌트 트리에서 데이터를 하위 컴포넌트로 전달하기 위해 여러 단계를 거쳐 프로퍼티를 내려주는 것. 상태 관리 라이브러리(Context API, Redux, Pinia, Vuex 등)를 통해 해결 가능
*   **Container-Presenter 패턴:** UI 로직과 비즈니스 로직 분리
    *   Presenter: UI 담당 (Props로 받은 데이터만 화면에 표시)
    *   Container: 데이터 관리 (API 호출, 상태 관리, 이벤트 핸들링, Presenter에게 데이터 전달)

---

## 🤔 궁금한 점 / 추가 학습 필요

*   Organism 중복 해결을 위한 Compound 컴포넌트
*   Figjam 사용법
*   프론트엔드 아키텍처 심화 학습

---

## 🔗 참고 자료

*   [https://tech.kakaoent.com/front-end/2022/220505-how-page-part-use-atomic-design-system/](https://tech.kakaoent.com/front-end/2022/220505-how-page-part-use-atomic-design-system/)
*   [https://velog.io/@teo/Atomic-Design-Pattern](https://velog.io/@teo/Atomic-Design-Pattern)
*   [https://brunch.co.kr/@amirjung/110](https://brunch.co.kr/@amirjung/110)
*   [https://velog.io/@teo/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C%EC%97%90%EC%84%9C-MV-%EC%95%84%ED%82%A4%ED%85%8D%EC%B3%90%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80%EC%9A%94](https://velog.io/@teo/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C%EC%97%90%EC%84%9C-MV-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80%EC%9A%94)
*   [https://kciter.so/posts/effective-atomic-design/](https://kciter.so/posts/effective-atomic-design/)
*   [https://velog.io/@rachel28/Prop-Drilling](https://velog.io/@rachel28/Prop-Drilling)