---
template: "TIL"
title: "React Native (Expo) Sentry 통합 설정 가이드"
created_at: "2026-02-11 08:19"
created_by:
  name: "gbpark"
  email: "gbpark@herit.net"
participants: []
tags:
  - "Sentry"
  - "React Native"
  - "Expo"
  - "에러 모니터링"
  - "설정"
category: "TIL"
status: "draft"
visibility: "internal"
related_docs: []
custom: {}
---

# TIL (Today I Learned)

## 📚 학습 주제
Sentry React Native (Expo) 통합 및 고급 설정 옵션

---

## 🔑 핵심 내용
-   **자동 설치 및 기본 설정:** `npx @sentry/wizard`를 통해 React Native 프로젝트에 Sentry SDK를 자동으로 설정하고, `Podfile`, `AppDelegate`, `build.gradle` 등의 네이티브 파일에 필요한 코드를 주입하는 과정을 학습했습니다.
-   **초기화 (`Sentry.init`) 상세 설정:** DSN 연결, 개인 식별 정보(PII) 수집 (`sendDefaultPii`), 로그 활성화 (`enableLogs`), 세션 리플레이 (`replaysSessionSampleRate`, `replaysOnErrorSampleRate`), 모바일 리플레이 및 피드백 통합 (`Sentry.mobileReplayIntegration`, `Sentry.feedbackIntegration`) 등 Sentry SDK의 핵심 초기화 옵션들을 이해했습니다.
-   **Expo 및 Metro 플러그인 통합:** `app.config.ts`에 `@sentry/react-native/expo` 플러그인을 추가하고, `metro.config.ts`에 Sentry Metro Serializer를 적용하여 빌드 시 소스맵 업로드 및 디버그 ID 삽입을 자동화하는 방법을 익혔습니다.
-   **이벤트 기록 및 그룹화 제어:** `Sentry.captureMessage`, `Sentry.captureException`을 통한 이벤트 전송 방법과 더불어, `level` 설정, `event.origin` 및 `event.environment` 태그 활용, 그리고 `fingerprint`를 이용한 이벤트 그룹화 커스텀 전략을 학습했습니다.
-   **다양한 이벤트 데이터 확장 옵션:** `Attachments`, `Breadcrumbs`, `Context`, `Event Processors`, `Scope`, `Screenshots`, `Tags`, `Users`, `View Hierarchy` 등 Sentry가 제공하는 풍부한 이벤트 데이터 기록 및 제어 옵션들을 파악했습니다.

---

## 📝 상세 내용

### 🛠️ 1. Sentry 자동 설치 마법사 (`npx @sentry/wizard`)
React Native 프로젝트에 Sentry SDK를 빠르고 쉽게 통합하기 위한 마법사 명령어를 학습했습니다.

```bash
npx @sentry/wizard@latest -i reactNative --saas --org herit-75 --project echal-app-frontend
```

-   `npx @sentry/wizard@latest`: 최신 버전의 Sentry 설정 마법사를 실행합니다.
-   `-i reactNative`: 프로젝트 플랫폼이 React Native임을 지정합니다.
-   `--saas`: 자체 서버가 아닌 Sentry 공식 클라우드(sentry.io) 서비스를 사용합니다.
-   `--org herit-75`: Sentry 대시보드의 조직 슬러그가 `herit-75`임을 명시합니다.
-   `--project echal-app-frontend`: 이슈를 기록할 특정 프로젝트 이름을 지정합니다.

**설정 마법사의 주요 기능:**

1.  **네이티브 설정 자동화:** iOS (`Podfile`, `AppDelegate`) 및 Android (`build.gradle`) 파일에 Sentry SDK가 동작하도록 필요한 코드를 주입합니다.
    *   **Podfile:** iOS용 외부 라이브러리 설치 목록을 관리하는 파일입니다. Sentry와 같은 외부 도구를 설치할 때 iOS 전용 기능들을 가져와 프로젝트에 연결하는 역할을 합니다.
    *   **AppDelegate:** 앱의 시작과 끝을 관리하는 iOS의 핵심 파일입니다. 앱이 켜질 때, 꺼질 때, 백그라운드로 갈 때 어떤 동작을 할지 정의합니다.
2.  **Sentry 설정 파일 생성:** 프로젝트 루트 및 `ios`, `android` 폴더에 `sentry.properties` 파일을 생성하여 인증 토큰과 프로젝트 정보를 저장합니다.
3.  **소스맵 업로드 설정:** 앱이 빌드될 때 JavaScript 코드를 Sentry가 읽을 수 있는 형태로 변환하여 업로드하는 스크립트를 빌드 단계에 추가합니다.
4.  **연결 확인:** 모든 설정이 완료된 후, 샘플 에러를 발생시켜 Sentry 대시보드에 실제로 기록되는지 테스트할 기회를 제공합니다.

### ⚙️ 2. Sentry 초기화 (`app/_layout.tsx`)
React Native 애플리케이션의 레이아웃 파일에서 Sentry SDK를 초기화하고 다양한 옵션을 설정하는 방법을 학습했습니다.

```typescript
import { Stack } from 'expo-router';
import { StatusBar } from 'expo-status-bar';

import { AppProvider } from '@/app/providers';
import * as Sentry from '@sentry/react-native';

Sentry.init({
	// 1. 기본 연결 및 데이터 수집
	// dsn: sentry 서버로 데이터를 보내기 위한 고유 주소.
	// dsn 은 .env 파일에 보관하는 것이 좋으며 빌드 및 배포를 위한 설정을 위해서 eas.json 에 넣어주면 됌.
  dsn: 'https://14e43f362ea941adb8bd022962047751@o4510655722684416.ingest.us.sentry.io/4510825575022593',

  // Adds more context data to events (IP address, cookies, user, etc.)
  // For more information, visit: https://docs.sentry.io/platforms/react-native/data-management/data-collected/
  // sendDefaultPii : pii 는 Personal Identifiable Information(개인 식별 정보)의 약자.
  // 이 옵션을 켜면 에러가 났을 때 사용자의 ip 주소 같은 데이터를 함께 수집해, 어떤 사용자가 에러를 겪었는지 더 구체적으로 파악하게 해줌
  sendDefaultPii: true,

  // enableLogs : sentry가 정상적으로 작동하고 있는지 개발자 도구에 로그를 남김. 초기 설정 단계에서 문제가 없는지 확인할 때 유용
  // Enable Logs
  enableLogs: true,

  // 2. 세션 리플레이 (replaySessionSampleRate, replayOnErrorSampleRate)
  // Configure Session Replay

  // replaysSessionSampleRate: 0.1 = 전체 사용자 세션 중 10%만 무작위로 리플레이를 기록
  replaysSessionSampleRate: 0.1,
  // replaysOnErrorSampleRate: 1 = 에러가 발생했을 경우에 100% 확률로 그 직전 상황을 녹화. 개발자가 에러를 재현하는데 결정적인 도움을 줌
  replaysOnErrorSampleRate: 1,

  // 3. 통합 기능(Sentry.mobileReplayIntegration, Sentry.feedbackIntegration)
  // Sentry.mobileReplayIntegration = 위에서 설명한 모바일 화면 리플레이 기능을 활성화
  // Sentry.feedbackUntegration = 사용자가 에러 발생 시 의견을 보낼 수 있는 피드백 ui 기능을 준비
  integrations: [Sentry.mobileReplayIntegration(), Sentry.feedbackIntegration()],

  // uncomment the line below to enable Spotlight (https://spotlightjs.com)
  // spotlight: __DEV__ ===> 개발 모드에서 sentry 대시 보드에 접속하지 않고도 로컬 환경에서 즉시 에러를 확인할 수 있게 해주는 도구.
  // spotlight: __DEV__,
});

export const unstable_settings = {
  initialRouteName: '(tabs)',
};

// Sentry.wrap(RootLayout(){}) : RootLayout 전체를 Sentry.wrap()으로 감싸기 ===> 자동 추적, 성능 모니터링
export default Sentry.wrap(function RootLayout() {
  return (
    <AppProvider>
      <Stack>
        <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
      </Stack>
      <StatusBar style="auto" />
    </AppProvider>
  );
});
```

-   **DSN (Data Source Name):** Sentry 서버로 데이터를 보내기 위한 고유 주소입니다. `.env` 파일에 보관하고 `eas.json`에서 설정하는 것이 권장됩니다.
-   **`sendDefaultPii`:** PII (Personal Identifiable Information)는 개인 식별 정보의 약자입니다. 이 옵션을 `true`로 설정하면 사용자의 IP 주소 같은 데이터를 함께 수집하여, 어떤 사용자가 에러를 겪었는지 더 구체적으로 파악할 수 있게 해줍니다.
-   **`enableLogs`:** Sentry가 정상적으로 작동하고 있는지 개발자 도구에 로그를 남깁니다. 초기 설정 단계에서 문제 확인에 유용합니다.
-   **세션 리플레이 (`replaysSessionSampleRate`, `replaysOnErrorSampleRate`):**
    *   `replaysSessionSampleRate: 0.1`은 전체 사용자 세션 중 10%만 무작위로 리플레이를 기록하도록 설정합니다.
    *   `replaysOnErrorSampleRate: 1`은 에러가 발생했을 경우 100% 확률로 에러 직전 상황을 녹화하여 개발자가 에러를 재현하는 데 결정적인 도움을 줍니다.
-   **통합 기능 (`integrations`):**
    *   `Sentry.mobileReplayIntegration()`: 모바일 화면 리플레이 기능을 활성화합니다.
    *   `Sentry.feedbackIntegration()`: 사용자가 에러 발생 시 의견을 보낼 수 있는 피드백 UI 기능을 준비합니다.
-   **`Sentry.wrap(RootLayout())`:** `RootLayout` 컴포넌트 전체를 `Sentry.wrap()`으로 감싸면, Sentry가 자동 추적 및 성능 모니터링을 수행할 수 있도록 합니다.

### 🔌 3. Sentry Expo 플러그인 추가 (`app.json` 또는 `app.config.ts`)
Expo 프로젝트에서 Sentry를 더욱 효과적으로 연동하기 위해 플러그인을 추가하는 방법을 학습했습니다.

**`app.json` 설정 예시:**
```json
// app.json
{
  "expo": {
    "plugins": [
      [
        "@sentry/react-native/expo",
        {
          "url": "https://sentry.io/",
          "note": "Use SENTRY_AUTH_TOKEN env to authenticate with Sentry.",
          "project": "example-project",
          "organization": "example-org"
        }
      ]
    ]
  }
}
```

**`app.config.ts(js)` 설정 예시:**
`app.config.ts` 파일에서 환경별 설정을 관리하면서 `withSentry`를 통해 Sentry 플러그인을 적용하는 방법을 학습했습니다.

```typescript
import { ConfigContext, ExpoConfig } from 'expo/config';
import { withSentry } from '@sentry/react-native/expo'

export default ({ config }: ConfigContext): ExpoConfig => {
  // 1. 기본 환경 변수 설정 (기본값은 development)
  const env = process.env.MY_ENVIRONMENT || 'development';

  // 2. 모든 환경 공통 설정 (app.json의 기본값 활용)
  const finalConfig: ExpoConfig = {
    ...config,
    name: config.name || '기본 앱 이름',
    slug: config.slug || 'echal-app-frontend',
  };

  // 3. 환경별 조건부 설정
  if (env === 'production') {
    return {
      ...finalConfig,
      name: 'production 앱 이름',
      ios: { ...finalConfig.ios, bundleIdentifier: 'com.company.app' },
      extra: { apiUrl: 'https://api.production.com' },
    };
  }

  if (env === 'preview') {
    return {
      ...finalConfig,
      name: 'preview 앱 이름',
      ios: { ...finalConfig.ios, bundleIdentifier: 'com.company.app.preview' },
      extra: { apiUrl: 'https://api.preview.com' },
    };
  }

  // 4. 그 외 상황 (development 등) - 최종 Fallback
  // withSentry : 마지막 return 문에서 finalConfig를 withSentry 로 감쌈. 이렇게 하면 빌드 시점에 Sentry가 앱의 설정을 가로채서 필요한 네이티브 코드와 소스맵 업로드 스크립트를 자동으로 심어줌.
  return withSentry(finalConfig, {
    url: "https://sentry.io/",
    project: "echal-app-frontend",
    organization: "herit-75",
  });
};
```
-   `withSentry`: 최종 `ExpoConfig`를 `withSentry`로 감싸면, 빌드 시점에 Sentry가 앱의 설정을 가로채 필요한 네이티브 코드와 소스맵 업로드 스크립트를 자동으로 주입합니다.

### 🔗 4. Sentry Metro 플러그인 추가 (`metro.config.ts`)
Metro Bundler 설정에 Sentry Serializer를 추가하여 빌드된 번들과 소스 맵에 고유한 디버그 ID가 할당되도록 하는 방법을 학습했습니다.

```typescript
// metro.config.ts
// 생성된 번들러와 소스 맵에 고유한 디버그 ID가 할당되도록 하려면 Metro 구성에 Sentry Serializer를 추가.
const {
  getSentryExpoConfig
} = require("@sentry/react-native/metro");

/** @type {import('expo/metro-config').MetroConfig} */
// 빌드할 때 sentry 가 소스맵을 생성하고 코드의 어느 부분이 에러가 났는지 추적하기 위한 디버그 id를 자동으로 삽입
const config = getSentryExpoConfig(__dirname);

// SVG Transformer 설정
// .svg 파일을 만났을 때 이를 이미지 파일이 아닌 React 컴포넌트로 변환해서 읽으라는 지시.
// 앱에서 <Logo /> 처럼 svg를 컴포넌트 형태로 사용할 수 있게 함.
config.transformer = {
  ...config.transformer,
  babelTransformerPath: require.resolve('react-native-svg-transformer'),
};

// assetExts: 기본적으로 svg를 일반 이미지 자산으로 취급. 여기서 svg 를 제거하여 일반 이미지 취급을 받지 않도록.
// sourceExts: 대신 svg를 소스코드 목록에 추가.
config.resolver = {
  ...config.resolver,
  assetExts: config.resolver.assetExts.filter((ext) => ext !== 'svg'),
  sourceExts: [...config.resolver.sourceExts, 'svg'],
};

module.exports = config;
```
-   `getSentryExpoConfig(__dirname)`: 빌드 시 Sentry가 소스 맵을 생성하고 코드의 어느 부분이 에러가 났는지 추적하기 위한 디버그 ID를 자동으로 삽입하도록 Metro 설정을 구성합니다.
-   **SVG Transformer 설정:** `.svg` 파일을 React 컴포넌트로 변환하여 앱에서 SVG를 컴포넌트 형태로 사용할 수 있도록 합니다. `assetExts`에서 `svg`를 제거하고 `sourceExts`에 추가하여 일반 이미지 자산이 아닌 소스 코드로 처리하게 합니다.

### 🚀 5. 설정 확인
Sentry 설정 후, 실제 메시지를 전송하여 대시보드에 기록되는지 확인하는 방법을 학습했습니다.

```typescript
<Button.Primary
        onPress={() => {
          console.log("Sentry 핑 보내는 중...");
          Sentry.captureMessage("Test Message from Local"); // Sentry로 전송해야 하는 텍스트 정보
          alert("메시지를 보냈습니다. 터미널 로그를 확인하세요.");
        }}
      >
```
-   `Sentry.captureException(err)`: 오류를 자동으로 포착하여 Sentry에 전송합니다.
-   `Sentry.captureMessage("~~")`: Sentry로 전송하고 싶은 텍스트 정보를 수동으로 보냅니다.
-   이벤트가 전송은 되지만 드롭되는 경우, Sentry 계정의 `usage` (할당량)을 확인해야 합니다.

### 🚦 6. 이벤트 레벨 설정
Sentry에 기록되는 이벤트의 중요도를 `level`로 설정하는 방법을 학습했습니다. 레벨은 일반적으로 통합 설정에 따라 기본적으로 추가되지만, 수동으로 지정할 수도 있습니다.

**사용 가능한 레벨:**
`fatal` | `error` | `warning` | `log` | `info` | `debug`

**범위(`Scope`) 내에서 레벨 설정:**
```typescript
Sentry.getCurrentScope().setLevel("warning");
```

**이벤트별로 레벨 설정:**
`withScope`를 사용하여 특정 에러에만 임시 설정을 적용할 수 있습니다.
```typescript
// 1. 특정 에러에만 임시 설정을 적용하기 위해 'withScope' 생성
Sentry.withScope(function (scope){
	// 이 scope 안에서 발생하는 모든 이벤트의 레벨을 "info"로 설정
	// 기본적으로 captureException은 'error' 레벨.
	scope.setLevel("info");

	// 이 에러는 info 레벨을 달고 sentry 대시보드로 날아감.
	Sentry.captureException(new Error("custom error"));
})

// scope 밖에서 설정된 메세지는 'error' 레벨로 날아감.
Sentry.captureException(new Error("custom erorr 2"));
```

### 🌍 7. 이벤트 Origin과 Environment 태그
Sentry 이벤트가 발생한 근원지와 기술적 배경을 구분하는 `event.origin`과 `event.environment` 태그를 학습했습니다.

| 태그              | 설명                                                      | 예시 값                                             |
| :---------------- | :-------------------------------------------------------- | :-------------------------------------------------- |
| `event.origin`    | 에러가 발생한 근원지 (OS/플랫폼)                          | `javascript`: RN JS 엔진(Hermes 등) 내부 논리 에러 |
|                   |                                                           | `android`: 안드로이드 기기 환경                     |
|                   |                                                           | `ios`: iOS 기기 환경                                |
| `event.environment` | 에러가 발생한 기술적 배경 (언어/환경)                   | `javascript`: `.js`, `.tsx` 코드에서 발생한 에러    |
|                   |                                                           | `java`: 안드로이드 전용 코드(Java/Kotlin) 에러      |
|                   |                                                           | `native`: iOS 전용 코드(Objective-C/Swift) 또는 C++ 등 기기 네이티브 단 에러 |

### 🔍 8. 핑거프린팅 (Fingerprinting)
여러 에러를 하나의 이슈로 그룹화하는 `핑거프린팅` 개념과 이를 커스텀하는 방법을 학습했습니다. Sentry는 에러 메시지, 스택 트레이스 등을 분석해 자동으로 지문을 생성하지만, 완벽하지 않을 때가 있어 커스텀이 필요할 수 있습니다.

**1. 에러를 더 세분화해서 나누고 싶을 때:**
API 요청 에러처럼 코드 위치가 항상 같아 하나로 묶이기 쉬운 경우, HTTP 메서드나 경로를 지문에 추가하여 각각 별도의 이슈로 관리할 수 있습니다.

```typescript
Sentry.withScope(function (scope){
	// 메서드, 경로, 상태 코드를 지문에 추가
	// 이렇게 하면 'GET /user 500'과 'POST /login 500'이 분리됨
	scope.setFingerprint([method, path, String(err.statusCode), '{{ default }}']);
	Sentry.captureException(err);
})
```
-   `{{ default }}`를 배열에 넣으면 "기본 알고리즘 + 내가 추가한 값"으로 계산되어 더 세분화된 그룹화를 가능하게 합니다.

**2. 에러를 강제로 하나로 합치고 싶을 때:**
관리 차원에서 'DB 오류'를 하나의 이슈로 보고 싶을 때와 같이 특정 에러를 강제로 그룹화할 수 있습니다.

```typescript
Sentry.init({
	dsn: "YOUR_DSN",
	beforeSend: function(event, hint){
		const exception = hint.originalException;
		// `hint.originalException`을 통해 원본 예외 객체에 접근
		if(exception instanceof DatabaseConnetionError){ // 'DatabaseConnetionError'는 예시 클래스
			// {{ default }}를 빼고 고정된 문자열만 넣으면 무조건 하나로 묶임.
			event.fingerprint = ["database-connection-error"];
		}
		return event;
	}
})
```
-   `{{ default }}`를 제외하고 고정된 문자열만 사용하면 Sentry의 기본 분석을 무시하고 개발자가 정한 규칙으로만 에러를 그룹화합니다.

### 📝 9. 자세하게 이벤트 기록하는 옵션들
Sentry가 제공하는 다양한 고급 옵션들을 통해 에러 발생 시 더 풍부한 정보를 수집하고 디버깅 효율을 높이는 방법을 학습했습니다.

1.  **Attachments (첨부파일) 📎**
    *   에러 발생 시 로그 파일, JSON 설정 값, 텍스트 파일 등을 함께 보냅니다. 앱의 내부 데이터베이스 덤프나 상세 로그 확인에 유용합니다.
    *   `Sentry.getCurrentScope().addAttachment()` 또는 `beforeSend`, `addEventProcessor`에서 추가할 수 있습니다.
    *   크기 초과 시 `HTTP 413 Payload Too Large` 오류가 발생하며 데이터는 즉시 삭제됩니다.
    *   **참고:** 첨부된 파일은 할당량에 영향을 미칩니다.

2.  **Breadcrumbs (흔적) 🐾**
    *   에러 직전 5~10분 동안 사용자가 수행한 활동 기록(클릭, 페이지 이동, API 요청 등)을 추적합니다. 어떻게 해서 이 에러까지 도달했는가'라는 경로를 파악하는 데 결정적인 역할을 합니다.
    *   `Sentry.addBreadcrumb()`을 통해 수동으로 추가하거나, `beforeBreadcrumb` 옵션을 통해 특정 Breadcrumbs를 필터링할 수 있습니다.

3.  **Context (문맥) 📑**
    *   임의의 데이터(객체, 리스트 등)를 에러에 붙입니다. 장바구니 상태, 게임 캐릭터 스탯 등 검색은 안 되지만 상세 페이지에서 확인해야 할 정보에 유용합니다.
    *   `Sentry.setContext("key", { ...data })`로 설정합니다. 최대 페이로드 크기를 초과하면 `413 payload too large`가 반환됩니다.

4.  **Event Processors (이벤트 처리기) ⚙️**
    *   에러가 서버로 전송되기 직전에 실행되는 커스텀 함수입니다. 모든 에러에서 특정 민감 정보를 지우거나 공통적인 데이터를 강제로 주입할 때 사용됩니다.
    *   특정 조건에서 `null`을 반환하여 해당 에러를 전송하지 않고 버릴(drop) 수 있습니다.
    *   `Sentry.addGlobalEventProcessor()`로 전역 설정하거나 `scope.addEventProcessor()`로 특정 범위에 설정할 수 있습니다. `beforeSend`와 달리 실행 순서가 보장되지 않습니다.
    *   **활용 예시:** 개인정보 비식별화, 동적 데이터 주입, 에러 필터링.

5.  **Scope (스코프) 🎯**
    *   Sentry 데이터의 유효 범위입니다. `withScope`를 사용하면 특정 함수나 화면 안에서만 적용될 태그나 유저 정보를 관리하고, 작업이 끝나면 자동으로 초기화됩니다. 이는 일시적인 컨텍스트를 제공하는 데 매우 유용합니다.

6.  **Screenshots (스크린샷) 📸**
    *   에러가 발생한 그 순간의 앱 화면을 캡처해서 보냅니다. UI 레이아웃이 깨졌거나, 사용자가 무엇을 보고 있었는지 시각적으로 바로 확인할 수 있습니다.

7.  **Tags (태그) #️⃣**
    *   `key/value` 형태의 짧은 문자열로, 대시보드에서 필터링과 검색이 가능합니다 (예: `ios_version: 17.0`). 에러를 분류하고 빠르게 찾을 때 핵심적인 역할을 합니다.

8.  **Transaction Name (트랜잭션 이름) 🏷️**
    *   현재 작업의 이름을 설정합니다 (예: `'GET /api/user'`). 성능 측정 시 어떤 페이지나 기능에서 시간이 많이 걸리는지 구분하는 기준이 됩니다.

9.  **Users (사용자) 👤**
    *   에러를 겪은 사용자의 ID, 이메일, IP 등을 설정합니다. 이 에러가 특정 사용자에게만 반복되는지 확인하고, 에러 해결 후 해당 유저에게 피드백을 줄 때 사용됩니다.

10. **View Hierarchy (뷰 계층 구조) 🌳**
    *   에러 시점의 UI 컴포넌트 구조(DOM/View Tree)를 3D 또는 트리 형태로 보여줍니다. 특정 버튼이 화면 밖으로 나갔거나, 다른 뷰에 가려져서 발생한 클릭 에러 등을 디버깅하는 데 시각적인 도움을 줍니다.

### ➕ 10. 확장 Config 옵션
`Sentry.init` 함수에 전달할 수 있는 다양한 확장 설정 옵션들을 학습했습니다.

1.  **Options:** `dsn`, `debug`, `enabled`, `attachScreenshot` 등 Sentry SDK의 전반적인 동작 방식을 결정하는 모든 설정값입니다.
2.  **WebView:** React Native 앱 안에 포함된 웹뷰 내부의 에러를 잡는 설정입니다. 웹뷰 내부 JavaScript에서 에러가 발생했을 때 이를 네이티브 에러와 연결하여 추적합니다.
3.  **App Hangs (앱 행/프리징):** 앱이 비정상적으로 종료되지는 않았지만, 사용자 입력에 반응하지 않고 멈춰버린 상태를 감지하고 보고하는 기능입니다. 주로 메인 스레드가 일정 시간 응답하지 않을 때 발생합니다.
4.  **Environments:** 에러가 발생한 서버 환경을 태그로 분류합니다 (예: `production`, `staging`, `development`). 로컬 개발 중 터진 에러와 실제 유저가 겪는 에러를 대시보드에서 섞이지 않게 필터링하는 기준이 됩니다.
5.  **Releases & Health:** 앱의 특정 버전을 명시하고 해당 버전의 무충돌율 (crash-free rate)을 측정합니다. 특정 업데이트 이후 앱이 더 많이 충돌하는지 등을 숫자로 확인할 수 있습니다.
6.  **Sampling:** 발생하는 모든 이벤트를 다 보낼지, 아니면 일부만 보낼지 비율을 정하여 할당량을 관리합니다.
7.  **Filtering:** 특정 에러를 전송하지 않도록 걸러냅니다. `beforeSend` 또는 `ignoreErrors` 옵션을 사용하여 무의미한 에러가 할당량을 잡아먹지 않게 막습니다.
8.  **Shutdown and Draining:** 앱이 예기치 않게 종료될 때 아직 서버로 전송되지 못한 에러 데이터를 어떻게 처리할지 결정합니다. 최대한 데이터를 안전하게 전송하고 종료되도록 대기 시간을 설정할 수 있습니다.
9.  **Tracking Touch Events:** 사용자가 화면의 어느 버튼을 눌렀는지 등을 자동으로 브레드크럼에 남겨, 사용자의 인터랙션 흐름을 파악하는 데 도움을 줍니다.

---

## 💡 새롭게 알게 된 점
-   `npx @sentry/wizard` 명령어를 통해 React Native 프로젝트에 Sentry를 자동 설치할 때, iOS의 `Podfile`과 `AppDelegate` 파일에 자동으로 코드가 주입되며 이 파일들이 각각 외부 라이브러리 관리 및 앱 생명주기 관리 역할을 한다는 점을 명확히 이해했습니다.
-   `Sentry.init` 시 `sendDefaultPii` 옵션을 통해 사용자의 IP 주소와 같은 개인 식별 정보를 수집할 수 있다는 점, 그리고 `replaysSessionSampleRate`와 `replaysOnErrorSampleRate`를 통해 세션 리플레이 샘플링을 정밀하게 제어할 수 있다는 점이 인상 깊었습니다.
-   Expo 프로젝트에서 `app.config.ts`의 `withSentry` 래퍼와 `metro.config.ts`의 `getSentryExpoConfig`가 빌드 시스템과 통합되어 소스맵 업로드 및 디버그 ID 삽입을 자동화하는 방식이 매우 효율적이라는 것을 깨달았습니다.
-   `event.origin` (발생 근원지 - OS/플랫폼)과 `event.environment` (기술적 배경 - 언어/환경) 태그의 명확한 구분은 에러를 분석하고 필터링하는 데 중요한 기준이 된다는 것을 학습했습니다.
-   Sentry `fingerprinting`에서 `{{ default }}` 키워드를 사용하여 기본 그룹화 로직을 유지하면서 커스텀 조건을 추가하거나, `{{ default }}` 없이 고정된 문자열로 에러를 강제 그룹화하는 전략을 통해 이슈 관리를 유연하게 할 수 있다는 점이 매우 유용했습니다.
-   `Attachments`, `Breadcrumbs`, `Context`, `Event Processors`, `Screenshots`, `View Hierarchy` 등 Sentry가 제공하는 방대한 이벤트 데이터 확장 옵션들이 에러 발생 시 상황을 재현하고 디버깅하는 데 있어 얼마나 강력한 도구가 될 수 있는지 알게 되었습니다. 특히 `View Hierarchy`를 통해 UI 컴포넌트 구조를 시각적으로 확인하는 기능은 UI 관련 에러 분석에 혁신적일 것이라 생각합니다.

---

## 🤔 궁금한 점 / 추가 학습 필요
-   `eas.json`에 DSN 및 기타 Sentry 관련 환경 변수를 어떻게 안전하게 통합하고 관리하는지에 대한 실제 구현 사례를 더 자세히 알아보고 싶습니다.
-   `beforeSend`와 `addEventProcessor` 간의 실행 순서 보장 여부 및 각 옵션의 최적 활용 시나리오에 대해 더 깊이 탐구하고 싶습니다.
-   `Attachments` 사용 시 "보조 저장소 옵션"을 고려하라는 언급이 있었는데, Sentry에서 제공하는 보조 저장소 옵션이 무엇이며, 대용량 첨부 파일을 효율적으로 관리하는 방법에 대해 추가 조사가 필요합니다.
-   `Tracking Touch Events`가 브레드크럼에 자동으로 남기는 정보가 어느 정도이며, 이를 통해 사용자 경험 흐름을 어떻게 시각화하고 분석할 수 있는지 구체적인 예시를 살펴보고 싶습니다.
-   실제 프로덕션 환경에서 `replaysSessionSampleRate` 및 `replaysOnErrorSampleRate`의 최적 값을 설정하는 전략에 대해 더 많은 자료를 찾아보고 싶습니다.

---

## 🔗 참고 자료
-   [작성 필요]

---
**작성자:** gbpark (gbpark@herit.net)
**작성일:** 2023-10-27 10:00
**참가자:** 0명
**태그:** Sentry, React Native, Expo, Error Monitoring, Crash Reporting