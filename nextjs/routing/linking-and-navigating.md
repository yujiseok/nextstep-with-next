- [Linking and Navigating](#linking-and-navigating)
  - [ Component](#-component)
    - [Usage](#usage)
    - [Example: Linking to Dynamic Segments](#example-linking-to-dynamic-segments)
  - [`useRouter()` Hook](#userouter-hook)
  - [How Navigation Works](#how-navigation-works)
    - [Client-side Caching of Rendered Server Components](#client-side-caching-of-rendered-server-components)
    - [Invalidating the Cache](#invalidating-the-cache)
    - [Prefetching](#prefetching)
      - [Static and Dynamic Routes:](#static-and-dynamic-routes)
      - [Good to know:](#good-to-know)
    - [Hard Navigation](#hard-navigation)
    - [Soft Navigation](#soft-navigation)
    - [Conditions for Soft Navigation](#conditions-for-soft-navigation)
    - [Back/Forward Navigation](#backforward-navigation)
    - [Focus and Scroll Management](#focus-and-scroll-management)
  - [What I Learned](#what-i-learned)

# Linking and Navigating

넥스트 라우터는 서버 중심의 라우팅과 클라이언트 사이드 네비게이션을 이용합니다.
이것은 즉각적인 로딩 상태와 concurrent rendering을 지원합니다.
이것은 네비게이션은 클라이언트 상태로 유지되며, 고비용 리렌더를 피하고, 방해 가능하며, 레이스 컨디션을 유발하지 않습니다.

두 방식으로 이동할 수 있습니다.

- <Link> 컴포넌트
- `useRouter` 훅

두 방식을 어떻게 사용하는 지 다루고, 네비게이션이 어떻게 이루어지는 지 소개합니다.

## <Link> Component

<Link>는 html의 a 태그를 확장한 리액트 컴포넌트 입니다.
프리페칭과 클라이언트 사이드 네비게이션을 제공합니다. 
넥스트에서 주로 사용되는 이동 방식입니다.

### Usage

<Link>를 사용하기 위해선, `next/link`를 임포트합니다. 그리고 href 프랍을 컴포넌트에 넘겨줍니다.

```tsx
import Link from "next/link";

export default function Page() {
  return <Link href="/dashboard">Dashboard</Link>;
}
```

### Example: Linking to Dynamic Segments

동적 세그먼트에 링크할 경우, 템플릿 리터럴을 통해 링크를 생성할 수 있습니다.
예를 들어 블로그 포스트의 리스트를 생성할 경우

```tsx
import Link from "next/link";

export default function PostList({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>
          <Link href={`/blog/${post.slug}`}>{post.title}</Link>
        </li>
      ))}
    </ul>
  );
}
```

## `useRouter()` Hook

useRouter 훅은 클라이언트 컴포넌트에서 프로그래밍적으로 라우트를 교체합니다.

useRouter 훅을 사용하기 위해선, `next/navigation`에서 임포트 해옵니다.
그리고 클라이언트 컴포넌트 내부에서 훅을 호출합니다.

```tsx
"use client";

import { useRouter } from "next/navigation";

export default function Page() {
  const router = useRouter();

  return (
    <button type="button" onClick={() => router.push("/dashboard")}>
      Dashboard
    </button>
  );
}
```

useRouter 훅은 `push()`, `refresh()`와 같은 메서드를 제공합니다.

> Recommendation: 특별한 필요가 없을 경우 useRouter 대신 <Link> 컴포넌트를 사용하세요

## How Navigation Works

- <Link> 혹은 `router.push()` 호출을 통해 라우트의 변화가 시작됩니다.
- 라우터가 브라우저의 주소창에 URL을 업데이트합니다.
- 라우터는 클라이언트 사이드 캐시를 통해 변하지 않는 세그먼트를 재사용하여 불필요한 일을 피합니다. 이것은 부분 렌더링으로 참조 됩니다.
- 만약 소프트 네비게이션과 마주치면, 라우터는 서버 보단 캐시에서 새로운 세그먼트를 페치합니다. 아니라면, 라우터는 하드 네비게이션을 사용해 서버에서 서버 컴포넌트 페이로드를 페치합니다.
- 만약 로딩 UI가 생성 되었다면, 페이로드가 페치 될 동안 서버에서 보여집니다.
- 라우터는 캐시 또는 새로운 페이로드를 사용해 새로운 세그먼트를 클라이언트에 렌더링합니다.

### Client-side Caching of Rendered Server Components

> Good to know: 클라이언트 사이드 캐시는 서버 사이드의 넥스트 http 캐시와 다릅니다.

새로운 라우터는 서버 컴포넌트의 렌더링의 결과를 저장하는 인 메모리 클라이언트 사이드 캐시를 갖습니다. 캐시는 모든 레벨에서 무효화 되며 일관성 있는 동시 렌더를 보장합니다.

유저들이 앱을 이동할 때, 라우터는 이전에 페치된 세그먼트들의 페이로드를 저장하고 캐시의 세그먼트들을 프리페치합니다.

이것은, 라우터는 서버에 새로운 요청을 하는 대신 캐시를 재활용할 수 있다는 것을 뜻합니다. 이것은 리페칭과 리렌더링을 방지해 퍼포먼스를 향상합니다.

### Invalidating the Cache

`router.refresh()`를 사용해 새로고침 할 수 있습니다. 이것은 서버에 새로운 요청을 보내며, 데이터의 리페칭과 서버 컴포넌트를 리렌더하게 합니다.

### Prefetching

프리페칭은 방문 전 라우터를 미리 로드하는 것입니다.
프리페치된 결과는 라우터의 클라이언트 사이드 캐시에 추가됩니다.
이것은 프리페치된 라우트에 바로 이동할 수 있게 해줍니다.

기본적으로, <Link> 컴포넌트를 활용해 뷰포트에 보이는 라우트들은 프리페치 됩니다.
이것은 첫 로드시 혹은 스크롤 시 일어납니다.
또, 라우트는 `useRouter()` 훅의 `prefetch` 메서드를 사용해 프리페칭할 수 있습니다.

#### Static and Dynamic Routes:

- 만약 라우트가 정적이라면, 라우트 세그먼트를 위한 모든 서버 컴포넌트의 페이로드들은 프리페치됩니다.
- 라우트가 동적이라면, 첫 번째 `loading.js` 파일이 프리페치될 때까지 첫 번째로 공유된 레이아웃에서 페이로드들이 공유됩니다. 이것은 모든 라우트를 동적으로 프리페칭하는 비용을 절감하고 즉각적인 로딩 상태를 허용합니다.

#### Good to know:

- 프리페칭은 프로덕션에서만 가능합니다.
- `prefetch={false}`를 <Link> 컴포넌트에 추가혀먼 프리페칭은 사용할 수 없습니다.

### Hard Navigation

네비게이션에서, 캐시가 무효화 되며 서버가 데이터를 리페치하고 변경된 세그먼트드을을 리렌더링 하는 경우

### Soft Navigation

변경된 세그먼트들이 캐시를 통해 재사용되고 서버에 데이터를 새로 요청하지 않는 경우

### Conditions for Soft Navigation

네비게이션에서, 넥스트는 라우트가 프리페치 되었고 동적 세그먼트 또는 현재 라우트와 똑같은 동적 파라미터를 갖는 경우 소프트 네비게이션을 사용합니다.

예를 들어, `[team]`이라는 동적 세그먼트를 갖는 라우트가 있는 경우 `/dashboard/[team]/*`
`/dashboard/[team]/*`는 오직 `[team]` 파라미터가 변할 경우에만 캐시가 무효화 됩니다.

- `/dashboard/team-red/*`에서 `/dashboard/team-red/*`로 이동하는 것은 소프트 네비게이션입니다.
- `/dashboard/team-red/*`에서 `/dashboard/team-blue/*`로 이동하는 것은 하드 네비게이션입니다.

### Back/Forward Navigation

앞으로 가기 혹은 뒤로 가기는 소프트 네비게이션입니다. 클라이언트 사이드 캐시를 재사용하고 즉시 네비게이션합니다.

### Focus and Scroll Management

기본적으로, 넥스트는 세그먼트가 이동으로 변했을 경우 포커스를 설정하고 스크롤합니다.
이런 접근성과 가용한 기능은 advanced routing patterns 패턴이 개발되었을 때, 확연히 보입니다.

---

## What I Learned

넥스트가 네비게이션에 대해 그리고 재사용에 관해 어떤 철학을 가졌는 지 알 수 있었다.
항상 어떤식으로 프리페칭이 되는 지 궁금했는데, 아직 완벽하게 이해하지 못했지만 미약하게라도 이해한 것 같다.
또 리페칭과 리렌더링을 최대한 피해 퍼포먼스를 향상하는 하려는 노력을 알 수 있었다.

출처

- https://beta.nextjs.org/docs/routing/linking-and-navigating
