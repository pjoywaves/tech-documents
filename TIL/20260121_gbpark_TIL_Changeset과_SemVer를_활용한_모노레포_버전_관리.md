---
template: "TIL"
title: "Changeset과 SemVer를 활용한 모노레포 버전 관리"
created_at: "2026-01-21 15:07"
created_by:
  name: "gbpark"
  email: "gbpark@herit.net"
participants: []
tags:
  - "changeset"
  - "모노레포"
  - "SemVer"
  - "버전 관리"
  - "자동 배포"
category: "TIL"
status: "draft"
visibility: "internal"
related_docs: []
custom: {}
---

# TIL (Today I Learned)

## 📚 학습 주제
모노레포 버전 관리 도구 Changeset과 Semantic Versioning (SemVer)

---

## 🔑 핵심 내용
- `Changeset`은 모노레포 환경에서 상호 의존하는 패키지들의 일관된 버전을 자동으로 관리하고, SemVer 규칙에 따라 레지스트리 배포를 용이하게 하는 도구입니다.
- `changeset`, `changeset version`, `changeset publish` 세 가지 주요 명령어를 통해 변경 사항 감지, 버전 적용 및 CHANGELOG 생성, 그리고 최종 배포 과정을 수행합니다.
- `GitHub Actions`와 `NPM 토큰` (publish 타입, Read and write permissions)을 GitHub Repository Secrets에 등록함으로써 `Changeset`의 자동 배포 워크플로우를 구축할 수 있습니다.
- `Semantic Versioning (SemVer)`은 버전 번호에 의미를 부여하여 변경의 영향 범위를 명확히 하는 규칙으로, Major, Minor, Patch로 구성되며 명확성, 호환성, 자동화를 목표로 합니다.
- SemVer는 버전 불변성, Pre-release, Build metadata 등 다양한 규칙과 개념을 포함하여 체계적인 버전 관리를 지원합니다.

---

## 📝 상세 내용

### Changeset이란? 📦
`Changeset`은 모노레포에서 상호 의존하는 패키지들의 일관성을 유지하기 위한 전문적인 라이브러리입니다. 여러 의존된 패키지들을 업데이트할 때마다 자동으로 [Semantic Versioning (SemVer)](#semantic-versioning-semver-이해하기-) 규칙에 따라 버전을 관리해주며, 간편한 명령어를 통해 레지스트리에 손쉽게 배포할 수 있도록 돕습니다.

```bash
# yarn 사용 시
$ yarn add @changesets/cli && yarn changeset init

# npm 사용 시
$ npm install @changesets/cli && npx changeset init
```

### Changeset 주요 명령어 🚀

`Changeset`은 모노레포 환경에서 버전을 관리하고 배포하는 일련의 워크플로우를 제공합니다.

-   **`changeset`**
    이 커맨드를 입력하면 패키지들의 변경 사항을 감지하고, 해당 변경이 `semver` 규칙에 따라 메이저(Major), 마이너(Minor) 또는 패치(Patch) 버전으로 업데이트될지 질의합니다. 개발자는 변경의 중요도에 따라 적절한 버전을 선택하게 됩니다.

    ```bash
    # yarn 사용 시
    $ yarn changeset

    # npm 사용 시
    $ npx changeset
    ```

-   **`changeset version`**
    배포하기로 결정한 후, 이 명령어를 통해 실제 버전 업데이트를 진행합니다. 설정된 업데이트 규칙에 따라 메이저, 마이너 또는 패치 버전이 증가하며, 해당 패키지에 의존하고 있는 다른 패키지들도 함께 업데이트됩니다. 또한, 각 패키지의 변경 이력을 담는 `CHANGELOG.md` 파일도 자동으로 생성됩니다.

    ```bash
    # yarn 사용 시
    $ yarn changeset version

    # npm 사용 시
    $ npx changeset version
    ```
    이 단계 이후, `changeset publish` 명령어를 사용해 내부적으로 `.npmrc` 파일을 참조하여 레지스트리에 배포할 수 있습니다. 자동 배포를 원한다면 `publish` 단계를 GitHub Actions에 스크립트로 작성하여 `push` 시 수행되도록 설정할 수 있습니다.

-   **`changeset publish`**
    이전 단계에서 수행한 버전 업데이트가 완료된 후, 이 명령어를 실행하면 자동으로 업데이트 예정인 패키지들을 NPM 등의 레지스트리에 배포합니다.

    ```bash
    # yarn 사용 시
    $ yarn changeset publish

    # npm 사용 시
    $ npx changeset publish
    ```

### Changeset 자동 배포 (CI/CD) 설정 ⚙️

`Changeset`은 CI/CD 파이프라인과의 통합을 강력하게 지원하여 자동 배포 시스템 구축을 용이하게 합니다.

-   **GitHub Action**
    `Changeset`은 CI/CD를 위해 GitHub Action 워크플로우 파일을 제공합니다. 이를 통해 PR 머지 등의 특정 이벤트 발생 시 자동으로 버전 업데이트 및 배포를 트리거할 수 있습니다.

-   **NPM Registry 연동**
    GitHub Actions에서 NPM 레지스트리에 패키지를 배포하려면 NPM 토큰이 필요합니다.

    1.  **NPM 토큰 발급:**
        *   NPM 사이트에 접속하여 로그인합니다.
        *   프로필 메뉴에서 `Access Tokens`를 선택하고 새 토큰을 발급받습니다.
        *   토큰 생성 시 **타입은 반드시 `Publish`**로 설정합니다.
        *   토큰 권한은 `Read and write permissions`를 부여합니다.
        *   발급된 토큰 값은 한 번만 보이므로, **필히 별도로 안전하게 보관**해야 합니다.
    2.  **GitHub Repository Secrets 등록:**
        *   발급된 NPM 토큰을 GitHub 레포지토리의 Secret 키로 등록합니다. 이 Secret 키는 GitHub Actions 워크플로우에서 사용될 변수값입니다.
        *   레포지토리의 `Settings` 탭으로 이동합니다.
        *   `Secrets and variables` -> `Actions` 메뉴를 선택합니다.
        *   `New repository secret`을 클릭하여 새로운 Secret을 추가합니다.
        *   `Name` 필드에 Secret 키 이름을 (예: `NPM_TOKEN`) 입력하고, `Secret` 필드에 발급받은 NPM 토큰 값을 입력합니다.

### Semantic Versioning (SemVer) 이해하기 🔢

`Semantic Versioning`은 버전을 관리하는 데 있어 단순한 숫자 나열이 아니라 버전 번호에 의미를 부여함으로써, 사용자(개발자) 모두가 변경이 미치는 영향 범위를 쉽게 파악할 수 있도록 설계된 규칙입니다.

-   **목표:** 명확성, 호환성, 자동화

-   **버전 구성:** `MAJOR.MINOR.PATCH` (예: `4.2.1`)
    | 컴포넌트 | 이름    | 설명                                                 |
    | :------- | :------ | :--------------------------------------------------- |
    | `4`      | `MAJOR` | 하위 버전과 호환되지 않는 큰 변화 (API 변경 및 삭제 등) |
    | `2`      | `MINOR` | 하위 버전과 호환되면서, 새로운 기능 추가 및 개선       |
    | `1`      | `PATCH` | 하위 버전과 호환되면서, 버그 수정                     |

-   **주요 규칙**
    *   각 컴포넌트는 자연수(0 포함)이며, 앞에 0이 붙어서는 안 됩니다 (예: `01` 불가).
    *   각 번호는 항상 증가해야 합니다.
    *   특정 버전으로 패키지를 배포하고 나면, 해당 버전의 내용은 **절대 변경하지 말아야** 합니다. 변경분이 있다면 반드시 새로운 버전으로 배포해야 합니다.
    *   `MAJOR` 버전이 변경될 때, `MINOR`와 `PATCH`는 `0`으로 초기화됩니다.
    *   `MINOR` 버전이 변경될 때, `PATCH`는 `0`으로 초기화됩니다.

-   **주요 개념**
    *   **Backward Compatibility (하위 호환성)**
        *   `Major` 릴리스: 기존 통합을 중단시킬 수 있는 (breaking change) 변화를 포함합니다.
        *   `Minor` 릴리스: 기존 동작을 중단하지 않고 새로운 기능을 추가하거나 개선합니다.
        *   `Patch` 릴리스: 새로운 기능을 추가하거나 기존 동작을 중단시키지 않고 문제를 해결(버그 수정)합니다.
    *   **Pre-release versions (프리-릴리스 버전)**
        *   하이픈(`-`)과 식별자를 추가하여 표시합니다 (예: `1.0.0-alpha`, `1.0.0-beta.1`).
        *   버전이 아직 안정적이지 않으며 테스트용임을 나타냅니다.
    *   **Build Metadata (빌드 메타데이터)**
        *   플러스(`+`)와 메타데이터를 추가하여 표시합니다 (예: `1.0.0+build123`).
        *   빌드 또는 배포 관련 세부 정보에 사용되며, 버전 우선 순위에서는 무시됩니다.

---

## 💡 새롭게 알게 된 점
- `changeset version` 명령어가 단순히 버전을 올리는 것을 넘어, 의존성 패키지들의 버전까지 함께 업데이트하고 `CHANGELOG.md` 파일을 자동으로 생성하는 핵심적인 역할을 수행한다는 점을 명확히 이해했습니다.
- `changeset publish`가 `version` 명령 이후에 실제로 NPM 레지스트리로 배포하는 최종 단계라는 워크플로우를 숙지했습니다.
- NPM 토큰 발급 시 `publish` 타입과 `Read and write permissions`가 필수적이며, 이를 GitHub Repository Secrets에 안전하게 등록하여 자동화된 배포 시스템을 구축하는 구체적인 과정을 학습했습니다.
- SemVer의 Major, Minor, Patch 개념을 넘어, **버전의 불변성**, `pre-release` (예: `1.0.0-alpha`), `build metadata` (예: `1.0.0+build123`)와 같은 상세 규칙과 개념들이 체계적인 버전 관리에 필수적이라는 것을 알게 되었습니다.

---

## 🤔 궁금한 점 / 추가 학습 필요
- 모노레포 내에서 여러 패키지가 서로 다른 `semver` 범주 (예: A는 `patch`, B는 `minor`)로 업데이트될 때 `changeset`이 이를 어떻게 효율적으로 관리하고 충돌을 해결하는지에 대한 심층 분석이 필요합니다.
- `changeset`의 커스텀 설정 옵션 (예: `CHANGELOG` 형식 변경, 배포 전/후 스크립트 추가)에 대해 더 자세히 알아보고 실제 프로젝트에 적용하는 방법을 학습하고 싶습니다.
- GitHub Actions 워크플로우 파일 내부의 상세 설정 (예: 조건부 실행, 에러 처리, 캐싱) 및 잠재적인 보안 취약점 방지 전략에 대해 추가 학습이 필요합니다.

---

## 🔗 참고 자료
- [changesets/action GitHub Repository](https://github.com/changesets/action)