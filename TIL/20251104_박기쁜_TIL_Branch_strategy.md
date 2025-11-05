---
template: "til"
title: "브랜치 전략 정리: Trunk-Based Development 중심 비교"
created_at: "2025-11-05 16:15:00"
created_by:
  name: "박기쁜"
  email: "gbpark@herit.net"
participants: []
tags:
  - "Git"
  - "브랜치 전략"
  - "Trunk-Based Development"
  - "Git Flow"
  - "CI/CD"
category: "TIL"
status: "draft"
visibility: "internal"
related_docs: []
custom: {}
---

# TIL (Today I Learned)

## 📚 학습 주제
브랜치 전략의 종류와 특징을 비교하고, Trunk-Based Development의 개념과 적용 방식을 이해하기

---

## 🔑 핵심 내용

*   브랜치 전략은 협업 효율, 코드 품질, 배포 속도에 직접적인 영향을 미침
*   **context(맥락)**를 줄여 불필요한 의존성을 제거하고, 병합 충돌 및 코드 관리 복잡도를 낮춤
*   대표적인 전략: Git Flow / GitHub Flow / GitLab Flow / Trunk-Based Development
*   **Trunk-Based Development(TBD)**는 작은 단위의 빈번한 병합과 자동화된 테스트를 통해 빠르고 안정적인 배포를 가능하게 함

---

## 📝 상세 내용

### 1️⃣ Git Flow
**명확한 릴리즈 주기와 버전 관리에 최적화된 구조**

| 브랜치 | 역할 |
|--------|------|
| `master` | 제품 배포용 기준 브랜치 |
| `develop` | 개발 통합 브랜치 |
| `feature/*` | 단위 기능 개발 브랜치 |
| `release/*` | QA 및 사전 검증 브랜치 |
| `hotfix/*` | 운영 중 긴급 수정용 브랜치 |

**장점**
- QA 및 테스트 환경 분리 용이
- 안정적 버전 관리 가능

**단점**
- 브랜치 구조 복잡, context 전파 많음
- 소규모 팀에는 관리 부담이 큼

---

### 2️⃣ GitHub Flow
**단순하고 빠른 배포 중심의 전략**

| 브랜치 | 역할 |
|--------|------|
| `main` | 배포 기준 브랜치 |
| `feature/*` | 기능 개발 후 PR → main 병합 |

**특징**
- develop/release/hotfix 제거
- PR 기반 리뷰와 CI 중심의 빠른 배포

**장점**
- 간결하고 효율적인 흐름
- Continuous Delivery에 적합

**단점**
- QA 절차 미흡
- 다수의 기능 병합 시 충돌 위험

---

### 3️⃣ GitLab Flow
**Git Flow와 GitHub Flow의 절충형**

| 구성 | 설명 |
|------|------|
| 환경별 브랜치 (`staging`, `production`) | 실제 배포 환경 단위로 관리 |
| Merge Request 기반 관리 | GitLab MR 프로세스와 결합 |

**장점**
- 환경별 배포 및 테스트 분리 용이
- CI/CD 파이프라인과 높은 호환성

**단점**
- 운영 환경이 많을수록 브랜치 관리 복잡도 증가

---

### 4️⃣ Trunk-Based Development (TBD)
**trunk(mainline) 하나만 유지하고, 빠른 주기로 병합 및 배포**

> “모든 개발자가 하루에 한 번 이상 trunk에 병합해야 한다.”

**핵심 원칙**
- **Small Changes:** 작은 단위로 자주 커밋
- **Quick Rhythm:** 빠른 배포 사이클 유지
- **Continuous Integration:** 자동화된 테스트 필수
- **Continuous Delivery:** 지속적 배포 환경 구축
- **Feature Flags:** 기능 단위로 배포 제어

**장점**
- 병합 충돌 최소화
- 빠른 배포와 QA 효율성 향상
- context 공유 최소화 → 협업 속도 향상

**단점**
- 자동화 테스트 환경 필수
- QA 단계를 대체하기 어렵기 때문에 품질 관리 체계 필요

---

### 💼 우리 팀 적용 방향

| 고려 항목 | 내용 |
|------------|------|
| 팀 규모 | 프론트엔드 + 백엔드가 함께 있는 **에너지 관련 중대규모 프로젝트 팀** |
| 서비스 범위 | **B2B + B2C** 동시 고려 (각 서비스별 배포 주기 및 검증 절차 상이) |
| 배포 주기 | 다빈도 기능 추가 및 지속적 업데이트 중심 |
| 협업 형태 | 여러 개발자가 병렬로 feature 작업 후 공통 mainline으로 통합 |
| 인프라 | GitLab + CI/CD 기반 자동 빌드 및 스테이징 배포 구성 |

**적용 방향**
- **Trunk-Based Development** 전략을 기본으로 운영하되, B2B·B2C 환경 차이를 고려하여 **staging 브랜치**를 별도로 유지
- `main` → production, `staging` → QA/검증용
- feature 브랜치는 단기(1~3일) 단위로 유지 후 병합
- QA는 자동화 테스트 + 스테이징 환경 수동 검증 병행
- 주요 기능은 **feature flag** 기반으로 배포 타이밍 제어

---

## 💡 새롭게 알게 된 점

*   단순히 코드 병합의 문제가 아니라, **팀의 개발 리듬과 배포 문화 전체를 결정하는 게 브랜치 전략**임
*   “브랜치 전략이 생각보다 중요하군…”  
    → 협업 난이도, QA 속도, 고객 대응력까지 구조적으로 바꿀 수 있다는 점이 인상 깊었음
*   Trunk-Based Development는 단일 브랜치 전략이 아니라  
    **CI/CD 자동화와 테스트 문화가 함께 움직여야 완성되는 철학적 접근**이라는 점

---

## 🤔 궁금한 점 / 추가 학습 필요

*   TBD 환경에서 QA 자동화의 구체적 구현 사례 (B2B/B2C 동시 운영 기준)
---

## 🔗 참고 자료

- [Trunk-Based Development 공식 사이트](https://trunkbaseddevelopment.com/)
- [당근마켓 기술 블로그 - 매일 배포하는 팀이 되는 여정](https://medium.com/daangn/%EB%A7%A4%EC%9D%BC-%EB%B0%B0%ED%8F%AC%ED%95%98%EB%8A%94-%ED%8C%80%EC%9D%B4-%EB%90%98%EB%8A%94-%EC%97%AC%EC%A0%95-1-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EC%A0%84%EB%9E%B5-%EA%B0%9C%EC%84%A0%ED%95%98%EA%B8%B0-1a1df85b2cff)
- [LaunchDarkly - Git Branching Strategies vs. TBD](https://launchdarkly.com/blog/git-branching-strategies-vs-trunk-based-development/)
- [Velog - Git branch 전략 정리](https://velog.io/@kw2577/Git-branch-%EC%A0%84%EB%9E%B5)
- [YouTube - Trunk Based Development Explained](https://www.youtube.com/watch?v=EV3FZ3cWBp8)
