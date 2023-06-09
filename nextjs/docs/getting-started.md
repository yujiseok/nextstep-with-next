- [Getting Started](#getting-started)
  - [Introducing the App Router](#introducing-the-app-router)
  - [Features Overview](#features-overview)
  - [Thinking in Server Components](#thinking-in-server-components)
  - [What I Learned](#what-i-learned)

# Getting Started

## Introducing the App Router

넥스트 팀은 리액트의 서버 컴포넌트와 리액트 18 버전의 특징들을 넥스트와 통합하기 위해 노력했습니다.
이 새로운 특징들은 새로운 디렉터리인 `app`에서 사용이 가능합니다.

> App 라우터는 베타이므로 프로덕션에서 사용하지 마세요!

## Features Overview

|   Features    | What's New?                                                                                                            |
| :-----------: | ---------------------------------------------------------------------------------------------------------------------- |
|    Routing    | 새로운 파일 시스템 기반 라우터는 서버 컴포넌트의 상위에서 레이아웃, 중첩 라우팅, 로딩 상태, 에러 핸들링 등을 제공한다. |
|   Rendering   | 클라이언트 컴포넌트와 서버 컴포넌트를 통한 CSR과 SSR                                                                   |
| Data Fetching | 리액트 컴포넌트에 `async`/`await` 지원을 통한 간소화된 데이터 페칭 그리고 리액트와 웹 플랫폼에 맞는 `fetch()` API      |
|    Caching    | 새로운 HTTP Cache와 클라이언트 사이드 cache를 통한 서버 컴포넌트와 클라이언트 사이드 네비게이션 최적화                 |
| Optimizations | 네이티브 브라우저의 레이지 로딩으로 향상된 이미지 컴포넌트 자동으로 최적화 되는 새로운 폰트 모듈                       |
| Transpilation | 로컬 패키지 또는 `node_modules` 자동 트랜스파일 및 빌드                                                                |
|      API      | 새롭게 변경된 API 디자인                                                                                               |
|    Tooling    | 러스트로 만들어진 웹팩 보다 700배 빠른 Turbopack                                                                       |

## Thinking in Server Components

[리액트가 UI를 만드는 것에 대한 생각을 바꿔준 것](https://github.com/yujiseok/reacting-with-react/blob/main/reactdev/Learn/thinking-in-react.md)처럼, 리액트 서버 컴포넌트는 서버와 클라이언트를 활용한 하이브리드 애플리케이션에 **새로운 멘탈 모델**을 소개했습니다.

리액트의 **전체 애플리케이션**을 클라이언트에서 렌더링 하는 것이 아닌, 리액트는 어디서 컴포넌트를 렌더링 할지 선택할 수 있는 유연성을 제공합니다.

넥스트의 애플리케이션을 예를 들겠습니다.

![](https://assets.vercel.com/image/upload/f_auto%2Cq_100%2Cw_1600/v1667581343/nextjs-docs/darkmode/thinking-in-server-components.png)

우리가 만약 페이지를 작은 컴포넌트들로 나눈다면, 중요한 컴포넌트들은 상호 작용이 없으며 서버에서 렌더 된다는 사실을 인지합니다. 상호작용이 필요한 작은 조각의 UI의 경우 우리는 클라이언트 컴포넌트로 뿌려줄 수 있습니다. 이것은 넥스트의 서버 컴포넌트 우선적 접근과 맞습니다.

이 전환을 쉽게 하기 위해, `app` 디렉터리는 기본적으로 서버 컴포넌트 사용합니다. 즉 사용하기 위한 추가 과정이 필요 없습니다. 그리고 선택적으로 필요에 의해 클라이언트 컴포넌트를 사용할 수 있습니다.

---

## What I Learned

넥스트로 블로그를 만들었는데, 부족한 점도 많아서 넥스트를 좀 더 깊이 알기 위해 공부를 시작한다.
서버 컴포넌트와 클라이언트 컴포넌트에 관해 좀 더 깊게 알아봐야 할 것 같다.
확실히 서버 사이드 렌더링이라는 것은 멋진 기술 같다.

> 그리고 내부적으로 도대체 어떻게 파일 기반 라우팅을 처리하는지 알고 싶다 🧐

<br/>

출처

- https://beta.nextjs.org/docs/getting-started
