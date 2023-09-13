# Caching in Next.js

넥스트는 애플리케이션의 성능을 향상시키고 렌더와 데이터 요청 캐싱에 대한 비용을 절감해준다.
이 페이지는 넥스트의 캐싱 메카니즘, api, 그들이 서로 어떻게 상호작용하는지 다룬다.

---

## Overview

목적에 따른 다른 캐싱 메카니즘에 대한 오버뷰이다.

![](https://github.com/yujiseok/nextstep-with-next/assets/83855636/7439dcba-f82e-4778-aac7-1f96ad2b93af)

기본적으로, 넥스트는 할 수 있는 만큼 성능 최적화와 비용을 절감한다. 이것은 라우트가 정적으로 렌더되고 데이터 요청이 선택을 안할때를 제외하고 캐시된다는 것이다.
다이어그램은 기본적인 캐싱 동작을 보여준다. 라우트가 빌드 시 정적으로 렌더되고 정적 라우트가 처음으로 방문 되었을 때.

![](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Fcaching-overview.png&w=3840&q=75&dpl=dpl_BDGcVPZewJz9vY9dPCQcx3UcGzVA)

캐싱 동작은 라우트가 정적인지 동적인지, 데이터가 캐시되었는지 안되었는지 그리고 첫 방문의 요청인지 혹은 후속 네비게이션일지에 따라 변화한다.
사용 용도에 따라, 각 라우트와 데이터 요청에 캐싱 동작을 설정할 수 있다.

---

## Request Memoization

리액트는 페치 api를 같은 url과 옵션에 대해 자동으로 요청을 메모이즈하도록 확장했다.
이것은 동일 데이터를 사용하는 여러 리액트 컴포넌트에서 오직 한번 실행되는 페치 함수를 호출할 수 있다는 뜻이다.

![](https://nextjs.org/_next/image?url%253D%252Fdocs%252Fdark%252Fdeduplicated-fetch-requests.png%2526w%253D3840%2526q%253D75%2526dpl%253Ddpl_BDGcVPZewJz9vY9dPCQcx3UcGzVA)

예를 들어, 같은 데이터를 라우트에 걸쳐 필요로 할 때, 페치 데이터를 프랍으로 넘겨줄 필요가 없다.
대신에, 성능의 문제를 걱정하지 않고 데이터가 필요한 컴포넌트에서 페치할 수 있다.

```tsx
async function getItem() {
  // The `fetch` function is automatically memoized and the result
  // is cached
  const res = await fetch("https://.../item/1");
  return res.json();
}

// This function is called twice, but only executed the first time
const item = await getItem(); // cache MISS

// The second call could be anywhere in your route
const item = await getItem(); // cache HIT
```

How Request Memoization Works

![](https://nextjs.org/_next/image?url%253D%252Fdocs%252Fdark%252Frequest-memoization.png%2526w%253D3840%2526q%253D75%2526dpl%253Ddpl_BDGcVPZewJz9vY9dPCQcx3UcGzVA)

- 라우트를 렌더할 때, 처음 특정 요청은 호출된디. 이것의 결과는 메모리에 없기에 캐시가 `MISS`가 된다.
- 그러므로, 함수는 실행되고, 데이터는 외부 소스로 페치된다. 그리고 결과가 메모리에 저장된다.
- 동일 렌더 패스의 요청에 대한 후속 함수 호출은 캐시가 `HIT`되고, 데이터는 함수를 실행하지 않고 메모리에서 리턴된다.
- 라우트가 렌더되고 렌더링 패스가 완료되면, 메모리는 초기화되며 모든 요청 메모이제이션 엔트리는 비워진다.

> - 요청 메모이제이션은 리액트의 기능으로, 넥스트의 기능이 아니다. 다른 메모이제이션과 어떻게 상호작용하는지 보여주기 위해 여기에 포함됨
> - 페치의 메모이제이션은 오직 겟 메서드에만 해당된다.
> - 메모이제이션은 오직 리액트 컴포넌트 트리에만 적용된다. 즉:
>   - `generateMetadata`, `generateStaticParams`, Layouts, Pages, 서버 컴포넌트의 페치 요청이 적용된다.
>   - 리액트 컴포넌트 트리가 아닌 라우트 핸들러에서 페치는 적용되지 않는다.
> - 페치가 맞지 않는 경우(e.g. some database clients, CMS clients, or GraphQL clients), 리액트의 캐시 함수를 통해 함수를 메모이즈할 수 있다.

### Duration

캐시는 리액트 컴포넌트 트리가 렌더링을 끝낼 때까지 서버 요청의 수명 동안 지속된다.

### Revalidating

메모이제이션은 서버 요청 간에 공유되지 않고 렌더링 중에만 적용되므로, 재검증할 필요가 없다.

### Opting out

페치 요청의 메모이제이션을 나가기 위해, `AbortController` `signal`을 요청에 전달하면 된다.

```js
const { signal } = new AbortController();
fetch(url, { signal });
```

---

## Data Cache

넥스트는 서버 요청과 배포 전반에 걸쳐 페치한 데이터 결과를 지속하는 빌트인 데이터 캐시를 갖는다.
이것은 넥스트가 네이티브 페치 api를 확장해 각 서버 요청에 각자의 캐시를 지속하도록 한다.

> 브라우저에서 페치의 캐시 옵션은 브라우저의 http 캐시와 상호작용 하고, 넥스트에서 캐시 옵션은 서버 사이드 요청이 서버 데이터 캐시와 어떻게 상호작용하는 지를 의미한다.

기본적으로, 페치를 사용하면 데이터 요청은 캐시된다. 캐싱 동작을 설정하기 위해 `next.revalidate` 옵션과 캐시를 사용할 수 잇다.

데이터 캐시의 동작 방식

![](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Fdata-cache.png&w=1920&q=75&dpl=dpl_An4L6XuPKRutmZMcpBqLzStFEXgS)

- 첫 페치 요청은 렌더 중에 호출 된다. 넥스트는 캐시된 응답에서 데이터 캐시를 확인한다.
- 만약 캐시된 응답이 있다면, 즉시 반환하고 메모이즈한다.
- 만약 캐시된 응답이 없다면, 요청이 일어나고, 그 결과가 데이터 캐시에 저장되고 메모이즈된다.
- 캐시되지 않은 데이터 (e.g. `{ cache: 'no-store' }`)의 결과는 항상 데이터 소스에서 페치되고, 메모이즈된다.
- 데이터가 캐시가 되든 안되든, 요청은 항상 메모이즈되어 리액트 렌더 패스 시 동일 요청을 피하도록한다.

> 데이터 캐시와 요청 메모이제이션의 차이
> 두 캐싱 메카니즘은 캐시된 데이터를 재사용하고, 데이터 캐시는 전체 요청과 배포 시 지속 되며 성능을 향상시키지만 메모이제이션은 오직 요청 시에만 지속된다.
>
> 메모이제이션을 통해, 데이터 캐시 서버(cdn,edge) 또는 데이터 소스(데이터베이스, cms)로 네트워크 경계를 넘어야하는 동일 렌더 패스의 중복된 요청을 줄일 수 있다. 데이터 캐시는 원래 데이터 소스에 관한 요청을 줄일 수 있다.

### Duration

데이터 캐시는 재검증하거나 선택 해제하지 않는 한 들어오는 요청 및 배포 전반에 걸쳐 지속된다.

### Revalidating

캐시된 데이터는 두 방법으로 재검증 될 수 있다:

- 시간 기반 재검증: 일정 시간 지난 뒤 데이터 재검증은 새로운 요청을 만든다. 이것은 데이터가 자주 바뀌지 않으며, 신선함이 중요하지 않을 때 유용하다.
- 요구에 따른 재검증: 이벤트에 기반한 재검증이다. 요구에 따른 재검증은 태그 기반 또는 경로 기반 접근으로 데이터 그룹을 한번에 재검증한다. 새로운 데이터가 가능한 빨리 보여야할 때 유용하다.

#### Time-based Revalidation

시간 인터벌로 데이터를 재검증하기 위해선 페치의 `next.revalidate`를 통해 캐시의 생명주기를 설정할 수 있다.

```ts
// Revalidate at most every hour
fetch("https://...", { next: { revalidate: 3600 } });
```

또는, 라우트 세그먼트 설정을 통해 페치를 사용할 수 없는 경우나 모든 페치 요청을 설정할 수 있다.

시간 기반 재검증이 동작하는 방법

![](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Ftime-based-revalidation.png&w=1920&q=75&dpl=dpl_An4L6XuPKRutmZMcpBqLzStFEXgS)

- 처음 `revalidate`와 페치 요청이 호출되면, 데이터는 외부 데이터 소스로부터 데이터를 페치하고 데이터 캐시에 저장한다.
- 특정 기간내 모든 요청은 캐시된 데이터를 반환한다.
- 기간이 지나면, 다음 요청은 여전히 캐시된 데이터를 반환한다.
  - 넥스트는 백그라운드에서 데이터 재검증을 시도한다.
  - 데이터 페치가 성공하면, 넥스트는 새로운 데이터를 데이터 캐시에 업데이트한다.
  - 만약 백그라운드 재검증이 실패하면, 이전 데이터는 유지된다.

이것은 [stale-while-revalidate](https://web.dev/stale-while-revalidate/) 동작과 비슷하다.

#### On-demand Revalidation

데이터는 경로 또는 캐시 태그에 의해 요청에 의해 재검증 된다.

요구 기반 재검증이 동작하는 방법

![](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Fon-demand-revalidation.png&w=1920&q=75&dpl=dpl_An4L6XuPKRutmZMcpBqLzStFEXgS)

- 페치 요청이 처음 호출되면, 외부 데이터 소스에서 데이터는 페치된 후 저장된다.
- 요구 기반 재검증이 일어나면, 상응하는 캐시 엔트리가 캐시에서 제거된다.
  - 이것은 스테일 데이터를 새로운 데이터가 페치될때까지 유지하는, 시간 기반 재검증과 다르다.
- 다음 요청이 만들어지면, 캐시 `MISS`되고 데이터는 외부 데이터 소스에서 페치된 후 저장된다.

### Opting out

각 데이터 요청에, 캐시 옵션의 `no-store`을 통해 캐시 설정을 뺄 수 있다. 이것은 페치가 호출 될때마다 데이터가 페치되는 것을 의미한다.

```ts
// Opt out of caching for an individual `fetch` request
fetch(`https://...`, { cache: "no-store" });
```

또는 라우트 세그먼트 설정을 통해 특정 라우트 세그먼트의 캐시를 뺄 수 있다. 이것은 라우트 세그먼트의 모든 데이터 요청과 써드 파티 라이브러리에도 영향을 끼친다.

```ts
// Opt out of caching for all data requests in the route segment
export const dynamic = "force-dynamic";
```

---

## Full Route Cache

> 자동 정적 최적화, 정정 사이트 생성, 또는 정적 렌더링이라는 용어는 빌드 시 애플리케이션의 경로를 렌더링하고 캐싱하는 프로세스를 나타내기 위해 같은 의미로 사용되는 것을 볼 수 있다.

넥스트는 빌드 시 자동으로 렌더와 캐시한다. 이것은 매 요청마다 서버에서 렌더하는 것 대신 캐시된 라우트를 제공해 빠른 페이지 로드를 제공한다.

전체 라우트 캐시 동작을 이해하기 위해 리액트가 어떻게 렌더링을 다루는지와 넥스트가 어떻게 캐시하는지 아는 것이 도움이 될 것이다.

### 1. React Rendering on the Server

서버에서, 넥스트는 리액트의 api를 사용해 렌더링을 조율한다. 렌더링 작업은 각 라우트 세그먼트와 서스펜스 경계 청크로 나뉜다.

각 청크는 두 단계를 거쳐 렌더된다.

1. 리액트는 서버 컴포넌트를 스트리밍을 최적화하기 위해, 리액트 서버 컴포넌트 페이로드라는 특별한 데이터 포맷으로 렌더한다.
2. 넥스트는 리액트 서버 컴포넌트 페이로드와 클라이언트 컴포넌트 자바스크립트로 서버에서 html을 렌더한다.

즉, 작업을 캐시하거나 응답을 보내기 전 모든 것이 렌더링될 때까지 기다릴 필요가 없다. 대신 작업이 완료되면 응답을 스트리밍할 수 있다.

### 2. Next.js Caching on the Server (Full Route Cache)

![](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Ffull-route-cache.png&w=1920&q=75&dpl=dpl_An4L6XuPKRutmZMcpBqLzStFEXgS)

넥스트의 기본 동작은 서버 라우트의 렌더 결과를 캐시하는 것이다. 이것은 빌드 시 정적으로 렌더되는 라우트 또는 재검증 시 적용된다.

### 3. React Hydration and Reconciliation on the Client

요청 시, 클라이언트에서:

1. html은 상호작용이 없는 클라이언트 컴포넌트와 서버 컴포넌트의 프리뷰로 보여진다.
2. 리액트 서버컴포넌트 페이로드는 돔 업데이트를 위해 클라이언트와 렌더되 서버 컴포넌트 트리를 재조정한다.
3. 자바스크립트가 클라이언트 컴포넌트의 하이드레이트를 위해 사용되며 상호작용이 가능하게 한다.

### 4. Next.js Caching on the Client (Router Cache)

리액트 서버컴포넌트 페이로드는 클라이언트 사이드 라우터 캐시에 라우트 세그먼트 별로 나뉘어 저장된다. 라우터 캐시는 이전에 방문한 라우트를 저장하고 미래의 라우트를 프리페치해 향상된 이동 경험에 사용된다.

### 5. Subsequent Navigations

프리페칭 시 후속 이동에서, 넥스트는 리액트 서버컴포넌트 페이로드가 라우터 캐시에 저장되었는지 확인한다. 만약 있다면, 서버로의 새로운 요청을 무시한다.

만약 라우트 세그먼트가 캐시에 없다면, 넥스트는 서버에서 리액트 서버 컴포넌트 페이로드를 페치한 후, 클라이언트의 라우터 캐시를 채운다.

### Static and Dynamic Rendering

빌드 시 라우트가 캐시될지 또는 되지 않을지 정해지는데, 정적 또는 동적으로 렌더되는지에 달려있다.
정적 라우트는 기본적으로 캐시되고, 동적 라우트는 요청 시 렌더되어, 캐시되지 않는다.

![](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Fstatic-and-dynamic-routes.png&w=1920&q=75&dpl=dpl_An4L6XuPKRutmZMcpBqLzStFEXgS)

### Duration

기본적으로, 전체 라우트 캐시는 지속된다. 즉, 렌더의 결과는 유저의 요청에 걸쳐 캐시된다.

### Invalidation

전체 라우트 캐시를 무효화하는 두 가지 방법:

- 데이터 재검증: 데이터 캐시 재검증은 서버에서 컴포넌트를 리렌더하고 새로운 결과를 캐시해 라우터 캐시를 무효화한다.
- 재배포: 데이터 캐시와 다르게, 배포 전반에 걸쳐 지속되는 전체 라우트 캐시는 새 배포시 지워진다.

### Opting out

전체 라우트 캐시를 뺄 수 있다. 즉, 모든 요청에 동적 렌더를 통해

- 동적 함수 사용: 이것은 요청시 동적 렌더로 라우트를 전체 라우트 캐시에서 뺄 수 있다. 데이터 캐시는 여전히 사용된다.
- `dynamic = 'force-dynamic'` 또는 `revalidate = 0`를 라우트 세그먼트 설정으로 사용하기: 이것은 전체 라우트 캐시와 데이터 캐시를 무시한다. 즉 매 데이터 요청시 컴포넌트가 리렌더된다. 라우터 캐시는 클라이언트 측 캐시이므로 계속 적용된다.
- Opting out of the Data Cache: 만약 라우트가 캐시되지 않은 페치 요청을 갖는다면, 이것은 전체 라우트 캐시에서 라우트를 제외한다. 특정 페치 요청의 데이터가 매 요청마다 페치된다. 다른 페치 요청은 캐시된다. 이것은 캐시되고 안된 데이터의 하이브리드를 허용한다.

---

## Router Cache

> 라우터 캐시가 클라이언트 사이드 캐시 또는 프리페치 캐시로 불리는 것을 볼 수 있다.
> 프리페치 캐시는 프리페치된 라우트 세그먼트로 클라이언트 사이드 캐시는 방문되고 프리페치된 세그먼트를 갖는 전체 라우터 캐시로 불린다.
> 이 캐시는 넥스트와 서버 컴포넌트에 특별히 적용되고 결과는 비슷하더라도 브라우저의 bfcache와는 다르다.

넥스트는 인메모리 클라이언트 캐시를 갖는데, 리액트 서버컴포넌트 페이로드, 각 라우트 세그먼트 단위, 유저 세션 기간을 저장한다.
이것은 라우터 캐시라 불린다.

라우터 캐시 동작 방법

![](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Frouter-cache.png&w=1920&q=75&dpl=dpl_An4L6XuPKRutmZMcpBqLzStFEXgS)

유저가 라우트를 이동하면, 넥스트는 방문된 라우트 세그먼트와 유저가 방문할 것 같은 프리페치된 라우트를 캐시한다.

이것은 유저의 이동 경험을 증대시킨다:

- 라우트가 캐시되었고 새로운 라우트가 프리페치되고 부분 렌더되었기에 즉각적인 앞/뒤 이동이 가능하다.
- 이동시 풀페이지 리로드가 없으며 리액트 상태 및 브라우저 상태를 유지한다.

> 라우터 캐시와 풀 라우트 캐시의 차이점
>
> 라우터 캐시는 유저 세션 동안 브라우저에 리액트 서버컴포넌트 페이로드를 임시로 저장한다. 반면에, 풀 라우트 캐시는 서버에서 유저의 여러 요청에 대한 리액트 서버컴포넌트 페이로드와 html을 지속적으로 저장한다.
>
> 풀 라우트 캐시가 오직 정적 렌더된 라우트만 캐시하는 반면, 라우터 캐시는 정적 그리고 동적 렌더된 라우트를 모두에 적용된다.

### Duration

캐시는 브라우저의 임시 메모리에 저장된다. 라우터 캐시의 지속시간을 결정하는 두 요소는 다음과 같다:

- 세션: 이동 동안에 캐시는 유지된다. 하지만, 페이지 새로고침 시 제거된다.
- 자동 무효화 기간: 각 세그먼트의 캐시는 일정 시간이 지나면 자동으로 무효화된다. 이 기간은 라우트가 정적 또는 동적 렌더인지에 달려있다.
  - 동적 렌더: 30초
  - 정적 렌더: 5분

페이지 새로고침 시 캐시된 세그먼트가 모두 지워지지만, 자동 무효화 기간은 마지막으로 액세스하거나 생성된 시간 이후의 개별 세그먼트에만 영향을 끼친다.

`prefetch={true}` 추가 또는 `router.prefetch`를 호출해 동적으로 렌더된 라우트를 5분 동안 캐시할 수 있다.

### Invalidation

라우터 캐시를 무효화할 수 있는 두 가지 방법이 있다:

- 서버 액션에서:
  - 경로 또는 캐시 태그를 사용한 요구 기반 재검증
  - 쿠키셋 또는 쿠키딜리트를 사용해 오래된 쿠키를 사용하는 것을 방지하기 위한 라우터 캐시 무효화
- `router.refresh`를 호출해 라우터 캐시를 무효화하고 현재 라우트에 대해 서버에 새롭게 요청

### Opting out

라우터 캐시를 제외하는 것은 불가능하다.

링크 컴포넌트의 `prefetch` 프랍을 false로 설정해 프리페치를 제외할 수는 있다. 하지만, 여전히 탭바, 앞뒤 이동과 같은 중첩되 세그먼트 간의 즉각 이동이 가능하도록 30초 동안 세그먼트가 임시로 저장된다. 방문환 라우트는 계속 캐시된다.

---

## Cache Interactions

다른 캐싱 메카니즘을 설정할 때, 그들이 어떻게 서로 상호작용하는지 아는 것이 중요하다.

### Data Cache and Full Route Cache

- 재검증 또는 데이터 캐시 제외는 렌더 결과에 기반한 데이터 풀 라우트 캐시를 무효화한다.
- 풀 라우트 캐시의 무효화 혹은 제외는 데이터 캐시에 영향을 끼치지 않는다. 동적으로 라우트를 렌더해 캐시된 데이터와 그렇지 않은 데이터를 가질 수 있다. 대부분의 페이지가 캐시된 데이터를 사용하지만 요청 시 페치하는 몇 가지 컴포넌트가 있는 경우 유용하다. 모든 데이터를 페치하는 성능 문제를 고려하지 않고 동적으로 렌더할 수 있다.

### Data Cache and Client-side Router cache

- 라우트 핸들러에서 데이터 캐시를 재검증해도 라우터 핸들러는 특정 라우트에 연결되어 있지 않으므로 라우터 캐시가 즉시 무효화되지 않는다. 즉 라우터 캐시는 강력 새로고침 전까지 또는 자동 무효화 기간이 경과했을 때까지 이전 페이로드를 계속 제공한다.
- 즉시 데이터 캐시와 라우터 캐시를 무효화하려면, revalidatePath 와 revalidateTag를 서버 액션에서 사용해야한다.

---

## APIs

아래의 테이블은 넥스트의 api들이 어떻게 캐시하는지 알려준다:

<table><thead><tr><th>API</th><th>Router Cache</th><th>Full Route Cache</th><th>Data Cache</th><th>React Cache</th></tr></thead><tbody><tr><td><a href="#link"><code>&lt;Link prefetch&gt;</code></a></td><td>Cache</td><td></td><td></td><td></td></tr><tr><td><a href="#routerprefetch"><code>router.prefetch</code></a></td><td>Cache</td><td></td><td></td><td></td></tr><tr><td><a href="#routerrefresh"><code>router.refresh</code></a></td><td>Revalidate</td><td></td><td></td><td></td></tr><tr><td><a href="#fetch"><code>fetch</code></a></td><td></td><td></td><td>Cache</td><td>Cache</td></tr><tr><td><a href="#fetch-optionscache"><code>fetch</code> <code>options.cache</code></a></td><td></td><td></td><td>Cache or Opt out</td><td></td></tr><tr><td><a href="#fetch-optionsnextrevalidate"><code>fetch</code> <code>options.next.revalidate</code></a></td><td></td><td>Revalidate</td><td>Revalidate</td><td></td></tr><tr><td><a href="#fetch-optionsnexttags-and-revalidatetag"><code>fetch</code> <code>options.next.tags</code></a></td><td></td><td>Cache</td><td>Cache</td><td></td></tr><tr><td><a href="#fetch-optionsnexttags-and-revalidatetag"><code>revalidateTag</code></a></td><td></td><td>Revalidate</td><td>Revalidate</td><td></td></tr><tr><td><a href="#revalidatepath"><code>revalidatePath</code></a></td><td></td><td>Revalidate</td><td>Revalidate</td><td></td></tr><tr><td><a href="#segment-config-options"><code>const revalidate</code></a></td><td></td><td>Revalidate or Opt out</td><td>Revalidate or Opt out</td><td></td></tr><tr><td><a href="#segment-config-options"><code>const dynamic</code></a></td><td></td><td>Cache or Opt out</td><td>Cache or Opt out</td><td></td></tr><tr><td><a href="#cookies"><code>cookies</code></a></td><td>Revalidate</td><td>Opt out</td><td></td><td></td></tr><tr><td><a href="#dynamic-functions"><code>headers</code>, <code>useSearchParams</code>, <code>searchParams</code></a></td><td></td><td>Opt out</td><td></td><td></td></tr><tr><td><a href="#generatestaticparams"><code>generateStaticParams</code></a></td><td></td><td>Cache</td><td></td><td></td></tr><tr><td><a href="#react-cache-function"><code>React.cache</code></a></td><td></td><td></td><td></td><td>Cache</td></tr><tr><td><a href="#unstable_cache"><code>unstable_cache</code></a> (Coming Soon)</td><td></td><td></td><td></td><td></td></tr></tbody></table>

### <Link>

기본적으로 <Link> 컴포넌트는 자동으로 풀 라우트 캐시에서 프리페치해 리액트 서버컴포넌트 페이로드를 라우터 캐시에 추가한다.

프리페칭을 비활성화하려면, prefetch 프랍을 false로 설정해라. 하지만, 이것은 캐시를 영구적으로 무시하지 않는데, 유저가 라우트를 방문했을 경우 라우트 세그먼트가 클라이언트 사이드에 캐시된다.

### router.prefetch

useRouter 훅의 prefetch 옵션은 라우트를 수동으로 프리페치하는데 사용된다. 이것은 리액트 서버컴포넌트 페이로드를 라우터 캐시에 추가한다.

### router.refresh

useRouter 훅의 refresh 옵션은 라우트를 수동으로 새로고침 하는데 사용된다. 이것은 완전히 라우터 캐시를 제거하고, 서버로 현재 라우트에 대한 새로운 요청을 만든다. refresh는 데이터나 풀 라우트 캐시에 영향을 끼치지 않는다.

리액트의 상태와 브라우저의 상태를 유지하며 렌더의 결과는 재조정된다.

### fetch

페치의 결과 데이터는 자동으로 데이터 캐시에 캐시된다.

```ts
// Cached by default. `force-cache` is the default option and can be ommitted.
fetch(`https://...`, { cache: "force-cache" });
```

### fetch options.cache

각 페치 요청의 데이터 캐싱을 캐시 옵션의 no-store를 통해 제외할 수 있다.

```ts
// Opt out of caching
fetch(`https://...`, { cache: "no-store" });
```

렌더 결과가 데이터에 의존하므로, cache: "no-store"는 페치 요청이 사용되는 라우트의 풀 라우트 캐시를 무시한다.
이것은, 라우트가 매 요청마다 렌더된다는 뜻이지만, 같은 라우트에 캐시된 데이터 페치를 둘 수 있다.

### fetch options.next.revalidate

페치에 next.revalidate를 사용해 각 페치 요청의 재검증 시간을 설정할 수 있다. 이것은 데이터 캐시를 재검증하고 풀 라우트 캐시를 재검증한다. 새로운 데이터가 페치되고, 컴포넌트가 서버에서 리렌더 된다.

```ts
// Revalidate at most after 1 hour
fetch(`https://...`, { next: { revalidate: 3600 } });
```

### fetch options.next.tags and revalidateTag

넥스트는 세분화된 데이터 캐싱 및 재검증을 위한 캐시 태깅 시스템 기능이 있다.

1. 페치 혹은 unstable_cache를 사용할 때, 하나 혹은 그 이상의 태그 캐시 엔트리를 설정할 수 있다.
2. 그 후, revalidateTag를 호출해 태그에 관련된 캐시 엔트리를 비운다.

예를 들어, 데이터를 페치할 때 태그를 설정할 수 있다.

```ts
// Cache data with a tag
fetch(`https://...`, { next: { tags: ["a", "b", "c"] } });
```

그 후, revalidateTag를 호출해 캐시 엔트리를 비운다.

```ts
// Revalidate entries with a specific tag
revalidateTag("a");
```

revalidateTag를 사용할 수 있는 두 장소가 존재한다.

1. 라우트 핸들러 - 써드 파티 이벤트의 응답으로 데이터 재검증. 라우터 핸들러가 특적 라우트에 속한 것이 아니기에, 라우터 캐시를 즉각적으로 무효화 하지 않는다.
2. 서버 액션 - 유저 액션 이후 데이터 재검증. 연관된 라우트의 라우터 캐시를 무효화한다.

### revalidatePath

revalidatePath는 수동으로 데이터를 재검증하고 특정 경로 아래의 라우트 새그먼트를 단일 작업으로 리렌더한다.
revalidatePath 메서드 호출은 데이터 캐시를 재검증하고 풀 라우트 캐시를 무효화한다.

```ts
revalidatePath("/");
```

revalidatePath를 사용할 수 있는 두 장소가 존재한다.

1. 라우트 핸들러 - 써드 파티 이벤트의 응답으로 데이터 재검증
2. 서버 액션 - 유저의 상호작용 이후 데이터 재검증

> revalidatePath vs router.refresh
> router.refresh 호출은 라우터 캐시를 비우고, 서버에서 데이터 캐시와 풀 라우트 캐시 무효화 없이 라우트 세그먼트를 리렌더한다.
>
> revalidatePath는 데이터 캐시와 풀 라우트 캐시를 비운다. router.refresh는 데이터 캐시와 풀 라우트 캐시가 클라이언트 사이드 api이기에 변경하지 않는다.

### Dynamic Functions

cookies, headers, useSearchParams, 그리고 searchParams은 모두 동적 함수로 런타임에 요청 정보에 의존한다.
이들의 사용은 라우트를 풀 라우트 캐시에서 제외한다. 즉 라우트가 동적으로 렌더된다.

### Segment Config Options

라우트 세그먼트 설정은 라우트 세그먼트의 기본 설정을 덮어쓰거나 페치를 사용하지 못하는 경우 사용된다.

아래의 라우트 세그먼트 옵션이 데이터 캐시와 풀 라우트 캐시에서 제외되게 한다.

- const dynamic = 'force-dynamic'
- const revalidate = 0

### generateStaticParams

동적 세그먼트의 경로가 generateStaticParams로 제공된다면, 빌드 시 풀 라우트 캐시에 캐시된다.
요청 시, 넥스트는 처음 방문 시 빌드 시 알려지지 않는 경로도 캐시한다.

export const dynamicParams = false를 사용해 요청시 캐시를 비활성화할 수 있다. 이 옵션이 사용되면, 오직 generateStaticParams로 제공된 경로만 보여지고, 다른 경로는 404혹은 캐치 올 라우트에 매치된다.

### React cache function

리액트의 캐시 함수는 함수의 반환 값을 메모이즈하게 해줘서 동일 함수를 한 번만 실행해 여러번 호출할 수 있게 해준다.

페치가 자동으로 메모이즈되기에, 리액트 캐시로 래핑할 필요 없다. 하지만, 페치 api가 적절하지 않은 경우 데이터 요청을 메모이즈할 수 있다. 예를 들어, 데이터베이스 클라이언트, cms 클라이언트 또는 그래프큐엘.

```ts
import { cache } from "react";
import db from "@/lib/db";

export const getItem = cache(async (id: string) => {
  const item = await db.item.findUnique({ id });
  return item;
});
```

### unstable_cache

unstable_cache는 실험적인 api로 페치 api가 적절하지 않은 경우 데이터 캐시에 값을 추가할 수 있게 해준다.

```ts
import { unstable_cache } from "next/cache";

export default async function Page() {
  const cachedData = await unstable_cache(
    async () => {
      const data = await db.query("...");
      return data;
    },
    ["cache-key"],
    {
      tags: ["a", "b", "c"],
      revalidate: 10,
    }
  )();
}
```
