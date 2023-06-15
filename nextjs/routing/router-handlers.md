# Route Handlers

라우트 핸들러는 웹 요청과 응답 api를 사용하는 커스텀 요청 핸들러를 만들 수 있게 해준다.

![](https://nextjs.org/_next/image?url=/docs/dark/route-special-file.png&w=1920&q=75&dpl=dpl_2AbcivsvwTBMCkHCfxoskzAAr6Uq)

> Good to know: 라우트 핸들러는 app 디렉토리에서만 유효하다. 이들은 pages 디렉토리의 api 라우트와 같으며 동시에 사용할 필요 없다.

## Convention

라우트 핸들러는 app 디렉토리 내부에 route.js|ts 파일로 정의된다.

```ts
export async function GET(request: Request) {}
```

라우터 핸들러는 앱 디레토리에 page.js와 layout.js처럼 중첩될 수 있다. 하지만, page.js 파일과 동일한 레벨의 라우트 세그먼트에 존재할 수 없다.

### Supported HTTP Methods

http 메서드들을 지원한다:get, post, put, patch, delete, head 그리고 options
만약 지원 되지 않는 메서드를 호출한다면, 넥스트는 405 Method Not Allowed를 응답으로 리턴한다.

### Extended NextRequest and NextResponse APIs

내장된 요청과 응답을 지원하기 위해, 넥스트는 그들을 NextRequest와 NextResponse로 확장해 추가 상황에 편리한 헬퍼를 제공한다.

## Behavior

### Static Route Handlers

라우터 핸들러들은 get 메서드를 Response 객체와 사용했을 경우 정적으로 평가된다.

```tsx
import { NextResponse } from "next/server";

export async function GET() {
  const res = await fetch("https://data.mongodb-api.com/...", {
    headers: {
      "Content-Type": "application/json",
      "API-Key": process.env.DATA_API_KEY,
    },
  });
  const data = await res.json();

  return NextResponse.json({ data });
}
```

> TypeScript Warning: Response.json()도 가능하지만, 내장된 타입스크립트 타입이 에러를 보이므로 NextResponse.json()로 타입된 응답을 사용해라.

### Dynamic Route Handlers

라우트 핸들러가 동적으로 평가되는 경우

- Request 객체와 get 메서드를 사용
- 다른 http 메서드 사용
- cookies나 headers 같은 동적 함수 사용
- 세그먼트 설정 옵션이 동적 모드일 경우

```tsx
import { NextResponse } from "next/server";

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const id = searchParams.get("id");
  const res = await fetch(`https://data.mongodb-api.com/product/${id}`, {
    headers: {
      "Content-Type": "application/json",
      "API-Key": process.env.DATA_API_KEY,
    },
  });
  const product = await res.json();

  return NextResponse.json({ product });
}
```

post 메서드도 비슷하게, 라우트 핸들러를 동적으로 평가한다.

```tsx
import { NextResponse } from "next/server";

export async function POST() {
  const res = await fetch("https://data.mongodb-api.com/...", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "API-Key": process.env.DATA_API_KEY,
    },
    body: JSON.stringify({ time: new Date().toISOString() }),
  });

  const data = await res.json();

  return NextResponse.json(data);
}
```

> Good to know: 이전에, api 라우트는 폼의 제출과 같은 용도로 사용되었다. 라우트 핸들러는 이런 용도가 아니다. 우리는 준비가 되면 뮤테이션을 사용하는 것을 추천한다.

### Route Resolution

라우트를 가장 낮은 원시 레벨로 생각할 수 있다.

- 그들은 페이지와 같이 레이아웃이나 클라이언트 사이드 네비게이션에 관여하지 않음
- route.js와 page.js가 같은 라우트에 존재할 수 없음

| Page               | Route            | Result   |
| ------------------ | ---------------- | -------- |
| app/page.js        | app/route.js     | Conflict |
| app/page.js        | app/api/route.js | Valid    |
| app/[user]/page.js | app/api/route.js | Valid    |

```tsx
// app/page.js
export default function Page() {
  return <h1>Hello, Next.js!</h1>;
}

// ❌ Conflict
// `app/route.js`
export async function POST(request) {}
```

## Examples

이 예들은 라우트 핸들러를 넥스트의 다른 api, 기능들과 결합하는 법을 보여준다.

### Revalidating Static Data

당신은 정적 데이터를 next.revalidate 옵션을 통해 재평가할 수 있다.

```ts
import { NextResponse } from "next/server";

export async function GET() {
  const res = await fetch("https://data.mongodb-api.com/...", {
    next: { revalidate: 60 }, // Revalidate every 60 seconds
  });
  const data = await res.json();

  return NextResponse.json(data);
}
```

revalidate 세그먼트 옵션을 통해 사용할 수도 있다.

```ts
export const revalidate = 60;
```

### Dynamic Functions

라우트 핸들러는 넥스트의 동적 함수인 cookies나 headers와 함께 사용될 수 있다.

#### cookies

cookies를 통해 next/headers의 cookies를 읽을 수 있다. 이 서버 함수는 라우트 핸들러에서 직접 호출 될 수 있고, 다른 함수안에 중첩될 수 있다.

cookies 인스턴스는 읽기 전용이다. cookies를 설정하기 위해, Set-Cookie 헤더를 사용하는 new Response 리턴해야한다.

```ts
import { cookies } from "next/headers";

export async function GET(request: Request) {
  const cookieStore = cookies();
  const token = cookieStore.get("token");

  return new Response("Hello, Next.js!", {
    status: 200,
    headers: { "Set-Cookie": `token=${token}` },
  });
}
```

또는 웹 api 위에 있는 추상화를 통해 cookies(NextRequest)를 읽어올 수 있습니다.

```ts
import { type NextRequest } from "next/server";

export async function GET(request: NextRequest) {
  const token = request.cookies.get("token");
}
```

#### headers

headers를 통해 next/headers의 headers를 읽을 수 있다. 이 서버 함수는 라우트 핸들러에서 직접 호출 될 수 있고, 다른 함수안에 중첩될 수 있다.

headers 인스턴스는 읽기 전용이다. headers를 설정하기 위해, 새로운 headers를 new Response로 리턴해야한다.

```ts
import { headers } from "next/headers";

export async function GET(request: Request) {
  const headersList = headers();
  const referer = headersList.get("referer");

  return new Response("Hello, Next.js!", {
    status: 200,
    headers: { referer: referer },
  });
}
```

또는 웹 api 위에 있는 추상화를 통해 headers(NextRequest)를 읽어올 수 있습니다.

```ts
import { type NextRequest } from "next/server";

export async function GET(request: NextRequest) {
  const requestHeaders = new Headers(request.headers);
}
```

### Redirects

```ts
import { redirect } from "next/navigation";

export async function GET(request: Request) {
  redirect("https://nextjs.org/");
}
```

### Dynamic Route Segments

라우트 핸들러는 동적 새그먼트를 사용해 동적 데이터로부터 요청 핸들러를 생성할 수 있다.

```ts
export async function GET(
  request: Request,
  { params }: { params: { slug: string } }
) {
  const slug = params.slug; // 'a', 'b', or 'c'
}
```

| Route                     | Example URL | params        |
| ------------------------- | ----------- | ------------- |
| app/items/[slug]/route.js | /items/a    | { slug: 'a' } |
| app/items/[slug]/route.js | /items/b    | { slug: 'b' } |
| app/items/[slug]/route.js | /items/c    | { slug: 'c' } |

### Streaming

```ts
// https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream#convert_async_iterator_to_stream
function iteratorToStream(iterator: any) {
  return new ReadableStream({
    async pull(controller) {
      const { value, done } = await iterator.next();

      if (done) {
        controller.close();
      } else {
        controller.enqueue(value);
      }
    },
  });
}

function sleep(time: number) {
  return new Promise((resolve) => {
    setTimeout(resolve, time);
  });
}

const encoder = new TextEncoder();

async function* makeIterator() {
  yield encoder.encode("<p>One</p>");
  await sleep(200);
  yield encoder.encode("<p>Two</p>");
  await sleep(200);
  yield encoder.encode("<p>Three</p>");
}

export async function GET() {
  const iterator = makeIterator();
  const stream = iteratorToStream(iterator);

  return new Response(stream);
}
```

### Request Body

당신은 표준 웹 api 메서드를 사용해 Request 바디를 읽을 수 있다.

```ts
import { NextResponse } from "next/server";

export async function POST(request: Request) {
  const res = await request.json();
  return NextResponse.json({ res });
}
```

### CORS

표준 웹 api 메서드를 사용해 Response에 CORS를 설정할 수 있다.

```ts
export async function GET(request: Request) {
  return new Response("Hello, Next.js!", {
    status: 200,
    headers: {
      "Access-Control-Allow-Origin": "*",
      "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE, OPTIONS",
      "Access-Control-Allow-Headers": "Content-Type, Authorization",
    },
  });
}
```

### Edge and Node.js Runtimes

라우트 핸들러는 Edge와 Node.js 런타임에 동일한 웹 api 지원한다.
라우트 핸들러는 페이지 및 레이이아웃과 동일한 라우트 세그먼트 구성을 사용하므로 정적으로 재생성된 라우트 핸들러와 같은 오랫동안 기다려온 기능을 제공한다.

당신은 runtime 세그먼트를 사용해 특정 런타임을 설정할 수 있다.

```ts
export const runtime = "edge"; // 'nodejs' is the default
```

### Non-UI Responses

라우트 핸들러를 통해 non-UI 컨텐트를 리턴할 수 있다. 예를 들어 sitemap.xml, robots.txt, app icons 그리고 og 이미지

```ts
export async function GET() {
  return new Response(`<?xml version="1.0" encoding="UTF-8" ?>
<rss version="2.0">
 
<channel>
  <title>Next.js Documentation</title>
  <link>https://nextjs.org/docs</link>
  <description>The React Framework for the Web</description>
</channel>
 
</rss>`);
}
```

### Segment Config Options

라우트 핸들러는 페이지와 레이아웃에 사용되는 동일한 세그먼트 설정을 사용한다.

```ts
export const dynamic = "auto";
export const dynamicParams = true;
export const revalidate = false;
export const fetchCache = "auto";
export const runtime = "nodejs";
export const preferredRegion = "auto";
```

---

## What I Learned

라우트 핸들러를 사용해 http 메서드를 처리할 수 있어서 풀스택으로 사용할 수 있다는 점이 놀랍다.
