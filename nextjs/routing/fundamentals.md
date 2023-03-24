- [Routing Fundamentals](#routing-fundamentals)
  - [Terminology](#terminology)
  - [The `app` Directory](#the-app-directory)
  - [Folders and Files inside `app`](#folders-and-files-inside-app)
  - [Route Segments](#route-segments)
  - [Nested Routes](#nested-routes)
  - [File Conventions](#file-conventions)
  - [Component Hierarchy](#component-hierarchy)
  - [Colocation](#colocation)
  - [Server-Centric Routing with Client-side Navigation](#server-centric-routing-with-client-side-navigation)
  - [Partial Rendering](#partial-rendering)
  - [Advanced Routing Patterns](#advanced-routing-patterns)
  - [What I Learned](#what-i-learned)

# Routing Fundamentals

넥스트 13은 리액트의 서버 컴포넌트(레이아웃, 중첩 라우팅, 로딩 상태, 에러 핸들링 등이 포함된)로 구성된 새로운 **App Router**를 소개합니다.

## Terminology

아마 문서 내에서 아래와 같은 용어를 계속 보게 될 것입니다.

![](https://assets.vercel.com/image/upload/f_auto%2Cq_100%2Cw_1600/v1666568302/nextjs-docs/darkmode/terminology-component-tree.png)

- Tree: 계층을 보여주기 위한 컨벤션. 예를 들어 부모 자식 컴포넌트가 있는 컴포넌트 트리, 폴더 구조, etc
- Subtree: 새로운 루트에서 시작하고 leaves에서 끝나는 tree의 한 부분
- Root: tree와 subtree의 첫 노드, 예를 들면 루트 레이아웃
- Leaf: 자식이 없는 subtree의 노드들로, URL 경로의 마지막 부분

![](https://assets.vercel.com/image/upload/f_auto%2Cq_100%2Cw_1600/v1666568301/nextjs-docs/darkmode/terminology-url-anatomy.png)

- URL Segment: URL 경로의 일부분 /로 구분되는
- URL Path: 도메인 뒤에 따라오는 URL 부분

## The `app` Directory

새로운 App Router는 `app` 디렉터리와 사용됩니다. `app` 디렉터리는 `pages` 디렉터리와 같이 동작하며, 점진적인 체택을 허용합니다. 이것은 새로운 역할들을 하는 라우트를 두고 기존의 `pages`에 이전의 역할들을 하는 라우트를 보존합니다.

> 디렉터리간 라우터들을 가로지르는 것은 빌드 타임 에러를 유발합니다.

![](https://assets.vercel.com/image/upload/f_auto%2Cq_100%2Cw_1600/v1666387689/nextjs-docs/darkmode/app-folder-page-folder.png)

기본적으로, `app`안의 컴포넌트들은 서버 컴포넌트를 기본을 합니다. 이것은 성능 최적화이고 그것들을 쉽게 채택할 수 있게 해줍니다. 물로 클라이언트 컴포넌트도 사용할 수 있습니다.

## Folders and Files inside `app`

`app` 디렉터리 안:

- Folders는 라우트로 정의됩니다. 계층 구조에 따라 루트 폴더부터 리프 폴더까지 아래로 흐르는 `page.js`를 포함하는 중첩된 폴더들의 단일 경로입니다.
- Files는 UI를 생성하는 것으로 라우트의 segment에 보입니다.

## Route Segments

라우트 안에 있는 각각의 폴더는 route segment로 표현됩니다. 각 route segment는 상응하는 URL segment에 매핑됩니다.

![](https://assets.vercel.com/image/upload/f_auto%2Cq_100%2Cw_1600/v1666568300/nextjs-docs/darkmode/route-segments-to-path-segments.png)

## Nested Routes

중첩된 라우트를 만들기 위해, 폴더들을 중첩해서 생성하면 됩니다. 예를 들어, `app` 디렉터리에 `/dashboard/settings`와 같이 중첩된 폴더를 생성할 수 있습니다.

`/dashboard/settings` 라우트는 총 3개의 segments로 구성되어 있습니다.

- / 루트
- dashboard (Segment)
- settings (Leaf segment)

## File Conventions

넥스트는 특별한 역할을 수행하는 파일들을 정해 놓았습니다.

- page.js: 라우트의 UI를 만들고 경로를 접근할 수 있도록 합니다.
  - route.js: server-side API 엔드포인트들을 생성합니다.
- layout.js: 공통의 UI를 생성합니다. 레이아웃은 페이지 혹은 자식 segment를 감쌉니다.
  - layout.js와 비슷한 파일로 새로운 컴포넌트 인스턴스가 네비게이션에 마운트 되는 것을 제외한, 이 동작이 필요하지 않으면 레이아웃 사용
- loading.js: segment와 자식들에 로딩 UI를 만듭니다. 페이지와 자식을 리액트 서스펜스로 감쌉니다.
- error.js: 에러 UI를 보여줍니다. 페이지와 자식을 리액트 에러 바운더리로 감쌉니다. 에러가 잡히면 에러를 보여줍니다.
  - global-error.js: 루트의 layout.js의 에러를 잡습니다.
- not-found.js: not found 라우트용 UI를 보여줍니다.

## Component Hierarchy

특별한 파일들은 특정한 계층에 따라 렌더됩니다.

- layout.js
- template.js
- error.js
- loading.js
- not-found.js
- page.js 또는 중첩된 layout.js

![](https://assets.vercel.com/image/upload/f_auto%2Cq_100%2Cw_1600/v1675248777/nextjs-docs/darkmode/file-conventions-component-hierarchy.png)

중첩된 라우트의 경우 부모 컴포넌트의 안에 중첩됩니다.

![](https://assets.vercel.com/image/upload/f_auto%2Cq_100%2Cw_1600/v1675248778/nextjs-docs/darkmode/nested-file-conventions-component-hierarchy.png)

## Colocation

.을 이용해 추가적인 파일들을 생성할 수 있습니다.

![](https://assets.vercel.com/image/upload/f_auto%2Cq_100%2Cw_1600/v1666568300/nextjs-docs/darkmode/collocating-assets-in-the-app-directory.png)

## Server-Centric Routing with Client-side Navigation

클라이언트 사이드 라우팅을 사용하는 `pages` 디렉터리와 다르게, `app` 디렉터리는 서버 컴포넌트와 서버에서의 데이터 페칭을 결합하여 서버 중심의 라우팅을 사용합니다.
서버 중심의 라우팅에서, 클라이언트는 라우트 맵을 다운로드할 필요가 없으며, 서버 컴포넌트가 라우트를 찾는데 사용됩니다. 이런 최적화는 모든 애플리케이션에 유용합니다. 그러나 라우터가 많은 애플리케이션일수록 효과가 있습니다.

비록 라우팅은 서버 중심이지만, 라우터는 Link 컴포넌트로 클라이언트 사이드 네비게이션을 사용할 수 있습니다. 즉 새로운 라우트로 이동했을 때 전체 페이지(SPA 처럼)를 리로드 하지 않아도 됩니다.
대신 URL이 업데이트되고 넥스트는 바뀐 segments들만 렌더 합니다.

게다가, 유저가 앱을 돌아다닐 때, 라우터는 리액트 컴포넌트의 페이로드들의 결과를 클라이언트 사이드 캐시에 저장합니다. 캐시는 라우트 별로 쪼개지며, 어떤 레벨에서도 무효화되며 동시 렌더 중 일관성을 보장합니다.
이말은 즉 이전의 캐시가 재사용 될 수 있음을 뜻하며, 퍼포먼스를 향상시킵니다.

## Partial Rendering

형제 라우트를 네비게이트 할 때 넥스트는 라우트가 바뀔 때만 레이아웃을 페치하고 렌더 합니다. subtree 위의 어떠한 segment도 리페치하거나 리렌더하지 않습니다. 즉 레이아웃을 공유하고, 그 레이아웃은 형제 페이지들을 네비게이트 할 때 보존된다는 것입니다.

![](https://assets.vercel.com/image/upload/f_auto%2Cq_100%2Cw_1600/v1671641891/nextjs-docs/darkmode/partial-rendering.png)

이런 부분 렌더링이 없을 경우 모든 네비게이션에서 full page 리렌더가 일어날 것입니다. segment 리렌더는 데이터 전송의 양을 줄이며, 실행 시간을 줄이고 퍼포먼스 향상을 일으킵니다.

## Advanced Routing Patterns

미래에 넥스트는 좀 더 발전된 라우팅 패턴을 제공할 것입니다.

- Parallel Routes: 동시에 두 가지 혹은 그 이상의 페이지를 보여주는 라우트로, 독립적으로 네비게이트될 수 있습니다. 분할된 화면에서 활용할 수 있습니다.
- Intercepting Routes: 라우트를 가로채 다른 경로의 컨텍스트를 표시합니다. 현재 페이지의 컨텍스트를 유지하는 것이 중요할 때 사용할 수 있습니다.
- Conditional Routes: 조건에 따라 렌더하는 조건부 라우트입니다.

## What I Learned

넥스트를 사용하며 제일 편했던 부분이 바로 이 라우팅이었는데 아무리 생각해도 엄청난 기술인 것 같다.
중첩 라우팅을 쉽게 하는 부분과 서스펜스를 활용해 로딩 UI를 보여주는 것이 매우 유용하다고 생각한다.

그리고 넥스트가 얼마나 퍼포먼스적으로 최적화하고 향상시키기 위해(Partial Rendering
, Link 컴포넌트) 노력했는지 알 수 있는 부분 같다.

내부적으로 어떤 로직으로 돌아가는지 알고 싶다.
