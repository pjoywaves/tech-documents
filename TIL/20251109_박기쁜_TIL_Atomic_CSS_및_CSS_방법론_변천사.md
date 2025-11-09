---
template: "TIL"
title: "Atomic CSS 및 CSS 방법론 변천사"
created_at: "2025-11-09 16:45"
created_by:
  name: "박기쁜"
  email: "gbpark@herit.net"
participants: []
tags:
  - "Atomic CSS"
  - "CSS 방법론"
  - "BEM"
  - "Tailwind CSS"
  - "CSS-in-JS"
category: "TIL"
status: "draft"
visibility: "internal"
related_docs: []
custom: {}
---

# TIL (Today I Learned)

## 📚 학습 주제
Atomic CSS 및 CSS 방법론에 대한 이해와 적용 방안 모색 (2025-11-03)

---

## 🔑 핵심 내용
- Atomic CSS의 개념과 Traditional CSS와의 차이점 이해
- 다양한 CSS 방법론(SMACSS, OOCSS, BEM, ITCSS)의 특징과 장단점 비교
- CSS-in-JS 및 Vanilla Extract 등 최신 CSS 기술 동향 파악
- Atomic CSS 라이브러리(Tailwind CSS, Windi CSS, UnoCSS, Tachyons) 비교 분석
- Vue/React/React Native 환경에서의 Atomic CSS 적용 전략 고민

---

## 📝 상세 내용
CSS 방법론과 Atomic CSS에 대해 학습하고, 실제 프로젝트에 적용할 수 있는 방안을 고민했습니다.

### CSS 방법론의 진화
HTML부터 시작하여 CSS, SCSS를 거쳐 다양한 CSS 방법론이 등장하게 된 배경과 각각의 특징을 살펴보았습니다.

1.  **HTML**: 컨텐츠에 의미를 부여하는 태그를 사용하여 서식을 꾸미는 방식
2.  **Inline-style**: 태그에 직접 스타일을 지정하는 방식 (가독성 및 유지보수 어려움)
3.  **CSS**: Inline-style의 중복을 제거하고 컨텐츠와 서식을 분리
4.  **SCSS**: 변수, 중첩, mixin 등을 활용하여 복잡한 selector 관리
5.  **Ajax**: 비동기적 정보 교환 기법을 통해 페이지 깜빡임 없이 데이터 업데이트

### 주요 CSS 방법론 비교
다양한 CSS 방법론들의 특징을 비교 분석했습니다.

| 방법론  | 특징                                                                   | 장점                                                               | 단점                                                                    |
| :-------- | :--------------------------------------------------------------------- | :----------------------------------------------------------------- | :---------------------------------------------------------------------- |
| SMACSS    | CSS를 범주화하여 패턴화                                                     | !important, id 셀렉터 사용 금지, 의미있는 class naming                        |                                                                         |
| OOCSS     | CSS를 모듈 방식으로 작성, 구조와 스타일 분리                                           | 공통된 부분 재활용 가능                                                     | 다중 클래스 사용으로 가독성 저하                                                  |
| BEM       | Block, Element, Modifier로 구분하여 클래스 이름 작성                                 | 어떠한 목적인가에 초점                                                     |                                                                         |
| ITCSS     | 스타일을 계층적으로 구성 (settings -> tools -> generic -> elements -> objects -> components -> utilities) | 일반적인 스타일에서 구체적인 스타일로 점진적 적용                                      |                                                                         |

### CSS Modules & CSS-in-JS
CSS Modules와 CSS-in-JS의 등장 배경과 특징을 이해했습니다.

- **CSS Modules**: Component 단위에서 사용되는 CSS에 hash를 추가하여 전역 변수 문제를 해결
- **CSS-in-JS (Styled Components)**: JS 컴포넌트 방식으로 CSS 작성

```javascript
// Styled Components 예시
import styled from 'styled-components';

const Button = styled.button`
  background-color: blue;
  color: white;
  padding: 10px 20px;
  border-radius: 5px;
`;
```

### Vanilla Extract
Vanilla Extract의 등장 배경과 장점을 이해했습니다.

- **Vanilla Extract**: 빌드 타임에 TS 파일을 CSS 파일로 변환하는 zero-runtime CSS in JS
- type-safe한 테마 관리, 프레임워크에 독립적인 스타일링 가능, variant 기반 스타일링

### Atomic CSS
Atomic CSS의 개념과 장단점을 학습했습니다.

- **Atomic CSS**: 의미 기반이 아닌 미리 만들어둔 시각적인 이름을 바탕으로 스타일 적용
    - 예: `.bg-gray-100`, `.w-32`, `.text-center`
- **장점**: 쉬운 이름, 재사용성, 컨벤션 맞추기 용이
- **단점**: HTML에 스타일 코드 반복, 의미론적 구분 부재, 테마 서식 적용 어려움, HTML 가독성 저하

### Atomic CSS 변천사
FunctionalCSS > Atomic CSS > Utility-first -> On-demand Atomic CSS 로의 발전 과정을 이해했습니다.
* **FunctionalCSS** : CSS 클래스가 기능 중심으로 단일 스타일 속성만 담당하도록 설계된 스타일링 방식 (.m-4, .text-red)
* **Utility-first(Tailwind.css)** : 상속보다는 컴포넌트를 선호하라. 유틸리티(단위 기능) 클래스 중심으로 스타일링. *이미 만들어진 스타일 클래스*를 조합해서 사용
```html
<!-- 유틸리티 클래스만으로 구성된 버튼 -->
<button class="bg-blue-500 text-white px-4 py-2 rounded">
  클릭
</button>
```
* **On-Demand Atomic CSS / On-Demand Utility CSS(Windi CSS, UnoCSS)** : 필요한 유틸리티 클래스만 *사용될 때, 요구될 * 런타임 혹은 빌드 타임에 생성하거나 포함

### Atomic CSS 라이브러리 비교

| 라이브러리  | 특징                                                                       | 장점                                                                                                   | 단점                                                                        |
| :------------ | :------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------- |
| Tailwind CSS  | Utility-First, JIT (Just-In-Time)                                              | 빌드 시 사용되는 클래스만 수집하여 사용                                                                                     |                                                                             |
| Windi CSS     | Tailwind 호환 + Atomic + On-Demand                                             |                                                                                                        |                                                                             |
| UnoCSS        | 완전 동적 빌드, Tailwind 호환, 단축형/맞춤 규칙 가능                                    | 페이지에서 실제로 필요한 순간에 CSS 생성                                                                                     |                                                                             |
| Tachyons      | 초창기 Atomic / Functional CSS 프레임워크                                          |                                                                                                        |                                                                             |

---

## 💡 새롭게 알게 된 점
- CSS 방법론에도 깊은 철학이 담겨 있다는 점을 알게 되었습니다.
- 기존 HZEMS 프로젝트의 다양한 방법론 혼재로 인한 유지보수 어려움을 명확히 인지하게 되었습니다.

---

## 🤔 궁금한 점 / 추가 학습 필요
- 각 CSS 방법론의 구체적인 적용 사례 연구 필요
- Atomic CSS 라이브러리들의 성능 비교 및 프로젝트 적용 시 고려사항 학습 필요
- Vue/React/React Native 프로젝트에서 Atomic CSS를 효율적으로 사용하는 방법 연구 필요

---

## 🔗 참고 자료
- [Atomic CSS에 대한 Velog 글](https://velog.io/@furium/Atomic-CSS)
- [AJAX에 대한 나무위키](https://namu.wiki/w/AJAX)
- [CSS 방법론(OOCSS, BEM, SMACSS) 비교](https://velog.io/@hahan/CSS%EB%B0%A9%EB%B2%95%EB%A1%A0OOCSS-BEM-SMACSS)
- [카카오 기술 블로그](https://tech.kakao.com/posts/518)
- [Sweeb's Tistory](https://sweeb.tistory.com/8)
- [Atomic CSS, utility-first CSS](https://yozm.wishket.com/magazine/detail/1326/)