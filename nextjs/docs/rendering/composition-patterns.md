# Server and Client Composition Patterns

리액트 애플리케이션을 만들 때, 어떤 부분이 클라이언트 또는 서버에서 렌더될지 고려해야한다.

---

## When to use Server and Client Components?

서버와 클라이언트 컴포넌트의 다른 사용 예의 간략한 요약이다.

![](https://github.com/yujiseok/nextstep-with-next/assets/83855636/da7fa7b8-1437-40b5-b941-ca0f974ab84d")

---

## Server Component Patterns

클라이언트 사이드 렌더링을 택하기 전에, 서버에서 데이터 페칭, 데이터베이스 혹은 백엔드 서비스에 접근하고 싶을 것이다.

서버 컴포넌트의 일반적인 사용 패턴이 있다:

### Sharing data between components

서버에서 데이터를 페치할 시, 다른 컴포넌트에서 데이터를 사용할 경우가 있다. 예를 들어, 레이아웃과 페이지가 동일한 데이터에 의존하는.

리액트 컨텍스트 사용과 데이터를 프랍으로 전달하는 것 대신, `fetch` 혹은 리액트의 `cache` 함수를 사용해 동일한 데이터를 필요로하는 컴포넌트에서 중복 상관없이 사용할 수 있다. 이것은 리액트가 `fetch`를 확장해 자동으로 데이터 요청을 메모이즈하고 `fetch`가 불가능할 경우 `cache` 함수를 사용할 수 있다.

### Keeping Server-only Code out of the Client Environment

자바스크립트 모듈이 서버와 클라이언트 컴포넌트 모듈에서 공유될 수 있기에, 서버에서만 실행되도록 의도한 코드가 클라이언트에 몰래 들어가는 것이 가능하다.

예를 들어, 데이터 페칭 함수가 있다:

```jsx
export async function getData() {
  const res = await fetch("https://external-service.com/data", {
    headers: {
      authorization: process.env.API_KEY,
    },
  });

  return res.json();
}
```

첫 눈에는, 함수가 서버와 클라이언트 두 환경에서 모드 동작하는 것으로 보인다. 하지만, 이 함수는 서버에서만 동작하도록 의도된 `API_KEY`를 포함한다.

환경 변수 `API_KEY`가 `NEXT_PUBLIC` 프리픽스가 없기에, 이것은 프라이빗한 변수로 서버에서만 접근 가능하다. 클라이언트에 환경변수가 노출되는 것을 막기위해, 넥스트는 빈 문자열로 변경한다.

함수가 클라이언트에서 임포트 되어도 의도대로 동작하지 않는다. 변수를 공개하면 클라이언트에서 함수가 작동하지만, 민감한 정보를 클라이언트에 노출시키고 싶지 않을 수 있다.

이런 의도되지 않은 서버 코드의 클라이언트 사용을 막기위해 `server-only` 패키지를 사용할 수 있다. 클라이언트 모듈에 임포트할 때, 빌드 시 에러를 알려준다.

```bash
npm install server-only
```

서버에서만 사용되는 코드에 불러와 사용한다.

```js
import "server-only";

export async function getData() {
  const res = await fetch("https://external-service.com/data", {
    headers: {
      authorization: process.env.API_KEY,
    },
  });

  return res.json();
}
```

이제 어떤 클라이언트 컴포넌트든 `getData()`를 불러올 경우, 빌드 시 에러를 내고 오직 서버에서 사용된다고 알려준다.

이와 상응하는 `client-only`는 오직 클라이언트에서만 사용된다.

### Interleaving Server and Client Components

클라이언트와 서버 컴포넌트를 배치할 때, UI를 컴포넌트 트리로 시각화하는 것이 도움이 된다.
서버 컴포넌트인 루트 레이아웃에서 시작해, `"use client"`를 사용하는 특정 하위트리의 클라이언트 컴포넌트를 렌더할 수 있다.

이런 클라이언트 트리와, 여전히 서버 컴포넌트를 중첩할 수 있으며 서버 액션을 호출할 수 있지만 명심해야할 것들이 있다:

- 요청-응답 생명주기에서, 코드는 서버에서 클라이언트로 이동한다. 만약 클라이언트에 있을 때, 서버에 있는 데이터 또는 자원에 접근하려고 하면 서버로 새로운 요청을 보내게된다.
- 새로운 서버로의 요청이 생겼을 경우, 모든 서버 컴포넌트가 먼저 렌더되는데, 클라이언트 내부에 중첩된 컴포넌트도 포함한다. 렌더의 결과는 클라이언트 컴포넌트의 위치의 참조값을 포함한다. 그 후, 서버에서 리액트는 RSC 페이로드를 사용해 서버와 클라이언트 컴포넌트를 하나의 트리로 재조정한다.
- 클라이언트 컴포넌트가 서버 컴포넌트 후 렌더 되었을 시, 서버 컴포넌트를 클라이언트 모듈로 가져올 수 없다. 대신, 서버 컴포넌트를 프랍으로 클라이언트 컴포넌트에 넘길 수 있다.

### Unsupported Pattern: Importing Server Components into Client Components

이 패턴은 지원되지 않는다. 서버 컴포넌트를 클라이언트 컴포넌트 내부에서 가져올 수 없다.

```tsx
"use client";

// You cannot import a Server Component into a Client Component.
import ServerComponent from "./Server-Component";

export default function ClientComponent({
  children,
}: {
  children: React.ReactNode;
}) {
  const [count, setCount] = useState(0);

  return (
    <>
      <button onClick={() => setCount(count + 1)}>{count}</button>

      <ServerComponent />
    </>
  );
}
```

### Supported Pattern: Passing Server Components to Client Components as Props

이 패턴은 지원된다. 서버 컴포넌트를 클라이언트 컴포넌트의 프랍으로 전달할 수 있다.

일반적인 패턴은 리액트의 `children` 프랍을 사용해 클라이언트 컴포넌트에 "slot"을 만드는 것이다.

예에서, 클라이언트컴포넌트는 자식 프랍을 받는다.

```tsx
"use client";

import { useState } from "react";

export default function ClientComponent({
  children,
}: {
  children: React.ReactNode;
}) {
  const [count, setCount] = useState(0);

  return (
    <>
      <button onClick={() => setCount(count + 1)}>{count}</button>
      {children}
    </>
  );
}
```

클라이언트컴포넌트는 자식이 서버 컴포넌트의 결과로 채워진다는 것을 알지 못한다. 클라이언트컴포넌의 역할은 오직 자식이 어디에 올지를 정하는 것이다.

부모 서버 컴포넌트에서, 클라이언트컴포넌트와 서버컴포넌트를 가져올 수 있으며 서버컴포넌트를 클라이언트컴포넌트의 자식으로 전달할 수 있다.

```tsx
// This pattern works:
// You can pass a Server Component as a child or prop of a
// Client Component.
import ClientComponent from "./client-component";
import ServerComponent from "./server-component";

// Pages in Next.js are Server Components by default
export default function Page() {
  return (
    <ClientComponent>
      <ServerComponent />
    </ClientComponent>
  );
}
```

이런 접근으로, 클라이언트컴포넌트와 서버컴포넌트는 분리되고 독립적으로 렌더된다.
이 예에서, 자식 서버컴포넌트는 클라이언트컴포넌트가 클라이언트에서 렌더되기전 서버에서 렌더된다.

> 굿 투 노:
>
> - 컨텐츠를 올리는 것은 부모 컴포넌트가 리렌더될 시 중첩된 자식 컴포넌트의 리렌더를 피하기위해 사용되었다.
> - 자식 프랍에 한정되지 않고 JSX에 어떤 프랍이든 사용 가능하다.
