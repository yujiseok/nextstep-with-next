# Error Handling

error.js 파일 컨벤션은 중첩된 라우트에서 런타임 에러를 우아하게 다룰수 있게 해준다.

- 자동으로 라우트 세그먼트와 중첩된 자식을 리액트 에러 바운더리로 감싼다.
- 특정한 세그먼트와 맞는 에러 UI를 파일 시스템 계층에 맞게 만든다.
- 다른 앱 기능들에서 에러의 영향이 있는 세그먼트만 분리한다.
- 풀페이지 로드 없이 에러를 회복하려 시도한다.

라우트 세그먼트에 에러 UI를 error.js를 추가해 생성하고 리액트 컴포넌트를 내보낸다.

```tsx
"use client"; // Error components must be Client Components

import { useEffect } from "react";

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  useEffect(() => {
    // Log the error to an error reporting service
    console.error(error);
  }, [error]);

  return (
    <div>
      <h2>Something went wrong!</h2>
      <button
        onClick={
          // Attempt to recover by trying to re-render the segment
          () => reset()
        }
      >
        Try again
      </button>
    </div>
  );
}
```

## How error.js Works

- error.js는 자동으로 리액트 에러 바운더리를 만들고 중첩된 자식 세그먼트나 page.js 컴포넌트를 감싼다.
- error.js에서 내보내진 컴포넌트는 fallback 컴포넌트로 보여진다.
- 에러가 에러 바운더리로 던져지면, 에러는 포함되고, fallback 컴포넌트로 랜더된다.
- fallback 에러 컴포넌트가 활성화되면, 에러 바운더리 상위의 레이아웃들은 상태와 상호작용을 유지하고 에러 컴포넌트는 에러를 고치기 위한 함수를 보여준다.

### Recovering From Errors

때로 에러는 일시적일 수 있다. 이 경우, 간단히 재시도 하는 것이 이슈를 해결할 수 있다.

에러 컴포넌트는 reset() 함수를 사용해 유저가 에러를 고치도록한다.
호출 되면, 그 함수는 에러 바운더리의 컨텐츠를 리랜더한다.
만약 성공하면, fallback 에러 컴포넌트는 리랜더의 결과물로 대체된다.

```tsx
"use client";

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
```

## Nested Routes

특별한 파일에 의해 만들어진 리액트 컴포넌트들은 특별한 중첩 계층이 있다.

예를 들어, 중첩된 두 세그먼트가 layout.js와 error.js를 둘 다 갖는다면, 컴포넌트 계층은 다음과 같다.

```tsx
<Layout>
  <ErrorBoundary fallback={<Error />}>
    <Layout>
      <ErrorBoundary fallback={<Error />}>
        <page></page>
      </ErrorBoundary>
    </Layout>
  </ErrorBoundary>
</Layout>
```

중첩된 컴포넌트 계층은 중첩된 error.js의 동작에 영향을 끼친다.

- 에러들은 가장 가까운 부모 에러 바운더리로 올라간다. 이것은 error.js가 중첩된 모든 자식 새그먼트의 에러를 다룬다는 뜻이다. 다른 레벨의 폴더나 라우트의 많거나 적은 UI들은 error.js를 통해 모아진다.
- error.js는 레이아웃 컴포넌트에 중첩되었기에 동일한 세그먼트의 layout.js에 던져진 에러는 다루지 않는다.

## Handling Errors in Layouts

error.js 바운더리는 같은 세그먼트의 layout.js나 template.js가 던진 에러를 받지 못한다.
이런 의도된 계층은 에러가 발생했을 시 중요한 UI를 형제 라우트에 보이도록 하고 기능 역시 가능하도록 한다.

특정한 레이아웃 혹은 템플릿의 에러를 다루기 위해, error.js파일을 레이아웃의 부모 세그먼트에 둔다.

루트 레이아웃이나 템플릿의 에러를 다루기 위해, error.js를 global-error.js 컨벤션으로 사용한다.

## Handling Errors in Root Layouts

루트의 app/error.js 바운더리는 루트 레이아웃이나 템플릿 컴포넌트에서 던져진 에러를 잡지 못한다.

루트 컴포넌트에서 에러를 다루기 위해, error.js를 app/global-error.js로 불러서 루트 app 디렉터리에 위치시킨다.

error.js와 다르게 global-error.js 에러 바운더리는 전체 애플리케이션을 감싸고, 이것의 fallback 컴포넌트는 루트 레이아웃을 대체한다.
이 이유로 global-error.js는 반드시 html과 body태그를 명시해야한다.

global-error.js는 최소 에러 UI이고 모든 앱의 에러를 전부 잡는다. 루트 컴포넌트는 일반적으로 동적이지 않아서, 대부분의 에러는 error.js에서 다룬다.

global-error.js가 정의 되었어도, 루트에 전역으로 루트 레이아웃을 공유하고 브랜딩을 포함하는 error.js를 정의하는 것을 추천한다.

```tsx
"use client";

export default function GlobalError({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <html>
      <body>
        <h2>Something went wrong!</h2>
        <button onClick={() => reset()}>Try again</button>
      </body>
    </html>
  );
}
```

## Handling Server Errors

만약 데이터 페칭과 서버 컴포넌트에서 에러가 발생되면, 넥스트는 에러 객체를 가까운 error.js에 에러 프랍으로 전달한다.

넥스트 dev를 실행할 때 에러는 직렬화되어 서버 컴포넌트에서 클라이언트 error.js로 보내진다. 넥스트 start시 보안을 보장하기 위해 일반 에러 메세지가 해시가 포함된 .digest와 함께 에러를 전달한다. 이 해시는 서버 로그에 해당하는 곳에서 사용할 수 있다.

---

## What I Learned

에러 핸들링을 잘 못하는 나로서는 정말 최고의 기능이 아닐까 한다.
자동으로 에러를 잡아주고 그에 관한 UI를 보여줄 수 있다니 정말 편리하다.

출처

- https://nextjs.org/docs/app/building-your-application/routing/error-handling
