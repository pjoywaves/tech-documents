---
template: "TIL"
title: "TanStack Query와 API 설계 학습 및 기존 방식 개선 방안"
created_at: "2025-11-14 19:01"
created_by:
  name: "박기쁜"
  email: "gbpark@herit.net"
participants: []
tags:
  - "TanStack Query"
  - "API 설계"
  - "프론트엔드"
  - "캐싱"
  - "REST API"
category: "TIL"
status: "draft"
visibility: "internal"
related_docs: []
custom: {}
---

# TIL (Today I Learned) - 2025-11-17

## 📚 학습 주제
TanStack Query를 활용한 효율적인 서버 상태 관리 및 API 설계 원칙 학습

---

## 🔑 핵심 내용
- TanStack Query는 React Query에서 확장된 범용 서버 상태 관리 라이브러리로, 데이터 캐싱, 중복 요청 제거, 자동 갱신 등의 기능을 통해 비동기 데이터 처리를 간소화합니다.
- `useQuery` (데이터 조회), `useInfiniteQuery` (무한 스크롤), `useMutation` (데이터 변경) 등 주요 훅을 활용하여 복잡한 서버 상태 로직을 효율적으로 구현할 수 있습니다.
- API는 애플리케이션 간 통신을 위한 인터페이스이며, REST, SOAP, GraphQL, gRPC 등 다양한 유형이 각각의 특징과 장단점을 가집니다.
- 성공적인 API 설계를 위해서는 일관성, 무상태성, 리소스 중심 설계 원칙을 준수하고, 요청/응답 스펙 정의, 인증, 버전 관리, 성능 최적화, 보안, 문서화 등 다양한 요소를 종합적으로 고려해야 합니다.
- 언더페칭/오버페칭 문제 해결, 효율적인 캐싱 전략 (CDN, Redis, ETag), 그리고 Offset 및 Cursor 기반 페이지네이션 기법을 이해하고 적용하는 것이 중요합니다.

---

## 📝 상세 내용

### TanStack Query: 강력한 서버 상태 관리 라이브러리

#### TanStack Query란?
TanStack Query는 서버 상태를 효율적으로 관리할 수 있는 라이브러리로, 기존에는 React Query라는 이름으로 React 프레임워크에 특화되었으나, v4부터 Vue, Svelte 등 다양한 프레임워크에서도 활용할 수 있도록 기능이 확장되어 `TanStack Query`로 명칭이 변경되었습니다.

**서버 상태 관리의 이점:**
-   **데이터 가져오기 및 캐싱:** 한 번 가져온 데이터를 임시 저장하여 재요청 시 빠르게 응답합니다.
-   **동일 요청의 중복 제거:** 동일한 데이터 요청이 여러 번 발생해도 실제 서버 요청은 한 번만 이루어집니다.
-   **신선한 데이터 유지:** 데이터의 유효 기간을 관리하여 항상 최신 상태의 데이터를 제공합니다.
-   **성능 최적화:** 무한 스크롤, 페이지네이션 등의 UI 패턴에서 최적화된 성능을 제공합니다.
-   **자동 갱신:** 네트워크 재연결이나 요청 실패 시 자동으로 데이터를 다시 가져와 동기화를 유지합니다.

#### 데이터 캐싱 (Caching)
TanStack Query는 데이터를 가져올 때 `queryKey`를 사용하며, 이 키는 캐시된 데이터를 식별하고 새로운 데이터를 가져올지 캐시된 데이터를 사용할지 결정하는 기준이 됩니다. `queryKey`와 일치하는 캐시 데이터가 없으면 서버에서 데이터를 가져와 캐시하고, 있다면 캐시된 데이터를 사용함으로써 중복 요청을 줄입니다.

```javascript
import { useQuery } from '@tanstack/react-query'

export default function DelayedData() {
  const { data } = useQuery({
    queryKey: ['delay'], // 고유한 쿼리 키
    queryFn: async () => (await fetch('https://api.heropy.dev/v0/delay?t=1000')).json()
  })
  return <div>{JSON.stringify(data)}</div>
}
```

#### 데이터의 신선도 (Data Freshness)
데이터의 '유통기한'은 `staleTime` 옵션으로 지정할 수 있습니다. `staleTime`이 경과하면 데이터는 '상한' 것으로 간주되며, `isStale` 상태로 이를 확인할 수 있습니다. 데이터가 신선하지 않을 때 새로운 요청이 발생하면 백그라운드에서 데이터를 다시 가져와 캐시를 갱신합니다.

```javascript
import { useQuery } from '@tanstack/react-query'

export default function DelayedData() {
  const { data, isStale } = useQuery({
    queryKey: ['delay'],
    queryFn: async () => (await fetch('https://api.heropy.dev/v0/delay?t=1000')).json(),
    staleTime: 1000 * 10 // 10초 후 상함. 즉, 10초 동안 신선함.
  })
  return (
    <>
      <div>데이터가 {isStale ? '상했어요..' : '신선해요!'}</div>
      <div>{JSON.stringify(data)}</div>
    </>
  )
}
```

#### `useQuery` 훅
가장 기본적인 쿼리 훅으로, 컴포넌트에서 서버 데이터를 **가져올 때** 사용합니다.

```typescript
const 반환 = useQuery<데이터타입>(옵션)
```

**주요 옵션:**
-   `queryKey`: 쿼리를 식별하는 고유한 배열 형태의 값. 다중 아이템 키 사용 시 순서가 중요합니다.
-   `queryFn`: 데이터를 가져오는 비동기 함수. 데이터를 반환하거나 오류를 던져야 합니다.
-   `enabled`: 쿼리 자동 실행 여부를 제어합니다. (기본값: `true`)
-   `staleTime`: 데이터가 '상했다고' 간주되기까지의 시간 (기본값: 0, 즉시 stale).
-   `gcTime`: 비활성(inactive) 캐시 데이터가 메모리에 남아있는 시간 (기본값: 5분).
-   `initialData`, `placeholderData`: 데이터가 로딩되기 전에 표시할 초기/임시 데이터.
-   `select`: 가져온 데이터를 변형하는 함수.
-   `structuralSharing`: 새로운 데이터와 이전 데이터를 비교하여 변경되지 않은 부분은 이전 데이터를 재사용함으로써 불필요한 리렌더링을 방지하고 메모리를 최적화합니다.
-   `refetchInterval`: 주기적으로 데이터를 자동 갱신하는 시간 간격.
-   `refetchOnMount`, `refetchOnReconnect`, `refetchOnWindowFocus`: 특정 이벤트 발생 시 데이터 갱신 여부.
-   `retry`, `retryDelay`: 쿼리 실패 시 재시도 횟수 및 간격.
-   `meta`: 쿼리에 대한 추가 정보를 지정할 수 있습니다.

**주요 반환 값:**
-   `data`: 성공적으로 가져온 데이터.
-   `error`: 오류가 발생했을 때의 오류 객체.
-   `status`: 쿼리의 현재 상태 (`'pending'`, `'error'`, `'success'`).
-   `isPending`: 캐시된 데이터가 없고 쿼리가 아직 완료되지 않은 상태.
-   `isFetching`: 쿼리 함수가 실행 중 (데이터를 가져오는 중).
-   `isLoading`: `isFetching && isPending`과 동일, 쿼리의 첫 번째 가져오기가 진행 중인 경우.
-   `isStale`: 캐시된 데이터가 `staleTime`이 경과되었거나 무효화되었는지 여부.
-   `isSuccess`, `isError`: 쿼리 성공 또는 오류 발생 여부.
-   `refetch`: 데이터를 수동으로 다시 가져오는 함수.

```javascript
import { useQuery, useQueryClient, queryOptions } from '@tanstack/react-query'

const options = queryOptions({
  queryKey: ['delay'],
  queryFn: async () => (await fetch('https://api.heropy.dev/v0/delay?t=1000')).json(),
  staleTime: 1000 * 10
})

export default function DelayedData() {
  const queryClient = useQueryClient()
  const { data, isStale } = useQuery(options)
  
  async function fetchData() {
    const data = await queryClient.fetchQuery(options)
    console.log(data) // 캐시된 데이터 또는 새로 가져온 데이터
  }
  return (
    <>
      <div>{data?.time}</div>
      <div>데이터가 상했나요?: {JSON.stringify(isStale)}</div>
      <button onClick={fetchData}>데이터 가져오기!</button>
    </>
  )
}
```

#### `useInfiniteQuery` 훅
'더 보기' 버튼이나 무한 스크롤과 같은 UI를 구현할 때 추가 데이터를 가져오는 데 사용합니다. `useQuery`의 모든 옵션을 사용하며, 추가 옵션과 반환 값을 제공합니다.

```typescript
const 반환 = useInfiniteQuery<페이지타입>(옵션)
```

**추가 옵션:**
-   `getNextPageParam`: 새로운 다음 페이지를 가져오면 다음 페이지의 정보를 호출합니다.
-   `getPreviousPageParam`: 새로운 이전 페이지를 가져오면 이전 페이지의 정보를 호출합니다.
-   `initialPageParam`: 첫 번째 페이지의 번호 (필수 옵션).
-   `maxPages`: 저장 및 출력할 최대 페이지 수.

**추가 반환 값:**
-   `fetchNextPage`, `fetchPreviousPage`: 다음/이전 페이지를 가져오는 함수.
-   `hasNextPage`, `hasPreviousPage`: 다음/이전 페이지가 있는지 여부.
-   `isFetchingNextPage`, `isFetchingPreviousPage`: 다음/이전 페이지를 가져오는 중인지 여부.

#### `useMutation` 훅
데이터 생성, 수정, 삭제와 같은 **데이터 변경 작업**을 처리할 때 사용합니다. 성공, 실패, 로딩 등의 상태를 관리하며, 요청 실패 시 자동 재시도나 낙관적 업데이트(Optimistic Update) 같은 고급 기능도 지원합니다. 낙관적 업데이트는 서버 응답을 기다리지 않고 먼저 UI를 업데이트하여 사용자 경험을 향상시키는 기법입니다.

```typescript
const 반환 = useMutation(옵션)
```

**주요 옵션:**
-   `mutationFn`: 실행할 비동기 변이(mutation) 함수.
-   `onMutate`: 변이 함수가 실행되기 전에 호출되는 함수 (낙관적 업데이트 로직에 주로 사용).
-   `onSuccess`: 변이가 성공할 때 호출되는 함수.
-   `onError`: 변이 중 오류가 발생할 때 호출되는 함수.
-   `onSettled`: 변이가 성공하거나 실패해도 항상 호출되는 함수.
-   `retry`, `retryDelay`: 변이 실패 시 재시도 횟수 및 간격.

**주요 반환 값:**
-   `data`: 성공적으로 반환된 데이터.
-   `error`: 오류 발생 시 오류 객체.
-   `isPending`: 변이 함수가 실행 중인지 여부.
-   `isSuccess`, `isError`: 변이 성공 또는 오류 발생 여부.
-   `mutate`: 변이 실행 함수.
-   `mutateAsync`: 비동기적으로 변이를 실행하고 Promise를 반환하는 함수.

#### TanStack Query 개발자 도구 활용
TanStack Query는 전용 개발자 도구를 제공하여 쿼리 상태, 캐시 데이터, 뮤테이션 등을 시각적으로 확인하고 디버깅할 수 있도록 돕습니다.

**설치 및 사용:**
```bash
npm i @tanstack/react-query-devtools
```

```javascript
import {
  QueryClient,
  QueryClientProvider,
} from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'
import DelayedData from './components/DelayedData'

const queryClient = new QueryClient()

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <DelayedData />
      <ReactQueryDevtools initialIsOpen={false} /> // 개발자 도구 컴포넌트, initialIsOpen으로 초기 열림 여부 설정
    </QueryClientProvider>
  )
}
```
애플리케이션 실행 후 브라우저 우측 하단에 생기는 버튼을 클릭하여 개발자 도구를 열 수 있습니다.

---

### API 이해와 효율적인 설계

#### API (Application Programming Interface)란?
API는 애플리케이션 프로그래밍 인터페이스의 약자로, 서로 다른 소프트웨어 시스템이 통신하고 데이터를 교환할 수 있도록 정의된 규칙과 세트입니다.

**주요 특징:**
-   **추상화:** 복잡한 내부 로직을 숨기고 간단한 인터페이스를 제공하여 사용 편의성을 높입니다.
-   **표준화:** 일관된 방식으로 데이터 교환을 가능하게 하여 통합을 용이하게 합니다.
-   **모듈성:** 애플리케이션을 독립적인 모듈로 분리하여 개발 및 유지보수를 효율적으로 만듭니다.
-   **확장성:** 기존 시스템에 새로운 기능을 쉽게 추가하고, 다른 시스템과 연동하는 데 용이합니다.

#### API 유형
다양한 API 유형이 존재하며, 각각의 특성과 사용 목적에 따라 적절한 API를 선택하는 것이 중요합니다.

1.  **REST (Representational State Transfer)**
    -   HTTP 메서드 (GET, POST, PUT, DELETE 등)를 사용하여 리소스에 대한 CRUD(Create, Read, Update, Delete) 작업을 수행합니다.
    -   각 요청이 독립적인 **Stateless** 특징을 가집니다.
    -   URL로 리소스를 식별하며, 간결하고 직관적입니다.
    -   **장점:** 간단하고 직관적이며, 캐싱이 용이합니다.
    -   **단점:** 클라이언트가 필요한 데이터보다 많거나 적게 받는 **오버페칭(Overfetching)** 및 **언더페칭(Underfetching)** 문제가 발생할 수 있습니다.
    ```
    GET /api/users/123
    ```

2.  **SOAP (Simple Object Access Protocol)**
    -   XML 기반의 메시지 포맷을 사용하며, HTTP뿐만 아니라 SMTP 등 다양한 프로토콜을 통해 통신할 수 있습니다.
    -   복잡한 연산과 높은 보안성을 요구하는 엔터프라이즈 환경에서 주로 사용됩니다.
    -   **장점:** 높은 신뢰성과 보안성을 제공하며, 트랜잭션과 같은 복잡한 연산을 지원합니다.
    -   **단점:** 메시지 구조가 상대적으로 무겁고 복잡하여 개발 및 유지보수가 어렵습니다.
    ```xml
    <soap:Envelope>
      <soap:Body>
        <GetUser>
          <UserId>123</UserId>
        </GetUser>
      </soap:Body>
    </soap:Envelope>  
    ```

3.  **GraphQL**
    -   클라이언트가 필요한 데이터만 정확히 요청할 수 있도록 합니다.
    -   오버페칭과 언더페칭 문제를 해결하는 데 효과적입니다.
    -   단일 엔드포인트로 다양한 데이터 요청을 처리할 수 있습니다.
    -   **장점:** 효율적인 데이터 로딩이 가능하며, 강력한 타입 시스템을 통해 안정적인 개발을 지원합니다.
    -   **단점:** 서버 구현의 복잡성이 증가할 수 있으며, 캐싱 구현이 REST보다 까다로울 수 있습니다.
    ```graphql
    query {
      user(id: "123") {
        name
        email
        posts {
          title
        }
      }
    }
    ```

4.  **gRPC**
    -   Google에서 개발한 고성능 RPC(Remote Procedure Call) 프레임워크입니다.
    -   HTTP/2와 프로토콜 버퍼(Protocol Buffers)를 사용하여 효율적인 통신을 지원합니다.
    -   양방향 스트리밍을 지원하며, 마이크로서비스 아키텍처에서 특히 유용합니다.
    -   **장점:** 고성능, 낮은 지연 시간, 언어 중립적.
    -   **단점:** 브라우저 지원이 제한적이며, 프로토콜 버퍼 스키마 정의가 필요합니다.
    ```protobuf
    service UserService {
      rpc GetUser (GetUserRequest) returns (User) {}
    }

    message GetUserRequest {
      string user_id = 1;
    }

    message User {
      string name = 1;
      string email = 2;
    }
    ```

#### API 설계의 주요 원칙
-   **1. 일관성:**
    -   **엔드포인트 명명 규칙 통일:** 복수형 명사 사용 (`/users`, `/comments`), 일관된 대소문자 사용 (소문자 추천).
    -   **응답 형식 표준화:** 일관된 JSON 구조 사용 (`{ "data": {...}, "meta": {...} }`).
    -   **오류 처리 방식 일관:** 표준 HTTP 상태 코드 사용, 상세하고 일관된 오류 메시지 제공.
-   **2. 무상태성 (Statelessness):**
    -   각각의 요청은 독립적이어야 하며, 이전 요청의 컨텍스트에 의존하지 않아야 합니다.
    -   서버 확장성을 향상시키고, 캐싱을 용이하게 하며, 클라이언트 구현을 단순화합니다.
-   **3. 리소스 중심 설계:**
    -   모든 것을 리소스로 생각하고 URL로 표현합니다.
    -   직관적인 URL 구조를 통해 리소스 접근 방식을 명확히 합니다.
    -   CRUD 작업과 HTTP 메서드의 자연스러운 매핑을 유도합니다.

#### API 설계 시 필요한 고려사항
효율적이고 유지보수 가능한 API를 설계하기 위해 다음 사항들을 종합적으로 고려해야 합니다.

1.  **무엇을 다룰지 정의 (도메인/리소스):** API가 제공할 핵심 자원과 기능을 명확히 합니다.
2.  **어떻게 제공할지 정의 (엔드포인트/메서드/스펙):** 각 리소스에 대한 접근 경로(엔드포인트), 허용할 HTTP 메서드, 그리고 각 메서드의 상세 스펙을 정의합니다.
3.  **요청 스펙:** 각 API가 받는 파라미터 (Path, Query, Body), 필드의 데이터 타입, 필수/선택 여부, 기본값 등을 상세히 정의합니다.
4.  **응답 스펙:** 응답 코드 (200, 404 등), 응답 Body의 형태, 페이지네이션 구조, 에러 응답 구조 등을 표준화합니다.
5.  **에러 구조:** 공통 에러 구조를 개발하여 일관된 방식으로 오류를 클라이언트에 전달합니다.
6.  **Authentication/Authorization:** API 별로 접근 권한 및 인증 방식 (토큰 인증 등)을 명확히 합니다.
7.  **Rate Limit:** 초당 요청 수 제한 등 API 사용량을 제어하는 정책을 수립합니다.
8.  **버전 관리:** API 변경 시 호환성 유지를 위해 버전 관리 전략 (`/api/v1`, Header 등)을 적용합니다.
9.  **성능 고려:** 페이지네이션 전략 (Offset vs. Cursor), N+1 문제 방지, 캐싱 전략 (CDN, Redis, ETag) 등을 통해 성능을 최적화합니다.
10. **보안 정책:** HTTPS 적용, CORS 설정, 입력 값 유효성 검증, SQL Injection 방어 등 보안 대책을 마련합니다.
11. **문서화:** Swagger, OpenAPI 등 도구를 활용하여 API 문서를 자동 생성하고 최신 상태로 유지하며, Request/Response 예시, 오류 코드 테이블 등을 제공합니다.
12. **테스트:** 설계된 API의 기능 및 성능을 검증하는 테스트 계획을 수립합니다.
13. **일관성:** 명명 규칙, 응답 형식, 상태 코드, 에러 포맷 등 전반적인 API 디자인에 일관성을 유지합니다.

---

### TanStack Query를 활용한 API 연동 (예시)

#### 1. 데이터 조회 (GET)
```typescript
// user.ts (API 클라이언트 설정)
import axios from 'axios';
export const userClient = axios.create({
  baseURL: 'https://api.example.com' // API 기본 URL
});
```
```typescript
// useUser.ts (TanStack Query 훅)
import { useQuery } from '@tanstack/react-query';
import { userClient } from './user'; // 위에서 정의한 API 클라이언트

const fetchUser = async (id: number) => {
  const response = await userClient.get(`/user/${id}`);
  return response.data.data; // 실제 데이터 반환
};

export const useUser = (id: number | null) => { // id가 null일 수도 있음을 명시
  return useQuery({
    queryKey: ['user', id], // 쿼리 키에 ID 포함
    queryFn: () => fetchUser(id!), // ID가 유효할 때만 fetchFn 호출
    enabled: !!id, // id가 있을 때만 쿼리 실행
    // staleTime, gcTime 등 추가 옵션 설정 가능
  });
};
```

#### 2. 데이터 생성 (POST)
```typescript
// useUser.ts (TanStack Query 훅)
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { userClient } from './user';

interface CreateUserPayload {
  name: string;
  email: string;
}

const createUser = async (payload: CreateUserPayload) => {
  const response = await userClient.post('/user', payload);
  return response.data.data;
};

export const useCreateUser = () => {
    const queryClient = useQueryClient();

    return useMutation({
      mutationFn: createUser,
      onSuccess: (newUser) => {
        // 사용자 생성 성공 시, 'user' 관련 캐시를 무효화하여 새로운 목록을 다시 가져오도록 유도
        queryClient.invalidateQueries({ queryKey: ['user'] }); 
        // 혹은 queryClient.setQueryData를 사용하여 캐시를 직접 업데이트할 수도 있습니다.
      },
      // onError, onMutate 등 추가 콜백 함수 설정 가능
    });
};
```

---

### 기존 API 방식과의 비교 및 개선점

**기존 방식의 문제점 (예시: 특정 Vue 프로젝트)**
-   **수동적인 API 호출 및 중복 요청:** 매 요청마다 `fetch` 또는 `axios`를 직접 호출하여 관리하며, 동일한 데이터를 여러 번 요청하는 중복 호출이 빈번하게 발생했습니다.
-   **불완전한 상태 관리:** `isLoading`, `isError`, `isSuccess`와 같은 API 호출 상태를 개별적인 변수로 수동으로 생성하고 토글하는 방식으로 관리하여 번거롭고 일관성이 부족했습니다.
-   **유지보수의 어려움:** `requestApi` 호출부터 응답 처리, 데이터 가공, 뷰 렌더링까지 한 Vue 페이지 스크립트 내에서 처리되어 파일 길이가 매우 길어졌습니다. 같은 기능을 여러 컴포넌트에서 각기 다르게 구현하여 API 로직의 유지보수 및 재사용성이 떨어졌습니다.
-   **부족한 버전 관리 및 문서화:** API의 버전 관리가 전혀 이루어지지 않았고, API 문서 또한 부재하여 개발자 간의 소통 및 협업에 어려움이 있었습니다.

**TanStack Query 적용 시 개선점**
-   **자동 캐싱 및 중복 요청 제거:** `queryKey`를 기반으로 데이터를 자동으로 캐싱하고, 동일한 데이터에 대한 중복 요청을 효과적으로 줄여 서버 부하를 감소시키고 응답 속도를 향상시킵니다.
-   **표준화된 상태 관리:** `isPending`, `isFetching`, `isError`, `isSuccess`, `isStale` 등 다양한 상태를 내장하여 API 호출의 라이프사이클을 일관되고 간편하게 관리할 수 있습니다.
-   **간결한 코드 및 로직 분리:** 데이터 페칭 로직을 Custom Hook (`useUser`, `useCreateUser` 등)으로 분리하여 UI 컴포넌트의 가독성을 높이고 관심사를 분리할 수 있습니다.
-   **자동 백그라운드 갱신 및 재시도:** 네트워크 재연결, 윈도우 포커스 시 자동으로 데이터를 갱신하고, 실패한 요청에 대한 자동 재시도 기능을 제공하여 안정성을 높입니다.
-   **개발자 도구를 통한 편리한 디버깅:** 전용 개발자 도구를 통해 쿼리 캐시 상태를 실시간으로 모니터링하며 디버깅 효율을 증대시킵니다.

**전반적인 API 개선 방향**
-   **API 응답/요청 구조 명확화:** 응답 및 요청 데이터의 구조를 사전에 명확하게 정의하고 표준화합니다.
-   **API 문서화 및 버전 관리:** Swagger, OpenAPI 등 도구를 활용하여 API 문서를 체계적으로 관리하고, 필요한 경우 API 버전 관리 전략을 도입합니다.
-   **API 호출 및 상태 관리 분리:** API 호출 로직을 전담하는 모듈이나 라이브러리를 사용하고, TanStack Query와 같은 서버 상태 관리 도구를 도입하여 상태 관리를 효율적으로 분리합니다.

---

## 💡 새롭게 알게 된 점

-   **언더페칭 (Underfetching)과 오버페칭 (Overfetching)**
    -   **언더페칭:** 클라이언트가 필요한 모든 데이터를 한 번의 요청으로 가져올 수 없어 여러 번의 추가 요청이 필요한 상황. (예: 사용자 정보를 가져온 후, 해당 사용자의 게시글을 별도로 다시 요청해야 하는 경우)
    -   **오버페칭:** 클라이언트가 필요로 하지 않는 추가 데이터까지 서버로부터 받아오는 상황. (예: 사용자 이름만 필요한데 모든 사용자 정보(주소, 전화번호 등)를 받아오는 경우)
-   **엔드포인트 (Endpoint)**
    -   API가 특정 리소스에 대한 요청을 받는 URL 경로를 의미합니다. 광의적으로는 네트워크에 연결된 물리적 또는 가상적 기기로, 네트워크와 정보를 주고받는 최종 지점을 뜻하기도 합니다.
-   **캐싱 (Caching) 전략**
    캐싱은 임시 저장소에 데이터를 저장해두어 나중에 같은 요청이 오면 더 빠르게 응답할 수 있도록 하는 기술입니다.
    1.  **CDN (Content Delivery Network)**
        -   **정의:** 전 세계 여러 지역에 분산된 서버(엣지 서버)에 정적 파일을 캐싱해두고, 사용자가 가장 가까운 서버에서 파일을 받아가도록 해주는 시스템.
        -   **활용 시점:** 이미지, JavaScript, CSS, 폰트, 비디오와 같은 정적 리소스, 또는 프론트엔드 배포 파일처럼 한 번 만들면 자주 변하지 않는 파일에 적합 (Vercel, CloudFront 등).
        -   **장점:** 전 세계 어디서든 빠른 로딩, 원본 서버 부하 감소, 대규모 트래픽 소화 용이.
        -   **흐름 예시:** 사용자가 `/assets/logo.png` 요청 → CDN 엣지 서버에 캐시 존재 시 즉시 반환 → 없으면 원본 서버에서 가져와 캐싱 후 반환.
    2.  **Redis 캐싱**
        -   **정의:** 메모리 기반의 초고속 데이터 저장소(키-값 DB)로, 데이터베이스보다 훨씬 빠르게 읽고 쓸 수 있어 캐싱용으로 최적화되어 있습니다.
        -   **활용 시점:** API 응답 캐싱, DB 조회 결과 캐싱, 세션 관리 (JWT Blacklist 등), 순간적으로 많은 요청을 버티기 위한 캐시로 활용. (예: 15분 간격으로 갱신되는 대시보드 에너지 데이터).
        -   **장점:** 매우 빠른 속도 (메모리 기반), TTL(Time To Live)을 통한 캐시 수명 관리 용이, 고성능 서비스에 필수적.
        -   **흐름 예시:** `GET /building/123/energy` 요청 → Redis에서 `key: energy:123` 조회 → 없으면 DB 조회 후 Redis에 캐싱 → 다음 요청부터 빠르게 처리.
    3.  **ETag (Entity Tag)**
        -   **정의:** 리소스가 변경되었는지 판단하기 위한 파일/응답의 '지문(hash)' 같은 것. HTTP 캐싱의 핵심 개념 중 하나입니다.
        -   **활용 시점:** 정적 파일 캐싱, API 응답 캐싱 시 클라이언트 캐시를 효율적으로 사용하고 싶을 때 (변경되지 않은 데이터는 캐시를 쓰도록 유도).
        -   **장점:** 네트워크 트래픽 감소, 불필요한 재요청 방지, 클라이언트 측 캐시 활용 증대.
        -   **흐름 예시:** 서버가 파일/API 응답과 함께 `ETag: "abcd1234"` 헤더 전송 → 브라우저는 다음 요청 시 `If-None-Match: "abcd1234"` 헤더로 전송 → 서버는 해시 비교 후 같으면 `304 Not Modified` 반환 (캐시 사용), 다르면 새 파일 `200 OK`와 함께 전송.
-   **다양한 페이지네이션 (Pagination) 방식**
    1.  **Offset 기반 페이지네이션**
        -   **정의:** 몇 번째 항목부터 가져올지 (offset)와 몇 개를 가져올지 (limit)를 기준으로 데이터를 잘라내는 방식. 데이터의 순서(레코드 번호)를 기준으로 합니다.
        -   **예시:** `GET /items?offset=20&limit=10`
        -   **장점:** 구현이 매우 간단하고 직관적입니다.
        -   **단점:** 데이터가 변경되면 (삭제, 삽입) 페이지 전체가 밀려 중복되거나 누락된 데이터가 발생할 수 있습니다. `offset` 값이 커질수록 데이터베이스 성능이 저하될 수 있어 대규모 데이터셋이나 무한 스크롤에는 불리합니다.
    2.  **Cursor 기반 페이지네이션**
        -   **정의:** 특정 항목 (cursor) 다음에 있는 데이터를 가져오는 방식으로, 데이터의 연속성을 보장합니다. 보통 이전에 가져온 마지막 항목의 고유한 키(ID, 타임스탬프 등)를 커서로 사용합니다.
        -   **예시:** `GET /items?cursor=CT12E&limit=10` (CT12E 다음에 있는 10개 항목 가져오기)
        -   **장점:** 데이터 변경에 관계없이 안정적이며, `offset` 값에 무관하게 항상 빠른 성능을 유지합니다. 무한 스크롤 구현에 최적화되어 중복/누락이 거의 발생하지 않습니다.
        -   **단점:** `offset` 방식보다 구현이 약간 더 복잡하며, 정렬 기준이 명확해야 합니다.
    
    **페이지네이션 방식 비교**

    | 항목            | Offset 기반 페이지네이션             | Cursor 기반 페이지네이션                    |
    | :-------------- | :----------------------------------- | :------------------------------------------ |
    | **기준**        | 레코드 번호 기반 (`offset`, `limit`) | 특정 item의 고유 키 기반 (`cursor`, `limit`) |
    | **안정성**      | 데이터 변할 때 밀림 발생 가능        | 데이터 변화에 안정적                        |
    | **성능**        | `offset`이 커질수록 느려짐           | 항상 빠름                                   |
    | **무한 스크롤** | 잘 맞지 않음                         | 최적                                        |
    | **중복/누락**   | 발생 가능                            | 거의 없음                                   |
    | **구현 난이도** | 매우 쉬움                            | 약간 노력 필요                              |

---

## 🤔 궁금한 점 / 추가 학습 필요
-   TanStack Query의 `useMutation` 훅을 활용한 낙관적 업데이트(Optimistic Update)를 실제 프로젝트에 적용할 때의 구체적인 구현 패턴과 예상되는 문제점 및 해결 방안.
-   GraphQL 또는 gRPC와 같은 API 유형을 실제 프로젝트에 적용할 경우의 백엔드 및 프론트엔드 설계 고려사항 및 개발 경험.
-   복잡한 서버 상태를 가진 애플리케이션에서 TanStack Query의 `queryClient`를 통한 캐시 무효화(`invalidateQueries`) 및 직접 캐시 업데이트(`setQueryData`) 전략을 효과적으로 사용하는 방법.
-   기존 레거시 Vue 프로젝트에 TanStack Query를 점진적으로 도입할 때의 마이그레이션 전략 및 주의사항.

---

## 🔗 참고 자료
-   [https://www.heropy.dev/p/HZaKIE](https://www.heropy.dev/p/HZaKIE)
-   [https://frontend-manchoon.tistory.com/132](https://frontend-manchoon.tistory.com/132)
-   [https://jinn-blog.tistory.com/190](https://jinn-blog.tistory.com/190)
-   [https://velog.io/@yoonezi/rest-API%EC%9D%98-%EC%96%B8%EB%8D%94%ED%8E%98%EC%B9%AD-%EC%98%A4%EB%B2%84%ED%8E%98%EC%B9%AD](https://velog.io/@yoonezi/rest-API%EC%9D%98-%EC%96%B8%EB%8D%94%ED%8E%98%EC%B9%AD-%EC%98%A4%EB%B2%84%ED%8E%98%EC%B9%AD)
-   [https://jinn-blog.tistory.com/190](https://jinn-blog.tistory.com/190)