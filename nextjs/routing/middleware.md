# Middleware

미들웨어는 요청이 완료되기 전 코드를 실행할 수 있도록 해준다.
요청에 따라, 응답을 재설정, 리다이렉팅, 응답 또는 요청 헤더 수정 또는 직접 응답을 할 수 있다.

미들웨어는 컨텐츠 캐시와 라우트 매치전에 실행된다.

## Convention

middleware.ts|js 파일을 프로젝트 루트에 둬 미들웨어를 정의한다.
예를 들어, pages나 app 또는 src와 같은 레벨.

## Example

```ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

// This function can be marked `async` if using `await` inside
export function middleware(request: NextRequest) {
  return NextResponse.redirect(new URL("/home", request.url));
}

// See "Matching Paths" below to learn more
export const config = {
  matcher: "/about/:path*",
};
```

## Matching Paths

미들웨어는 프로젝트의 모든 라우트에서 호출된다. 다음 실행 순서에 따라:

1. headers from next.config.js
2. redirects from next.config.js
3. Middleware (rewrites, redirects, etc.)
4. beforeFiles (rewrites) from next.config.js
5. Filesystem routes (public/, \_next/static/, pages/, app/, etc.)
6. afterFiles (rewrites) from next.config.js
7. Dynamic Routes (/blog/[slug])
8. fallback (rewrites) from next.config.js

어떤 경로 미들웨어를 실행할지 두 방법이 있다.

1. Custom matcher config
2. Conditional statements

### Matcher

매쳐는 미들웨어가 특정 경로에서 실행되도록 한다.

```ts
export const config = {
  matcher: "/about/:path*",
};
```

당신은 하나 혹은 여러개의 경로를 배열로 매치할 수 있다.

```ts
export const config = {
  matcher: ["/about/:path*", "/dashboard/:path*"],
};
```

매쳐의 설정은 정규식을 허용해 부정적이거나 캐릭터 매칭이 지원된다.
특정 경로를 제외한 모든 경로와 일치하는 부정적인 미리 보기의 예는 다음과 같다.

```ts
export const config = {
  matcher: [
    /*
     * Match all request paths except for the ones starting with:
     * - api (API routes)
     * - _next/static (static files)
     * - _next/image (image optimization files)
     * - favicon.ico (favicon file)
     */
    "/((?!api|_next/static|_next/image|favicon.ico).*)",
  ],
};
```

> Good to know: 매쳐의 값은 상수로 빌드 시 분석된다. 동적 값은 무시된다.

Configured matchers:

1. 반드시 /로 시작
2. 명명된 파라미터를 포함할 수 있다: /about/:path는 /about/a와 /about/b와 일치하지만, /about/a/c는 아니다.
3. 명명된 파라미터에 모디파이어를 추가할 수 있다: /about/:path\*는 /about/a/b/c와 매치된다. \*는 0이거나 많게, ?는 0이거나 하나 +는 하나거나 더 많게
4. 소괄호와 정규식을 사용할 수 있다. `/about/(.*)`와 `/about/:path*`는 같다.

> Good to know: 이전 버전과 호환성을 위해, 넥스트는 /public을 /public/index로 간주한다. 그러므로 /public/:path와 매치한다.

### Conditional Statements

```ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  if (request.nextUrl.pathname.startsWith("/about")) {
    return NextResponse.rewrite(new URL("/about-2", request.url));
  }

  if (request.nextUrl.pathname.startsWith("/dashboard")) {
    return NextResponse.rewrite(new URL("/dashboard/user", request.url));
  }
}
```

## NextResponse

NextResponse api는 다음을 허용한다:

- `redirect` 들어오는 요청을 다른 URL로 redirect
- `rewrite` 받아온 응답을 rewrite
- api 라우트를 위한 요청 헤더 설정, getServerSideProps, 그리고 rewrite 목적지
- 응답 쿠키 설정
- 응답 헤더 설정

미들웨어에서 응답을 생성하기 위에:

1. rewrite 응답을 생성하게
2. NextResponse을 직접 리턴

## Using Cookies

쿠키는 보통의 헤더이다. Request에서 Cookie헤더로 분류된다. Response에서 그들은 Set-Cookie 헤더에 있다.
넥스트는 쿠키에 간편하게 접근하고 조작하기 위해 NextRequest와 NextResponse의 익스텐션 cookies를 제공한다.

1. 들어오는 요청시, cookies는 get, getAll, set 그리고 delete 메서드를 갖는다. 또한 쿠키가 존재하는지 확인하는 has와 clear로 모든 쿠키를 제거할 수 있다.
2. 응답의 경어, cookies는 get, getAll, set 그리고 delete 메서드를 갖는다.

```ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  // Assume a "Cookie:nextjs=fast" header to be present on the incoming request
  // Getting cookies from the request using the `RequestCookies` API
  let cookie = request.cookies.get("nextjs");
  console.log(cookie); // => { name: 'nextjs', value: 'fast', Path: '/' }
  const allCookies = request.cookies.getAll();
  console.log(allCookies); // => [{ name: 'nextjs', value: 'fast' }]

  request.cookies.has("nextjs"); // => true
  request.cookies.delete("nextjs");
  request.cookies.has("nextjs"); // => false

  // Setting cookies on the response using the `ResponseCookies` API
  const response = NextResponse.next();
  response.cookies.set("vercel", "fast");
  response.cookies.set({
    name: "vercel",
    value: "fast",
    path: "/",
  });
  cookie = response.cookies.get("vercel");
  console.log(cookie); // => { name: 'vercel', value: 'fast', Path: '/' }
  // The outgoing response will have a `Set-Cookie:vercel=fast;path=/test` header.

  return response;
}
```

## Setting Headers

NextResponse를 api를 활용해 요청 응답 헤더를 설정할 수 있다.

```ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  // Clone the request headers and set a new header `x-hello-from-middleware1`
  const requestHeaders = new Headers(request.headers);
  requestHeaders.set("x-hello-from-middleware1", "hello");

  // You can also set request headers in NextResponse.rewrite
  const response = NextResponse.next({
    request: {
      // New request headers
      headers: requestHeaders,
    },
  });

  // Set a new response header `x-hello-from-middleware2`
  response.headers.set("x-hello-from-middleware2", "hello");
  return response;
}
```

> Good to Know: 헤더를 크게 설정하는 것은 431에러를 유발한다.

## Producing a Response

미들웨어의 응답에 따라 Response 또는 NextResponse를 직접 리턴할 수 있다.

```ts
import { NextRequest, NextResponse } from "next/server";
import { isAuthenticated } from "@lib/auth";

// Limit the middleware to paths starting with `/api/`
export const config = {
  matcher: "/api/:function*",
};

export function middleware(request: NextRequest) {
  // Call our authentication function to check the request
  if (!isAuthenticated(request)) {
    // Respond with JSON indicating an error message
    return new NextResponse(
      JSON.stringify({ success: false, message: "authentication failed" }),
      { status: 401, headers: { "content-type": "application/json" } }
    );
  }
}
```

## Advanced Middleware Flags

---

## What I Learned

라우트 핸들러와 미들웨어 부분은 솔직히 잘이해가 안간다. 좀 더 읽어보고 내용을 추가해야겠다.
