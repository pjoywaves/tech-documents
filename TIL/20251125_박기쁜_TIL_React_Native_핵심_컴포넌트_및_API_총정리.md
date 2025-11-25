---
template: "TIL"
title: "React Native 핵심 컴포넌트 및 API 총정리"
created_at: "2025-11-25 11:18"
created_by:
  name: "박기쁜"
  email: "gbpark@herit.net"
participants: []
tags:
  - "React Native"
  - "컴포넌트"
  - "API"
  - "성능 최적화"
  - "접근성"
category: "TIL"
status: "draft"
visibility: "internal"
related_docs: []
custom: {}
---

# TIL (Today I Learned)

## 📚 학습 주제
**React Native 핵심 컴포넌트 및 API 심층 분석**

---

## 🔑 핵심 내용
*   React Native의 `View`, `Text`, `Image`, `TextInput`, `Pressable`, `ScrollView` 등 기본 컴포넌트의 상세 속성 및 사용법을 익혔습니다.
*   `FlatList`와 `SectionList`를 통해 대량의 데이터를 효율적으로 렌더링하고 관리하는 가상화 리스트의 중요성을 이해했습니다.
*   `StyleSheet`의 `create`, `flatten`, `compose` 메서드를 활용한 효과적인 스타일링 전략과 성능 최적화 방법을 학습했습니다.
*   `BackHandler`, `PermissionsAndroid` 등 Android 플랫폼 고유 API와 `ActionSheetIOS` 등 iOS 고유 API 사용법을 파악했습니다.
*   `ActivityIndicator`, `Alert`, `Animated`, `Modal` 등 크로스 플랫폼에서 사용되는 핵심 유틸리티 API를 깊이 있게 다뤘습니다.

---

## 📝 상세 내용

### ⚛️ Core Components and APIs

#### 1. View
UI의 기본 박스 컨테이너로, 레이아웃, 배경, 패딩, 마진 등 시각적 요소를 담당합니다.

*   **언제 쓰는가?**
    *   화면 레이아웃 구성
    *   요소 그룹핑
    *   스타일링(배경, 패딩 등)
    *   Flex 레이아웃 적용
    *   Touch 영역 감싸는 wrapper

*   **주요 속성 / 타입 / 설명**
    *   `style`: `ViewStyle` - `width`, `height`, `padding`, `margin`, `flex` 등 View 스타일
    *   `pointerEvents`: `'auto' | 'none' | 'box-none' | 'box-only'` - 터치 이벤트 전달 방식 제어
    *   `onLayout`: `({nativeEvent: {layout: {width, height, x, y}}}) => void` - View가 렌더링/레이아웃 잡힌 후 호출
    *   `accessibilityLabel`, `accessibilityRole`, `accessibilityHint` 등: 접근성 지원 속성들
    *   `collapsable`: `boolean` (Android) - `false`면 레이아웃 최적화에서 제거되지 않고 항상 존재
    *   `testID`: `string` - 자동화 테스트용 ID
    *   `importantForAccessibility`: `'auto' | 'yes' | 'no' | 'no-hide-descendants'`

```javascript
<View
  style={{ padding: 20, backgroundColor: "#f3f4f6" }}
  onLayout={(e) => console.log(e.nativeEvent.layout)}
  testID="container-view"
>
  <Text>내용</Text>
</View>
```

#### 2. Text
텍스트 렌더링 전용 컴포넌트입니다.

*   **언제 쓰는가?**
    *   글자 출력
    *   버튼 안의 표시 텍스트
    *   멀티라인 텍스트
    *   스타일링(Text 전용 스타일 지원)

*   **주요 속성 / 타입 / 설명**
    *   `style`: `TextStyle` - `fontSize`, `fontWeight`, `lineHeight` 등 텍스트 스타일
    *   `numberOfLines`: `number` - 줄 수 제한
    *   `ellipsizeMode`: `'head' | 'middle' | 'tail' | 'clip'` - 텍스트가 넘칠 때 어떻게 생략할지
    *   `selectable`: `boolean` - 텍스트 선택 가능 여부
    *   `onPress`: `() => void` - 텍스트 클릭 이벤트
    *   `allowFontScaling`: `boolean` - OS 글자 크기 조절 적용 여부
    *   `adjustsFontSizeToFit`: `boolean` - 박스에 맞게 글자 자동 축소
    *   `accessibility props`, `testID` 모두 지원

```javascript
<Text
  numberOfLines={1}
  ellipsizeMode="tail"
  style={{ fontSize: 18, fontWeight: "600" }}
  onPress={() => console.log("텍스트 클릭")}
>
  긴 텍스트를 한 줄로 자르고 말줄임표 붙여줌
</Text>
```

#### 3. Image
이미지를 화면에 표시할 때 쓰는 핵심 컴포넌트입니다. 네트워크 이미지, 앱 번들 내 리소스, 디스크 로컬 이미지 등 다양한 타입이 지원됩니다.

*   **언제 쓰는가?**
    *   앱 내 이미지 리소스를 표시할 때 (`require('./foo.png')`)
    *   네트워크 URL에서 이미지 불러와야 할 때 (`uri: 'https://...'`)
    *   로딩 인디케이터, 배경 이미지, 아이콘 외 이미지 등
    *   리사이즈 모드, 로딩 상태, 오류 처리 등이 필요할 때
    *   이미지 크기나 비율 제어가 필요할 때 (`resizeMode`, `getSize` 등)

*   **주요 속성 / 타입 / 설명**
    *   `accessible`: `boolean` - 접근성 요소로 만들지 여부. (스크린리더 대상) `Default: false`
    *   `accessibilityLabel`: `string` - 이미지가 스크린리더에 의해 읽혀질 때 사용할 문자열.
    *   `alt`: `string` - 이미지의 대체 텍스트. 접근성에 자동으로 이 요소를 “accessible”로 처리함.
    *   `blurRadius`: `number` - 이미지에 적용할 블러 필터 반경.
    *   `capInsets`: `{ top:number, left:number, bottom:number, right:number }` (iOS) - 이미지 리사이즈 시, 테두리 및 코너를 유지하고 중앙 부분만 늘리기 위한 영역 설정.
    *   `crossOrigin`: `'anonymous' | 'use-credentials'` - 이미지 요청 시 CORS 모드 지정.
    *   `defaultSource`: `ImageSourcePropType` - 이미지가 로딩 중일 때 보여줄 로컬 고정 이미지. Android Debug 빌드에서는 무시될 수 있음.
    *   `fadeDuration`: `number` (Android) - 이미지 로딩 후 나타나는 페이드 애니메이션 지속시간(ms). `Default: 300ms`.
    *   `loadingIndicatorSource`: `ImageSourcePropType` - 로딩 중 보여줄 이미지.
    *   `onError`: `({ nativeEvent: { error: any } }) => void` - 이미지 로드 실패 시 호출되는 콜백.
    *   `onLoadStart`: `() => void` - 이미지 로딩 시작 시 호출.
    *   `onLoad`: `({ nativeEvent: { source: { width, height, uri } } }) => void` - 이미지 로딩 성공 시 호출. 이미지 실제 크기 정보 포함.
    *   `onLoadEnd`: `() => void` - 이미지 로딩 성공 또는 실패 후 호출됨.
    *   `onLayout`: `({ nativeEvent: LayoutEvent }) => void` - 컴포넌트 레이아웃 완료 후 호출됨. (크기/위치 정보)
    *   `onProgress`: `({ nativeEvent: { loaded: number, total: number } }) => void` (iOS/Android) - 다운로드 중 진행률 콜백.
    *   `progressiveRenderingEnabled`: `boolean` (Android) - `true`면 Android에서 progressive JPEG 스트리밍 가능하도록 함. `Default: false`.
    *   `resizeMethod`: `'auto' | 'resize' | 'scale' | 'none'` (Android) - 이미지를 뷰 크기에 맞게 어떻게 처리할지 방식 지정. `Default: 'auto'`.
    *   `resizeMode`: `'cover' | 'contain' | 'stretch' | 'repeat' | 'center'` - 이미지가 뷰 영역에 어떻게 맞춰질지 정의. `Default: 'cover'`.
    *   `resizeMultiplier`: `number` (Android) - `resizeMethod`가 'resize'일 때, 디코딩할 이미지 크기에 곱해주는 값. `Default: 1.0`.
    *   `source`: `ImageSourcePropType` - 이미지 리소스 지정. 로컬 리소스(`require`) 또는 URL 객체.
    *   `src`: `string` - 원격 URL을 string 형태로 빠르게 지정. 우선순위 높음.
    *   `srcSet`: `string` - 여러 해상도 후보 URL을 지정하는 문자열. 우선순위 높음.
    *   `style`: `ImageStyle` (및 Layout/Transform 포함) - 이미지 컴포넌트 스타일 지정.
    *   `testID`: `string` - 테스트 자동화에서 이 이미지 요소 식별하기 위한 ID.
    *   `tintColor`: `color` - 이미지 픽셀 중 투명하지 않은 부분의 색상을 이 색상으로 바꿈.
    *   `width`: `number` - 이미지 컴포넌트의 너비 지정 (height도 style 통해 지정 가능)

```javascript
import { Image, View, SafeAreaView, Text } from "react-native";

export default function App() {
  return (
    <SafeAreaView style={{ flex: 1, alignItems: "center", justifyContent: "center" }}>
      <Image
        testID="logo-image"
        accessibilityLabel="앱 로고 이미지"
        style={{ width: 120, height: 120, resizeMode: "contain" }}
        source={{ uri: "https://reactnative.dev/img/tiny_logo.png" }}
        defaultSource={require("./assets/logo-placeholder.png")}
        onLoad={() => console.log("이미지 로드 완료")}
        onError={({ nativeEvent }) => console.log("이미지 로드 실패:", nativeEvent.error)}
      />
      <Text>로고 표시됨</Text>
    </SafeAreaView>
  );
}
```

#### 4. TextInput
사용자 입력을 받을 때 쓰는 핵심 입력 컴포넌트입니다.

*   **언제 쓰는가?**
    *   로그인/회원가입 입력 필드
    *   검색창
    *   숫자, 이메일, 비밀번호 입력
    *   포커스/Blur 제어해야 할 때

*   **주요 속성 / 타입 / 설명**
    *   **기본 입력 관련:** `value`, `defaultValue`, `onChangeText`, `onChange`, `placeholder`, `placeholderTextColor`, `editable`, `selectTextOnFocus`, `selectionColor`, `selection`
    *   **키보드·입력 방식 관련:** `keyboardType`, `returnKeyType`, `blurOnSubmit`, `onSubmitEditing`, `onEndEditing`, `autoCorrect`, `autoComplete`, `autoCapitalize`, `secureTextEntry`, `passwordRules` (iOS), `textContentType`
    *   **포커스 & 제어:** `autoFocus`, `onFocus`, `onBlur`, `caretHidden` (iOS)
    *   **스타일·레이아웃 관련:** `style`, `textAlign`, `textAlignVertical` (Android), `underlineColorAndroid` (Android)
    *   **멀티라인 관련:** `multiline`, `numberOfLines`, `maxLength`, `onContentSizeChange`
    *   **내용 선택/조작:** `selectable`, `contextMenuHidden`, `onSelectionChange`, `selection`
    *   **텍스트 표시 옵션:** `allowFontScaling`, `maxFontSizeMultiplier`, `minimumFontScale`, `adjustsFontSizeToFit`, `lineBreakStrategyIOS`
    *   **입력 제어 (고급):** `inputMode`, `importantForAutofill`, `keyboardAppearance` (iOS), `showSoftInputOnFocus`, `readOnly` (iOS 16+)
    *   **iOS 전용:** `clearButtonMode`, `clearTextOnFocus`, `dataDetectorTypes`, `inputAccessoryViewID`, `rejectResponderTermination`, `scrollEnabled`, `enablesReturnKeyAutomatically`
    *   **Android 전용:** `disableFullscreenUI`, `inlineImageLeft`, `inlineImagePadding`, `returnKeyLabel`, `textBreakStrategy`
    *   **이벤트 관련:** `onKeyPress`, `onScroll`, `onTextInput`
    *   **테스트·접근성:** `testID`, `accessibilityLabel`, `accessible`, `accessibilityHint`, `accessibilityRole`

```javascript
import { TextInput } from "react-native";
import React, { useState } from "react";

export default function App() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  return (
    <>
      <TextInput
        value={email}
        placeholder="이메일 입력"
        placeholderTextColor="#9ca3af"
        autoCapitalize="none"
        autoCorrect={false}
        keyboardType="email-address"
        returnKeyType="next"
        textContentType="emailAddress"
        onChangeText={setEmail}
        style={{
          padding: 12,
          borderWidth: 1,
          borderRadius: 8,
          fontSize: 16,
          marginBottom: 10,
        }}
        maxLength={50}
        onFocus={() => console.log("email focus")}
        onBlur={() => console.log("email blur")}
        testID="email-input"
        accessibilityLabel="이메일 입력창"
      />
      <TextInput
        value={password}
        placeholder="비밀번호 입력"
        placeholderTextColor="#9ca3af"
        secureTextEntry
        autoCapitalize="none"
        autoCorrect={false}
        keyboardType="default"
        returnKeyType="done"
        textContentType="password"
        onChangeText={setPassword}
        onSubmitEditing={() => console.log("비밀번호 제출")}
        style={{
          padding: 12,
          borderWidth: 1,
          borderRadius: 8,
          fontSize: 16,
        }}
        maxLength={20}
        testID="password-input"
        accessibilityLabel="비밀번호 입력창"
      />
    </>
  );
}
```

#### 5. Pressable
`TouchableOpacity`/`TouchableHighlight`의 최신 대체 컴포넌트입니다. “Press 상태(`nowPressing`)”를 직접 감지하여 상태 기반 스타일링이 가능합니다.

*   **언제 쓰는가?**
    *   버튼 UI를 직접 만들 때
    *   Press 상태에 따라 스타일 변경할 때
    *   LongPress, hover, focus 등 고급 이벤트 필요할 때
    *   텍스트/이미지/아이콘 어떤 것도 버튼화할 때

*   **주요 속성 / 타입 / 설명**
    *   `onPress`: `() => void`
    *   `onPressIn / onPressOut`: 누르는 순간, 뗄 때
    *   `onLongPress`: 길게 누를 때
    *   `disabled`: `boolean`
    *   `hitSlop`: `number | {top,left,bottom,right}` - 터치 영역 확장
    *   `android_ripple`: `{color, radius, borderless}` - Android 클릭 효과 설정
    *   `style`: `(state: {pressed: boolean, hovered: boolean, focused: boolean}) => ViewStyle` - Pressable만의 핵심 기능 → 상태 기반 스타일링
    *   `testID`, `accessibility` 전부 지원

```javascript
import { Pressable, Text, StyleSheet } from "react-native";
import React from "react";

export default function App() {
  return (
    <Pressable
      onPress={() => console.log("눌림")}
      style={({ pressed }) => ([
        styles.button,
        pressed && styles.buttonPressed,
      ])}
    >
      <Text style={styles.buttonText}>Pressable 버튼</Text>
    </Pressable>
  );
}

const styles = StyleSheet.create({
  button: {
    padding: 16,
    backgroundColor: "#fff",
    borderRadius: 8,
    alignItems: "center",
    justifyContent: "center",
    elevation: 2, // Android shadow
    shadowColor: '#000', // iOS shadow
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.2,
    shadowRadius: 1.41,
  },
  buttonPressed: {
    backgroundColor: "#ddd",
  },
  buttonText: {
    fontSize: 16,
    fontWeight: "bold",
    color: "#333",
  },
});
```

#### 6. ScrollView
화면을 스크롤 가능한 컨테이너로 만드는 컴포넌트입니다.

*   **언제 쓰는가?**
    *   소량의 데이터 스크롤
    *   화면 몇 개 정도의 영역만 스크롤할 때
    *   `FlatList`로 갈 만큼 데이터가 많지 않을 때
    *   (대량 데이터는 `FlatList` 사용 권장)

*   **주요 속성 / 타입 / 설명**
    *   `horizontal`: `boolean` - 가로 스크롤 여부
    *   `contentContainerStyle`: `ViewStyle` - 내부 컨텐츠 스타일
    *   `showsVerticalScrollIndicator`: `boolean` - 세로 스크롤바 표시 여부
    *   `showsHorizontalScrollIndicator`: `boolean` - 가로 스크롤바 표시 여부
    *   `bounces`: `boolean` (iOS) - 스크롤 탄성 여부
    *   `pagingEnabled`: `boolean` - 페이지 단위 스크롤 (Carousel 느낌)
    *   `refreshControl`: `<RefreshControl />` - 당겨서 새로고침 컴포넌트
    *   `onScroll`: `(event) => void` - 스크롤 변화 이벤트
    *   `scrollEventThrottle`: `number` - 스크롤 이벤트 발생 빈도
    *   `nestedScrollEnabled`: `boolean` (Android) - 중첩 스크롤 활성화 여부
    *   `maximumZoomScale / minimumZoomScale`: `number` (iOS) - 줌 기능 지원

```javascript
import { ScrollView, Text, View } from "react-native";
import React from "react";

export default function App() {
  return (
    <ScrollView
      style={{ flex: 1 }}
      contentContainerStyle={{ padding: 20 }}
      showsVerticalScrollIndicator={false}
    >
      <Text style={{ fontSize: 24, marginBottom: 10 }}>스크롤 가능 영역</Text>
      {Array.from({ length: 20 }).map((_, i) => (
        <View key={i} style={{ padding: 15, borderBottomWidth: 1, borderBottomColor: '#eee' }}>
          <Text>아이템 {i + 1}</Text>
        </View>
      ))}
    </ScrollView>
  );
}
```

#### 7. StyleSheet
React Native 스타일 객체 생성기입니다. 성능 최적화 및 JS 객체를 스타일처럼 정리할 수 있도록 돕습니다.

*   **언제 쓰는가?**
    *   스타일을 객체로 분리하고 재사용할 때
    *   Inline 스타일 남발을 피하고 구조화할 때
    *   전역 스타일 관리할 때

*   **주요 API**
    *   `StyleSheet.create(styles)`
        *   스타일 객체를 정적으로 만들고 성능 최적화
    *   `StyleSheet.flatten(style)`
        *   배열 스타일을 하나의 객체로 병합 (`[style1, style2, style3] -> { ...merged }`)
        *   주로 여러 개의 동적/조건부 스타일을 합칠 때 유용
    *   `StyleSheet.compose(style1, style2)`
        *   두 스타일을 합쳐서 하나의 스타일 객체로 만듦 (`style1, style2 -> { ...merged }`)
        *   두 스타일만 병합할 때 `flatten`보다 미세하게 더 가벼움

*   **`StyleSheet.flatten` 예제**
```javascript
import { StyleSheet, View, Text } from "react-native";
import React from "react";

const styles = StyleSheet.create({
  base: {
    padding: 10,
    backgroundColor: "white",
  },
  rounded: {
    borderRadius: 8,
  },
});

const isActive = true;

// 배열 스타일을 flatten으로 병합
const mergedStyle1 = StyleSheet.flatten([
  styles.base,
  styles.rounded,
  { backgroundColor: "skyblue" }
]);

console.log("Merged Style 1:", mergedStyle1);
/*
출력:
{
  padding: 10,
  backgroundColor: "skyblue",
  borderRadius: 8
}
*/

// 조건부 스타일을 flatten으로 병합
const mergedStyle2 = StyleSheet.flatten([
  styles.base,
  isActive && { opacity: 1 },
  !isActive && { opacity: 0.5 },
]);

console.log("Merged Style 2:", mergedStyle2);

export default function App() {
  return (
    <View style={mergedStyle1}>
      <Text>Flatten 예제</Text>
    </View>
  );
}
```

*   **`StyleSheet.compose` 예제**
```javascript
import { StyleSheet, View, Text } from "react-native";
import React from "react";

const styles = StyleSheet.create({
  base: {
    padding: 10,
    backgroundColor: "white",
  },
  primary: {
    backgroundColor: "blue",
    color: "white", // primary에 추가된 색상
  },
});

// compose로 두 스타일을 병합
const composedStyle = StyleSheet.compose(styles.base, styles.primary);

console.log("Composed Style:", composedStyle);
/*
결과:
{
  padding: 10,
  backgroundColor: "blue",
  color: "white"
}
*/

// inline 스타일과 compose
const finalStyle = StyleSheet.compose(
  composedStyle,
  { borderWidth: 2, borderColor: "green" }
);

export default function App() {
  return (
    <View style={finalStyle}>
      <Text style={{color: finalStyle.color}}>Compose 예제</Text>
    </View>
  );
}
```

### 📱 User Interface

#### 1. Button
간단한 버튼 UI를 제공하는 컴포넌트입니다.

*   **주요 속성 / 타입 / 설명**
    *   `onPress`: `({nativeEvent: PressEvent}) => void` (필수)
    *   `title`: `string` (필수) - 버튼의 안쪽 텍스트. 안드로이드는 대문자로 변환해서 표현.
    *   `accessibilityLabel`: `string` - 스크린리더 같은 보조기술이 버튼을 읽어 줄 때 사용하는 '읽기 전용 이름'.
    *   `accessibilityLanguage`: `string` - 해당 컴포넌트를 어떤 언어로 읽어야 하는지 스크린 리더에게 알려주는 속성.
    *   `accessibilityActions`: `array` - 스크린 리더 사용자를 위해 제공하는 추가 행동 목록을 정의하는 속성.
    *   `onAccessibilityAction`: `function` - `accessibilityActions`가 실행됐을 때 호출되는 핸들러.
    *   `color`: `string` - 버튼 색상.
    *   `disabled`: `boolean` - 해당 컴포넌트 비활성화 여부.
    *   `hasTVPreferredFocus`: `boolean` - TV 앱에서 화면이 켜지면 자동으로 이 버튼이 선택된 상태가 됨.
    *   `nextFocusDown`, `nextFocusUp`, `nextFocusLeft`, `nextFocusRight`, `nextFocusForward`: `number` - TV용 UI 버튼 배치가 복잡할 때 포커스가 엉뚱한 곳으로 튀지 않게 포커스 이동 경로를 고정.
    *   `testID`: `string` - UI 테스트에서 '이 버튼이 맞는지' 구분하는 식별자.
    *   `touchSoundDisabled`: `boolean` - Android에서 터치할 때 나는 ‘띡’ 소리를 끄는 옵션.

```javascript
import { Button, Alert, SafeAreaView, View, StyleSheet } from "react-native";
import { SafeAreaProvider } from "react-native-safe-area-context";
import React, { useState } from "react";

const loginBtnId = 1; // 가상 ID
const headerBtnId = 2; // 가상 ID
const menuBtnId = 3; // 가상 ID
const settingsBtnId = 4; // 가상 ID

export default function App() {
  const [count, setCount] = useState(0);

  return (
    <SafeAreaProvider>
      <SafeAreaView style={styles.container}>
        <View>
          <Button
            testID="submit-button"
            touchSoundDisabled={true}
            title={`Press me (${count})`}
            color="#f194ff"
            nextFocusDown={loginBtnId}
            nextFocusUp={headerBtnId}
            nextFocusLeft={menuBtnId}
            nextFocusRight={settingsBtnId}
            hasTVPreferredFocus={true}
            onPress={() => Alert.alert('Button with adjusted color pressed')}
            accessibilityLabel="어떠한 데이터를 전달하는 버튼입니다."
            accessibilityLanguage="ko"
            accessibilityActions={[
              { name: "increment", label: "증가" },
              { name: "decrement", label: "감소" }
            ]}
            onAccessibilityAction={(event) => {
              switch (event.nativeEvent.actionName) {
                case "increment":
                  setCount((c) => c + 1);
                  break;
                case "decrement":
                  setCount((c) => c - 1);
                  break;
              }
            }}
          />
        </View>
      </SafeAreaView>
    </SafeAreaProvider>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});
```

#### 2. Switch
[작성 필요]

### 📃 List Views

#### 1. FlatList
대량 데이터를 렌더링할 때 스크롤 퍼포먼스를 위해 가상화(`Virtualized`)된 리스트를 제공합니다. 필요한 아이템만 렌더링하여 성능을 최적화합니다.

*   **언제 쓰는가?**
    *   데이터가 많고 스크롤이 필요한 리스트일 때
    *   스크롤 퍼포먼스가 중요할 때
    *   단순 배열 데이터를 표시할 때 (섹션 단위가 필요하면 `SectionList`)
    *   **`ScrollView`와 비교:** `ScrollView`는 모든 아이템을 한 번에 렌더링하여 대량 데이터에서 성능 저하를 일으킬 수 있는 반면, `FlatList`는 필요한 아이템만 렌더링하여 대량 데이터(1000개 이상)에서도 뛰어난 성능을 보입니다.

*   **주요 속성 / 타입 / 설명**

| 속성 | 타입 | 설명 |
| :-------------------------- | :------------------------------------------- | :------------------------------------------------------ |
| `data` | `Array<T>` | 렌더링할 데이터 배열 (필수) |
| `renderItem` | `({item: T, index: number, separators}) => ReactNode` | 각 아이템을 어떻게 렌더링할지 정의 (필수) |
| `keyExtractor` | `(item, index) => string` | 리스트 아이템을 고유하게 식별할 key 정의 (성능 개선에 중요) |
| `ListHeaderComponent` | `ReactNode` | 리스트 맨 위에 고정된 컴포넌트 |
| `ListFooterComponent` | `ReactNode` | 리스트 맨 아래에 고정된 컴포넌트 |
| `ItemSeparatorComponent` | `ReactNode` | 각 아이템 사이에 들어갈 구분선 |
| `horizontal` | `boolean` | 가로 스크롤 여부 |
| `numColumns` | `number` | 그리드 형태로 보여줄 때 column 개수 |
| `onEndReached` | `() => void` | 스크롤 끝(하단) 도달 시 호출되는 함수 (페이징 처리) |
| `onEndReachedThreshold` | `number` | 하단 도달 임계값 (0~1 범위) |
| `refreshing` | `boolean` | Pull-to-refresh 상태 |
| `onRefresh` | `() => void` | 당겨서 새로고침 동작 |
| `showsVerticalScrollIndicator` | `boolean` | 세로 스크롤바 표시 여부 |
| `showsHorizontalScrollIndicator` | `boolean` | 가로 스크롤바 표시 여부 |
| `initialNumToRender` | `number` | 처음 렌더링될 아이템 개수 (성능 최적화에 매우 중요) |
| `maxToRenderPerBatch` | `number` | 한 번에 최대 렌더링할 아이템 수 |
| `windowSize` | `number` | 스크롤 윈도우 범위 (가상화 경계) |
| `removeClippedSubviews` | `boolean` | 화면 밖의 컴포넌트를 제거하여 성능 향상 (Android 강력 추천) |
| `getItemLayout` | `function` | 고정된 item height가 있을 때 스크롤 최적화 (스크롤 점프 성능↑) |
| `inverted` | `boolean` | 리스트 반전 (채팅 UI에서 사용) |
| `contentContainerStyle` | `Style` | 내부 컨테이너 스타일 (padding 등) |
| `style` | `Style` | 외부 스타일 |
| `ListEmptyComponent` | `ReactNode` | `data`가 빈 배열일 때 보여줄 컴포넌트 |
| `pagingEnabled` | `boolean` | 페이지 단위 스크롤 (카루셀처럼) |
| `scrollEventThrottle` | `number` | 스크롤 이벤트 발생 빈도 (애니메이션, 오프셋 계산에 중요) |
| `testID` | `string` | 테스트 자동화에서 해당 리스트를 찾기 위한 ID |
| `accessibilityLabel` | `string` | 스크린리더가 읽는 리스트의 설명 |
| `hasTVPreferredFocus` | `boolean` | TV UI에서 리스트 기본 포커스 |
| `nextFocusDown / Up / Left / Right` | `number` | TV 포커스 이동 경로 제어 |

```javascript
import { FlatList, Text, View, SafeAreaView, StyleSheet } from "react-native";
import React from "react";

const DATA = [
  { id: "1", title: "첫 번째 아이템" },
  { id: "2", title: "두 번째 아이템" },
  { id: "3", title: "세 번째 아이템" },
  { id: "4", title: "네 번째 아이템" },
  { id: "5", title: "다섯 번째 아이템" },
  { id: "6", title: "여섯 번째 아이템" },
  { id: "7", title: "일곱 번째 아이템" },
];

export default function App() {
  return (
    <SafeAreaView style={styles.container}>
      <FlatList
        testID="main-list"
        data={DATA}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <View style={styles.item}>
            <Text>{item.title}</Text>
          </View>
        )}
        ListHeaderComponent={
          <Text style={styles.listHeader}>리스트 헤더</Text>
        }
        ListFooterComponent={
          <Text style={styles.listFooter}>리스트 푸터</Text>
        }
        ItemSeparatorComponent={() => (
          <View style={styles.separator} />
        )}
        horizontal={false}
        showsVerticalScrollIndicator={false}
        initialNumToRender={5}
        maxToRenderPerBatch={10}
        windowSize={21}
        removeClippedSubviews={true}
        onEndReached={() => console.log("리스트 끝 도달")}
        onEndReachedThreshold={0.1}
        ListEmptyComponent={<Text style={styles.emptyList}>데이터 없음</Text>}
        refreshing={false}
        onRefresh={() => console.log("새로고침")}
        accessibilityLabel="게시글 목록입니다"
        hasTVPreferredFocus={false}
        contentContainerStyle={styles.contentContainer}
      />
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  item: {
    padding: 20,
    borderBottomWidth: 1,
    borderBottomColor: '#f0f0f0',
  },
  listHeader: {
    padding: 10,
    fontSize: 18,
    fontWeight: 'bold',
    backgroundColor: '#e0e0e0',
  },
  listFooter: {
    padding: 10,
    fontSize: 14,
    color: '#666',
    backgroundColor: '#e0e0e0',
  },
  separator: {
    height: 1,
    backgroundColor: "#ccc",
  },
  emptyList: {
    padding: 20,
    textAlign: 'center',
    color: '#999',
  },
  contentContainer: {
    paddingBottom: 100,
  },
});
```

#### 2. SectionList
섹션 단위로 데이터를 표시하는 리스트 컴포넌트입니다. 내부적으로 `FlatList` 기반이지만, 섹션 헤더/푸터 및 섹션별 데이터 관리 기능이 추가됩니다.

*   **언제 쓰는가?**
    *   제목(섹션) 아래 데이터 목록이 있는 경우 (예: 날짜별 메시지, 카테고리별 리스트, 알파벳별 연락처 목록, 월별 스케줄)
    *   그룹화된 리스트를 가상화 성능 그대로 쓰고 싶을 때
    *   `ScrollView + map`으로 그룹을 구현하는 것보다 훨씬 효율적입니다.

*   **주요 속성 / 타입 / 설명**

| 속성 | 타입 | 설명 |
| :-------------------------- | :------------------------------------ | :----------------------------------------------------------------------------- |
| `sections` | `Array<{title: string, data: Array<any>}>` | 섹션 단위로 데이터를 묶음 (필수) |
| `renderItem` | `({item, index, section}) => ReactNode` | 섹션 내 아이템을 렌더링 (필수) |
| `renderSectionHeader` | `({section}) => ReactNode` | 섹션 헤더(예: “11월 업데이트”) 렌더링 |
| `renderSectionFooter` | `({section}) => ReactNode` | 섹션 푸터 렌더링 |
| `keyExtractor` | `(item, index) => string` | 아이템 고유 key 생성 |
| `ListHeaderComponent` | `ReactNode` | 리스트 최상단 헤더 |
| `ListFooterComponent` | `ReactNode` | 리스트 최하단 푸터 |
| `SectionSeparatorComponent` | `ReactNode` | 섹션 사이 구분선 |
| `ItemSeparatorComponent` | `ReactNode` | 아이템 사이 구분선 |
| `stickySectionHeadersEnabled` | `boolean` | 스크롤할 때 섹션 헤더 고정 (기본 `true`) |
| `horizontal` | `boolean` | 가로 스크롤 여부 |
| `onEndReached` | `() => void` | 스크롤 끝 도달 시 (페이징 용도) |
| `onEndReachedThreshold` | `number` | 하단 임계값 (0~1) |
| `refreshing` | `boolean` | pull-to-refresh 상태 |
| `onRefresh` | `() => void` | 당겨서 새로고침 |
| `initialNumToRender` | `number` | 초기에 렌더할 아이템 수 |
| `windowSize` | `number` | 가상화 윈도우 범위 |
| `removeClippedSubviews` | `boolean` | 화면 밖 아이템 제거 (Android 성능 증가) |
| `getItemLayout` | `function` | 고정된 높이일 경우 스크롤 최적화 |
| `inverted` | `boolean` | 섹션 포함 전체 리스트 반전 (채팅 UI에서 사용) |
| `contentContainerStyle` | `Style` | 전체 컨텐츠 padding 등 지정 |
| `style` | `Style` | 컨테이너 스타일 |
| `ListEmptyComponent` | `ReactNode` | `sections`가 빈 배열일 때 렌더링되는 컴포넌트 |
| `pagingEnabled` | `boolean` | 페이지 단위 스크롤 |
| `scrollEventThrottle` | `number` | 스크롤 이벤트 호출 빈도 |
| `testID` | `string` | 테스트 자동화용 ID |
| `accessibilityLabel` | `string` | 스크린리더용 설명 |
| `hasTVPreferredFocus` | `boolean` | TV에서 첫 포커스 받을지 |
| `nextFocusDown / Up / Left / Right` | `number` | TV 리모컨 포커스 이동 경로 |

```javascript
import { SectionList, Text, View, SafeAreaView, StyleSheet } from "react-native";
import React from "react";

const SECTIONS = [
  {
    title: "2025년 11월",
    data: ["아이템 1", "아이템 2", "아이템 3"],
  },
  {
    title: "2025년 12월",
    data: ["아이템 A", "아이템 B"],
  },
  {
    title: "2026년 01월",
    data: ["새해 계획", "새로운 목표"],
  },
];

export default function App() {
  return (
    <SafeAreaView style={styles.container}>
      <SectionList
        testID="main-section-list"
        sections={SECTIONS}
        keyExtractor={(item, index) => item + index}
        renderItem={({ item }) => (
          <View style={styles.item}>
            <Text>{item}</Text>
          </View>
        )}
        renderSectionHeader={({ section }) => (
          <View style={styles.sectionHeader}>
            <Text style={styles.sectionTitle}>{section.title}</Text>
          </View>
        )}
        renderSectionFooter={({ section }) => (
          <View style={styles.sectionFooter}>
            <Text style={styles.sectionFooterText}>
              {section.data.length}개의 아이템
            </Text>
          </View>
        )}
        ItemSeparatorComponent={() => (
          <View style={styles.itemSeparator} />
        )}
        SectionSeparatorComponent={() => (
          <View style={styles.sectionSeparator} />
        )}
        stickySectionHeadersEnabled={true}
        onEndReached={() => console.log("섹션 리스트 끝")}
        onEndReachedThreshold={0.2}
        refreshing={false}
        onRefresh={() => console.log("새로고침")}
        initialNumToRender={5}
        windowSize={21}
        removeClippedSubviews={true}
        accessibilityLabel="섹션 리스트 데이터입니다"
        hasTVPreferredFocus={false}
        contentContainerStyle={styles.contentContainer}
      />
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  item: {
    padding: 15,
    backgroundColor: '#fff',
  },
  sectionHeader: {
    backgroundColor: "#eee",
    padding: 10,
  },
  sectionTitle: {
    fontWeight: "bold",
    fontSize: 16,
  },
  sectionFooter: {
    padding: 10,
    backgroundColor: '#fff',
  },
  sectionFooterText: {
    color: "#666",
    fontSize: 12,
  },
  itemSeparator: {
    height: 1,
    backgroundColor: "#ddd",
  },
  sectionSeparator: {
    height: 10,
    backgroundColor: "#fafafa",
  },
  contentContainer: {
    paddingBottom: 50,
  },
});
```

### 🤖 Android Components and APIs

#### 1. BackHandler
안드로이드의 뒤로가기 버튼 이벤트를 감지하고 제어하는 API입니다.

*   **언제 쓰는가?**
    *   뒤로가기 버튼 눌렀을 때 특정 페이지에서 앱 종료 막기
    *   Drawer/Modal 닫기
    *   WebView 뒤로가기 처리
    *   커스텀 네비게이션 처리

*   **주요 메서드**
    *   `BackHandler.addEventListener("hardwareBackPress", handler)`
    *   `BackHandler.removeEventListener("hardwareBackPress", handler)`
    *   `BackHandler.exitApp()`
    *   핸들러는 `true`를 반환하면 기본 뒤로가기 동작을 막습니다.

```javascript
import { BackHandler, Text, View } from "react-native";
import React, { useEffect } from "react";

export default function App() {
  useEffect(() => {
    const backHandler = BackHandler.addEventListener(
      "hardwareBackPress",
      () => {
        console.log("뒤로가기 버튼 눌림. 기본 동작을 막음.");
        // 여기서 원하는 동작 수행 (예: 모달 닫기, 앱 종료 방지)
        return true; // true를 반환하여 기본 뒤로가기 동작을 막음
      }
    );

    return () => backHandler.remove(); // 컴포넌트 언마운트 시 이벤트 리스너 제거
  }, []);

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text>Android 뒤로가기 버튼을 눌러보세요.</Text>
    </View>
  );
}
```

#### 2. DrawerLayoutAndroid
Android 용 Drawer UI (우측/좌측 슬라이드 메뉴)를 제공합니다.

*   **언제 쓰는가?**
    *   Navigation Drawer를 직접 만들 때
    *   React Navigation 라이브러리 없이 Drawer만 필요할 때

*   **주요 props**
    *   `drawerWidth`: `number`
    *   `drawerPosition`: `"left" | "right"`
    *   `renderNavigationView`: `() => ReactNode`
    *   `onDrawerOpen`: `() => void`
    *   `onDrawerClose`: `() => void`
    *   `keyboardDismissMode`: `"none" | "on-drag"`

```javascript
import { DrawerLayoutAndroid, Text, View, StyleSheet } from "react-native";
import React, { useRef } from "react";
import { Button } from "react-native";

export default function App() {
  const drawer = useRef(null);
  const navigationView = () => (
    <View style={styles.navigationContainer}>
      <Text style={styles.paragraph}>I'm in the Drawer!</Text>
      <Button
        title="Close drawer"
        onPress={() => drawer.current.closeDrawer()}
      />
    </View>
  );

  return (
    <DrawerLayoutAndroid
      ref={drawer}
      drawerWidth={300}
      drawerPosition="left"
      renderNavigationView={navigationView}
      onDrawerOpen={() => console.log("Drawer opened")}
      onDrawerClose={() => console.log("Drawer closed")}
    >
      <View style={styles.container}>
        <Text style={styles.paragraph}>
          DrawerLayoutAndroid example. Swipe right or press the button.
        </Text>
        <Button
          title="Open drawer"
          onPress={() => drawer.current.openDrawer()}
        />
      </View>
    </DrawerLayoutAndroid>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center",
    padding: 16,
  },
  navigationContainer: {
    flex: 1,
    paddingTop: 50,
    backgroundColor: "#ecf0f1",
    padding: 20,
  },
  paragraph: {
    padding: 16,
    fontSize: 15,
    textAlign: "center",
  },
});
```

#### 3. PermissionsAndroid
Android 권한 요청 (카메라, 마이크 등)을 처리하는 API입니다.

*   **언제 쓰는가?**
    *   카메라 접근
    *   저장소 접근
    *   위치 권한
    *   오디오 레코딩 등 필수 권한 요청

*   **주요 메서드**
    *   `PermissionsAndroid.request(permission: string)`: 단일 권한 요청
    *   `PermissionsAndroid.check(permission: string)`: 권한 확인
    *   `PermissionsAndroid.requestMultiple(permissions: string[])`: 여러 권한 요청

```javascript
import { PermissionsAndroid, Button, Alert, View, Text } from "react-native";
import React from "react";

export default function App() {
  const requestCameraPermission = async () => {
    try {
      const granted = await PermissionsAndroid.request(
        PermissionsAndroid.PERMISSIONS.CAMERA,
        {
          title: "카메라 권한",
          message: "앱이 카메라를 사용하려면 권한이 필요합니다.",
          buttonNeutral: "나중에",
          buttonNegative: "취소",
          buttonPositive: "확인",
        },
      );
      if (granted === PermissionsAndroid.RESULTS.GRANTED) {
        Alert.alert("카메라 권한 허용", "카메라를 사용할 수 있습니다.");
        console.log("카메라 허용됨");
      } else {
        Alert.alert("카메라 권한 거부", "카메라를 사용할 수 없습니다.");
        console.log("카메라 거부됨");
      }
    } catch (err) {
      console.warn(err);
    }
  };

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text style={{ marginBottom: 20 }}>카메라 권한 요청 예제</Text>
      <Button title="카메라 권한 요청" onPress={requestCameraPermission} />
    </View>
  );
}
```

#### 4. ToastAndroid
Android 전용 Toast 메시지를 보여주는 API입니다.

*   **주요 메서드**
    *   `ToastAndroid.show(message: string, duration: number)`: 짧거나 긴 토스트 메시지 표시
    *   `ToastAndroid.showWithGravity(message: string, duration: number, gravity: number)`: 위치 지정 토스트 메시지
    *   `ToastAndroid.SHORT`, `ToastAndroid.LONG`: `duration` 상수
    *   `ToastAndroid.TOP`, `ToastAndroid.CENTER`, `ToastAndroid.BOTTOM`: `gravity` 상수

```javascript
import { ToastAndroid, Button, View } from "react-native";
import React from "react";

export default function App() {
  const showToast = () => {
    ToastAndroid.show("저장되었습니다!", ToastAndroid.SHORT);
  };

  const showToastWithGravity = () => {
    ToastAndroid.showWithGravity(
      "아래 중앙에 표시!",
      ToastAndroid.SHORT,
      ToastAndroid.BOTTOM
    );
  };

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Button title="짧은 토스트 메시지 보기" onPress={showToast} />
      <View style={{ marginVertical: 10 }} />
      <Button title="위치 지정 토스트 메시지 보기" onPress={showToastWithGravity} />
    </View>
  );
}
```

### 🍎 iOS Components and APIs

#### 1. ActionSheetIOS
iOS 기본 액션시트 UI를 제공합니다.

*   **언제 쓰는가?**
    *   사용자에게 여러 선택지(선택/삭제/취소 등)를 제공할 때
    *   특정 작업을 수행하기 전 확인을 요구할 때

*   **주요 메서드**
    *   `ActionSheetIOS.showActionSheetWithOptions(options: object, callback: (buttonIndex: number) => void)`
    *   `ActionSheetIOS.showShareActionSheetWithOptions(options: object, failureCallback: (error: Error) => void, successCallback: (completed: boolean, activityType: string) => void)`

```javascript
import { ActionSheetIOS, Button, View, Text, Platform } from "react-native";
import React from "react";

export default function App() {
  const showActionSheet = () => {
    if (Platform.OS === 'ios') {
      ActionSheetIOS.showActionSheetWithOptions(
        {
          options: ["취소", "사진 촬영", "갤러리에서 선택", "삭제"],
          destructiveButtonIndex: 3, // 삭제 버튼을 빨간색으로 표시
          cancelButtonIndex: 0, // 취소 버튼 인덱스
          title: "이미지 선택",
          message: "사진을 촬영하거나 갤러리에서 선택하세요.",
        },
        (buttonIndex) => {
          if (buttonIndex === 0) {
            console.log("취소됨");
          } else if (buttonIndex === 1) {
            console.log("사진 촬영 선택");
          } else if (buttonIndex === 2) {
            console.log("갤러리에서 선택");
          } else if (buttonIndex === 3) {
            console.log("삭제 선택");
          }
        }
      );
    } else {
      alert("이 기능은 iOS에서만 지원됩니다.");
    }
  };

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text style={{ marginBottom: 20 }}>iOS ActionSheet 예제</Text>
      <Button title="액션시트 열기" onPress={showActionSheet} />
    </View>
  );
}
```

### 🌐 Others (Cross-platform APIs)

#### 1. ActivityIndicator
원형의 로딩 인디케이터를 표시하는 컴포넌트입니다.

*   **주요 속성 / 타입 / 설명**
    *   `animating`: `boolean` - 인디케이터를 보여줄지 여부
    *   `size`: `'small' | 'large' | number` - 인디케이터의 사이즈 (`small`/`large` 또는 숫자 값)
    *   `color`: `string` - 인디케이터의 색상 (기본값: Android - `null` / iOS - '#999999')

```javascript
import { ActivityIndicator, SafeAreaView, StyleSheet, Text, View } from "react-native";
import { SafeAreaProvider } from "react-native-safe-area-context";
import React from "react";

export default function App() {
  return (
    <SafeAreaProvider>
      <SafeAreaView style={styles.container}>
        <Text style={styles.title}>ActivityIndicator 예제</Text>
        <View style={styles.indicatorsContainer}>
          <ActivityIndicator animating={true} />
          <ActivityIndicator size="large" />
          <ActivityIndicator size="small" color="#0000ff" />
          <ActivityIndicator size="large" color="#00ff00" />
          <ActivityIndicator animating={false} color="#ff0000" />
        </View>
      </SafeAreaView>
    </SafeAreaProvider>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  title: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 30,
  },
  indicatorsContainer: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    width: '80%',
  }
});
```

#### 2. Alert
네이티브 팝업(Alert)을 띄우는 API입니다 (iOS/Android).

*   **주요 메서드**
    *   `Alert.alert(title: string, message?: string, buttons?: AlertButton[], options?: AlertOptions)`

```javascript
import { Alert, Button, View } from "react-native";
import React from "react";

export default function App() {
  const showAlert = () => {
    Alert.alert(
      "경고",
      "삭제하시겠습니까? 이 작업은 되돌릴 수 없습니다.",
      [
        { text: "취소", style: "cancel", onPress: () => console.log("취소됨") },
        { text: "삭제", style: "destructive", onPress: () => console.log("삭제됨") },
      ],
      { cancelable: true } // 외부 탭 시 닫힘 (Android)
    );
  };

  const showSimpleAlert = () => {
    Alert.alert("알림", "간단한 알림 메시지입니다.");
  };

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Button title="경고 팝업 보기" onPress={showAlert} />
      <View style={{ marginVertical: 10 }} />
      <Button title="간단 알림 팝업 보기" onPress={showSimpleAlert} />
    </View>
  );
}
```

#### 3. Animated
React Native의 네이티브 애니메이션 시스템입니다.

*   **언제 쓰는가?**
    *   Fade in/out, Slide 애니메이션
    *   스크롤 연동 애니메이션
    *   Interpolation 기반 애니메이션 (값 범위 변환)

*   **주요 API**
    *   `Animated.Value`: 애니메이션의 현재 값을 저장
    *   `Animated.timing`: 시간 기반 애니메이션
    *   `Animated.spring`: 용수철(스프링) 물리 기반 애니메이션
    *   `Animated.sequence`: 애니메이션을 순차적으로 실행
    *   `Animated.parallel`: 애니메이션을 동시에 실행
    *   `Animated.stagger`: 애니메이션을 번갈아가며 지연 실행

```javascript
import { Animated, Button, StyleSheet, View, Text } from "react-native";
import React, { useRef } from "react";

export default function App() {
  const opacity = useRef(new Animated.Value(0)).current; // 초기 투명도 0
  const translateY = useRef(new Animated.Value(50)).current; // 초기 y 위치 50

  const fadeInAndSlideUp = () => {
    Animated.parallel([
      Animated.timing(opacity, {
        toValue: 1,
        duration: 800,
        useNativeDriver: true, // 네이티브 스레드에서 실행
      }),
      Animated.timing(translateY, {
        toValue: 0,
        duration: 800,
        useNativeDriver: true,
      }),
    ]).start();
  };

  const fadeOutAndSlideDown = () => {
    Animated.parallel([
      Animated.timing(opacity, {
        toValue: 0,
        duration: 500,
        useNativeDriver: true,
      }),
      Animated.timing(translateY, {
        toValue: 50,
        duration: 500,
        useNativeDriver: true,
      }),
    ]).start();
  };

  return (
    <View style={styles.container}>
      <Animated.View
        style={[
          styles.animatedBox,
          {
            opacity: opacity,
            transform: [{ translateY: translateY }],
          },
        ]}
      >
        <Text style={styles.animatedText}>애니메이션!</Text>
      </Animated.View>
      <View style={styles.buttonContainer}>
        <Button title="보이기" onPress={fadeInAndSlideUp} />
        <Button title="숨기기" onPress={fadeOutAndSlideDown} />
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center",
  },
  animatedBox: {
    width: 150,
    height: 150,
    backgroundColor: "#61dafb",
    borderRadius: 10,
    justifyContent: "center",
    alignItems: "center",
    marginBottom: 30,
  },
  animatedText: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#fff',
  },
  buttonContainer: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    width: '60%',
  }
});
```

#### 4. Dimensions
화면 크기(가로/세로)를 가져오는 API입니다. 반응형 UI를 구현할 때 유용합니다.

*   **주요 메서드**
    *   `Dimensions.get("window")`: 현재 보이는 창의 크기 (소프트 키보드 등이 가릴 수 있음)
    *   `Dimensions.get("screen")`: 디바이스 전체 화면의 크기

```javascript
import { Dimensions, Text, View, StyleSheet } from "react-native";
import React, { useEffect, useState } from "react";

export default function App() {
  const [dimensions, setDimensions] = useState(Dimensions.get("window"));

  useEffect(() => {
    const subscription = Dimensions.addEventListener("change", ({ window }) => {
      setDimensions(window);
    });
    return () => subscription?.remove();
  });

  const { width, height } = dimensions;

  return (
    <View style={styles.container}>
      <Text style={styles.text}>
        현재 화면 너비: {width.toFixed(0)} dp
      </Text>
      <Text style={styles.text}>
        현재 화면 높이: {height.toFixed(0)} dp
      </Text>
      <View style={[styles.box, { width: width * 0.5, height: height * 0.2 }]} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center",
  },
  text: {
    fontSize: 18,
    marginBottom: 10,
  },
  box: {
    backgroundColor: '#ffc107',
    marginTop: 20,
    justifyContent: 'center',
    alignItems: 'center',
    borderRadius: 8,
  }
});
```

#### 5. KeyboardAvoidingView
키보드가 올라올 때 Input이 가려지지 않게 자동으로 View를 올려주는 컴포넌트입니다.

*   **주요 props**
    *   `behavior`: `"padding" | "position" | "height"` - 키보드 회피 동작 방식
    *   `keyboardVerticalOffset`: `number` - 키보드와의 추가적인 수직 오프셋 (헤더 바 높이 등)

```javascript
import { KeyboardAvoidingView, TextInput, StyleSheet, View, Platform, Text } from "react-native";
import React from "react";

export default function App() {
  return (
    <KeyboardAvoidingView
      behavior={Platform.OS === "ios" ? "padding" : "height"}
      style={styles.container}
      keyboardVerticalOffset={Platform.OS === "ios" ? 60 : 0} // iOS 상태바/내비게이션바 높이 고려
    >
      <View style={styles.inner}>
        <Text style={styles.header}>키보드를 피해라!</Text>
        <TextInput placeholder="입력하세요" style={styles.textInput} />
        <View style={styles.btnContainer}>
          <Text>아래 버튼을 누르면 키보드가 올라옵니다.</Text>
        </View>
      </View>
    </KeyboardAvoidingView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  inner: {
    padding: 24,
    flex: 1,
    justifyContent: "space-around",
    alignItems: "center",
  },
  header: {
    fontSize: 24,
    marginBottom: 48,
  },
  textInput: {
    height: 40,
    borderColor: "#000000",
    borderBottomWidth: 1,
    width: "80%",
    paddingHorizontal: 8,
  },
  btnContainer: {
    backgroundColor: "white",
    marginTop: 12,
  },
});
```

#### 6. Linking
외부 링크, 앱 실행, 딥링크 제어를 위한 API입니다.

*   **주요 메서드**
    *   `Linking.openURL(url: string)`: 주어진 URL을 사용하여 외부 앱 열기
    *   `Linking.canOpenURL(url: string)`: URL을 열 수 있는지 확인 (앱 설치 여부 확인 등에 사용)
    *   `Linking.getInitialURL()`: 앱이 딥링크로 열렸을 경우 초기 URL 가져오기
    *   `Linking.addEventListener('url', callback)`: 딥링크 수신 시 이벤트 리스너

```javascript
import { Linking, Button, View, Text, Alert } from "react-native";
import React from "react";

export default function App() {
  const openExternalLink = async () => {
    const url = "https://google.com";
    const supported = await Linking.canOpenURL(url);

    if (supported) {
      await Linking.openURL(url);
    } else {
      Alert.alert(`이 URL을 열 수 없습니다: ${url}`);
    }
  };

  const openPhoneDialer = async () => {
    const phoneNumber = "tel:01012345678";
    const supported = await Linking.canOpenURL(phoneNumber);

    if (supported) {
      await Linking.openURL(phoneNumber);
    } else {
      Alert.alert(`전화 걸기 기능을 사용할 수 없습니다.`);
    }
  };

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text style={{ marginBottom: 20 }}>Linking API 예제</Text>
      <Button title="구글 웹사이트 열기" onPress={openExternalLink} />
      <View style={{ marginVertical: 10 }} />
      <Button title="전화 걸기 (010-1234-5678)" onPress={openPhoneDialer} />
    </View>
  );
}
```

#### 7. Modal
네이티브 모달 컴포넌트입니다.

*   **주요 props**
    *   `visible`: `boolean` - 모달 표시 여부
    *   `transparent`: `boolean` - 모달 배경을 투명하게 할지 여부
    *   `animationType`: `"none" | "slide" | "fade"` - 모달이 나타나고 사라질 때의 애니메이션
    *   `onRequestClose`: `function` - Android의 뒤로가기 버튼이나 모달 외부 터치 시 호출 (필수)
    *   `presentationStyle`: `'fullScreen' | 'pageSheet' | 'formSheet' | 'overFullScreen'` (iOS) - 모달의 프리젠테이션 스타일

```javascript
import { Modal, Text, View, Button, StyleSheet } from "react-native";
import React, { useState } from "react";

export default function App() {
  const [modalVisible, setModalVisible] = useState(false);

  return (
    <View style={styles.container}>
      <Text style={styles.title}>모달 컴포넌트 예제</Text>
      <Button title="모달 열기" onPress={() => setModalVisible(true)} />

      <Modal
        animationType="slide" // fade, slide, none
        transparent={true} // 모달 배경 투명하게
        visible={modalVisible}
        onRequestClose={() => {
          // Android 뒤로가기 버튼 처리
          setModalVisible(!modalVisible);
          console.log("Modal has been closed.");
        }}
        presentationStyle="pageSheet" // iOS 모달 스타일 (iOS 13+ 추천)
      >
        <View style={styles.centeredView}>
          <View style={styles.modalView}>
            <Text style={styles.modalText}>안녕하세요, 모달입니다!</Text>
            <Button
              title="모달 닫기"
              onPress={() => setModalVisible(!modalVisible)}
            />
          </View>
        </View>
      </Modal>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center",
  },
  title: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 30,
  },
  centeredView: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center",
    marginTop: 22,
    backgroundColor: 'rgba(0,0,0,0.5)', // 반투명 배경
  },
  modalView: {
    margin: 20,
    backgroundColor: "white",
    borderRadius: 20,
    padding: 35,
    alignItems: "center",
    shadowColor: "#000",
    shadowOffset: {
      width: 0,
      height: 2,
    },
    shadowOpacity: 0.25,
    shadowRadius: 4,
    elevation: 5,
  },
  modalText: {
    marginBottom: 15,
    textAlign: "center",
    fontSize: 18,
  },
});
```

#### 8. PixelRatio
픽셀 비율 계산 API입니다. dp ↔ px 변환에 사용됩니다.

*   **주요 메서드**
    *   `PixelRatio.get()`: 디바이스의 픽셀 비율 (예: iPhone X는 3)
    *   `PixelRatio.getFontScale()`: 디바이스의 폰트 스케일
    *   `PixelRatio.getPixelSizeForLayoutSize(layoutSize: number)`: DP 단위를 픽셀 단위로 변환
    *   `PixelRatio.roundToNearestPixel(layoutSize: number)`: 가장 가까운 픽셀 단위로 반올림

```javascript
import { PixelRatio, Text, View, StyleSheet } from "react-native";
import React from "react";

export default function App() {
  const pixelRatio = PixelRatio.get();
  const fontScale = PixelRatio.getFontScale();
  const dpSize = 100;
  const pixelSize = PixelRatio.getPixelSizeForLayoutSize(dpSize);
  const roundedDp = PixelRatio.roundToNearestPixel(dpSize);

  return (
    <View style={styles.container}>
      <Text style={styles.text}>디바이스 픽셀 비율: {pixelRatio}</Text>
      <Text style={styles.text}>폰트 스케일: {fontScale}</Text>
      <Text style={styles.text}>
        {dpSize} dp는 {pixelSize} px (정확한 픽셀 크기)
      </Text>
      <Text style={styles.text}>
        {dpSize} dp를 가장 가까운 픽셀로 반올림: {roundedDp} dp
      </Text>
      <View style={[styles.box, { width: dpSize, height: dpSize }]} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center",
  },
  text: {
    fontSize: 16,
    marginBottom: 10,
  },
  box: {
    backgroundColor: '#9c27b0',
    marginTop: 20,
    justifyContent: 'center',
    alignItems: 'center',
    borderRadius: 5,
  }
});
```

#### 9. RefreshControl
`ScrollView` / `FlatList`에서 당겨서 새로고침 UI를 제공하는 컴포넌트입니다.

*   **주요 props**
    *   `refreshing`: `boolean` - 현재 새로고침 중인지 여부
    *   `onRefresh`: `() => void` - 당겨서 새로고침 액션이 발생했을 때 호출될 함수
    *   `tintColor`: `string` (iOS) - 새로고침 인디케이터 색상
    *   `title`: `string` (iOS) - 새로고침 텍스트
    *   `titleColor`: `string` (iOS) - 새로고침 텍스트 색상
    *   `colors`: `string[]` (Android) - 새로고침 인디케이터 색상 배열
    *   `progressBackgroundColor`: `string` (Android) - 새로고침 배경 색상

```javascript
import { ScrollView, RefreshControl, Text, View, StyleSheet } from "react-native";
import React, { useState, useCallback } from "react";

export default function App() {
  const [refreshing, setRefreshing] = useState(false);
  const [data, setData] = useState(Array.from({ length: 10 }).map((_, i) => `초기 아이템 ${i + 1}`));

  const onRefresh = useCallback(() => {
    setRefreshing(true);
    setTimeout(() => {
      const newData = Array.from({ length: 10 }).map((_, i) => `새로고침 아이템 ${Math.random().toFixed(2)}`);
      setData(newData);
      setRefreshing(false);
    }, 2000);
  }, []);

  return (
    <ScrollView
      contentContainerStyle={styles.scrollView}
      refreshControl={
        <RefreshControl
          refreshing={refreshing}
          onRefresh={onRefresh}
          tintColor="#f8f8f8" // iOS
          title="새로고침 중..." // iOS
          titleColor="#000" // iOS
          colors={["#9Bd35A", "#689F38"]} // Android
          progressBackgroundColor="#ffffff" // Android
        />
      }
    >
      <Text style={styles.title}>당겨서 새로고침</Text>
      {data.map((item, index) => (
        <View key={index} style={styles.item}>
          <Text>{item}</Text>
        </View>
      ))}
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  scrollView: {
    flex: 1,
    backgroundColor: '#f0f0f0',
    alignItems: 'center',
    justifyContent: 'center',
  },
  title: {
    fontSize: 22,
    fontWeight: 'bold',
    marginBottom: 20,
  },
  item: {
    backgroundColor: '#fff',
    padding: 15,
    marginVertical: 5,
    borderRadius: 8,
    width: '80%',
    alignItems: 'center',
  }
});
```

#### 10. StatusBar
상태바(상단 바) 스타일을 제어하는 컴포넌트입니다.

*   **주요 props**
    *   `barStyle`: `'default' | 'light-content' | 'dark-content'` - 상태바 텍스트/아이콘 색상
    *   `backgroundColor`: `string` (Android) - 상태바 배경 색상
    *   `hidden`: `boolean` - 상태바 숨김 여부
    *   `translucent`: `boolean` (Android) - 상태바가 반투명하여 앱 콘텐츠가 상태바 뒤로 확장될지 여부
    *   `animated`: `boolean` - 속성 변경 시 애니메이션 효과 적용 여부

```javascript
import { StatusBar, Button, View, Text, StyleSheet, Platform } from "react-native";
import React, { useState } from "react";

export default function App() {
  const [barStyle, setBarStyle] = useState<'default' | 'light-content' | 'dark-content'>('dark-content');
  const [backgroundColor, setBackgroundColor] = useState('#ffffff');
  const [hidden, setHidden] = useState(false);

  const toggleBarStyle = () => {
    setBarStyle(prevStyle => 
      prevStyle === 'dark-content' ? 'light-content' : 'dark-content'
    );
    setBackgroundColor(prevColor => 
      prevColor === '#ffffff' ? '#333333' : '#ffffff'
    );
  };

  const toggleHidden = () => {
    setHidden(prevHidden => !prevHidden);
  };

  return (
    <View style={styles.container}>
      <StatusBar 
        barStyle={barStyle} 
        backgroundColor={backgroundColor} 
        hidden={hidden} 
        animated={true}
        translucent={Platform.OS === 'android' ? true : false} // Android에서만 적용
      />
      <Text style={styles.title}>StatusBar 제어 예제</Text>
      <Button title="스타일 변경" onPress={toggleBarStyle} />
      <View style={{ marginVertical: 10 }} />
      <Button title={hidden ? "상태바 보이기" : "상태바 숨기기"} onPress={toggleHidden} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center",
    // Android에서 translucent=true일 경우, 콘텐츠가 상태바 뒤로 들어가므로 padding 조정 필요
    paddingTop: Platform.OS === 'android' ? StatusBar.currentHeight : 0, 
  },
  title: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 30,
  }
});
```

---

## 💡 새롭게 알게 된 점
*   **ActivityIndicator `color` 속성:** `ActivityIndicator`의 `color` 속성은 iOS/Android 모두 동일하게 적용되는 공통 속성이며, Android 전용 `color` 속성은 따로 존재하지 않음을 확인했습니다. 플랫폼별로 색상을 다르게 하고 싶다면 `Platform.OS`를 통해 분기 처리해야 합니다.
*   **Button 접근성 및 TV 속성 심층 이해:**
    *   `accessibilityLabel`은 시각 장애인을 위한 '읽기 전용 이름'으로, 아이콘 버튼이나 추상적인 텍스트 버튼에 특히 유용함을 명확히 인지했습니다.
    *   `accessibilityLanguage`, `accessibilityActions`, `onAccessibilityAction`을 통해 스크린리더 사용자에게 더 풍부한 정보와 상호작용을 제공할 수 있음을 학습했습니다.
    *   `hasTVPreferredFocus`, `nextFocusDown/Up/Left/Right/Forward`는 TV 앱 환경에서 리모컨 포커스 이동을 제어하는 필수 속성으로, 복잡한 TV UI에서 포커스 흐름을 명확히 정의하는 데 중요함을 이해했습니다.
    *   `testID`는 자동화 테스트를 위한 식별자이며, `touchSoundDisabled`는 Android의 기본 터치 사운드를 비활성화하는 데 사용됩니다.
*   **`StyleSheet.flatten` vs `StyleSheet.compose` 활용:**
    *   `StyleSheet.flatten`은 여러 개의 스타일(특히 배열)을 하나의 스타일 객체로 병합하여 동적/조건부 스타일링에 유연하게 사용할 수 있음을 알게 되었습니다.
    *   `StyleSheet.compose`는 두 개의 스타일만을 병합하는 데 최적화되어 있으며, 단순한 베이스 스타일과 변형 스타일을 결합할 때 유용합니다.
    *   다수의 조건부 스타일이 적용되는 커스텀 버튼 컴포넌트에서는 `StyleSheet.compose`를 여러 번 중첩하는 것보다 `style` 프롭에 스타일 배열을 직접 전달하는 방식이 가독성과 유지보수 측면에서 훨씬 우수하며 React Native 커뮤니티의 표준 패턴임을 재확인했습니다.

---

## 🤔 궁금한 점 / 추가 학습 필요
*   `Switch` 컴포넌트에 대한 상세 설명 (속성, 타입, 예제)을 추가해야 합니다.
*   `FlatList` 및 `SectionList`의 `getItemLayout`을 실제로 적용하여 스크롤 성능을 최적화하는 구체적인 사례를 더 학습하고 싶습니다.
*   React Native 앱의 전반적인 접근성(Accessibility)을 향상시키기 위한 Best Practice와 도구 활용법에 대해 추가 학습이 필요합니다.
*   `Animated` API의 `useNativeDriver`가 `true`일 때와 `false`일 때의 성능 차이 및 어떤 속성에 적용 가능한지 심층적으로 이해하고 싶습니다.

---

## 🔗 참고 자료
- [React Native Components and APIs](https://reactnative.dev/docs/components-and-apis)
- [React Native Image Component](https://reactnative.dev/docs/image)
- [React Native TextInput Component](https://reactnative.dev/docs/textinput)
- [React Native FlatList Component](https://reactnative.dev/docs/flatlist)
- [React Native SectionList Component](https://reactnative.dev/docs/sectionlist)
- [React Native StyleSheet API](https://reactnative.dev/docs/stylesheet)