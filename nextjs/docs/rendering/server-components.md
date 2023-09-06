# Server Components

리액트 서버 컴포넌트는 UI가 렌더되고 서버에서 선택적으로 캐시될 수 있도록 해준다. 넥스트에서, 렌더링은 라우트 새그먼트에 의해 나뉘어 스트리밍과 부분 렌더 그리고 3개의 다른 렌더 전략이 있다.

- 정적 렌더링
- 동적 렌더링
- 스트리밍

---

## [Benefits of Server Rendering](https://nextjs.org/docs/app/building-your-application/rendering/server-components#benefits-of-server-rendering)

서버 렌더링의 장점은 다음과 같다.

- 데이터 페칭: 서버 컴포넌트는 데이터 페치를 서버로 옮겨주고, 데이터 자원에 가깝게 해준다. 이것은 클라이언트 요청이 만들어 내는 양과 렌더에 필요한 시간을 줄여준다.
- 보안: 서버 컴포넌트는 민감한 데이터와 로직을 서버에 두도록 하는데, 토큰이나 api 키들을 클라이언트에 노출되지 않도록 한다.
- 캐싱: 서버에서 렌더링을 통해, 결과가 캐시되고 후속 요청에 재사용된다. 이것은 렌더링과 각 요청에 데이터를 페칭하는 양을 줄여준다.
- 번들 사이즈: 서버 컴포넌트는 클라이언트 자바스크립트 번들 사이즈에 영향을 주는 큰 디펜던시를 유지할 수 있다. 이것은 느린 인터넷 또는 좋지 못한 장치에서 유용한데, 클라이언트가 다운로드할 필요 없이, 서버 컴포넌트를 위한 자바스크립트를 실행하고 파싱한다.
- **Initial Page Load and [First Contentful Paint (FCP)](https://web.dev/fcp/):** 서버에서 html을 생성해 유저가 즉각적으로 페이지를 볼 수 있다. 클라이언트의 다운로드를 기다릴 필요 없이 자바스크립트를 파싱하고 실행한다.
- seo: 렌더된 html은 검색엔진 봇에 의에 인덱싱 되고 소셜 네트워크 봇들이 프리뷰 카드를 만들게 한다.
- 스트리밍: 서버 컴포넌트는 렌더링 작업을 청크단위로 나누고 준비 되는 대로 클라이언트로 스트리밍한다. 이것은 유저가 전체 페이지가 서버에서 렌더되는 것을 기다리는 것이 아닌 페이지의 일부를 먼저 볼 수 있게 한다.

---

### [Using Server Components in Next.js](https://nextjs.org/docs/app/building-your-application/rendering/server-components#using-server-components-in-nextjs)

기본적으로, 넥스트는 서버 컴포넌트를 사용하는데, 이것은 어떤 설정 없이 자동 서버 렌더링이 구현되며, 클라이언트로 사용이 필요할 때 클라이언트 컴포넌트로 선택할 수 있다.

---

### [How are Server Components rendered?](https://nextjs.org/docs/app/building-your-application/rendering/server-components#how-are-server-components-rendered)

서버에서, 넥스트는 리액트의 api를 통해 렌더링을 조율한다. 렌더 작업은 청크로 나뉜다: 각 라우트 새그먼트와 서스펜스

각 청크는 두 단계로 렌더 된다:

1. 리액트는 서버 컴포넌트를 특별한 데이터 포맷인 리액트 서버 페이로드로 렌더한다.
2. 넥스트는 rsc 페이로드와 클라이언트 컴포넌트는 자바스크립트를 사용해 html을 서버에 렌더한다.

그 후 클라이언트에서:

1. html은 즉각적으로 라우트의 상호 작용 없는 프리뷰로 사용된다. - 첫 페이지 로드 온리
2. 리액트 서버 컴포넌트 페이로드는 클라이언트와 서버 컴포넌트 트리를 조정하고 돔을 변경한다.
3. 자바스크립트는 클라이언트 컴포넌트의 하이드레이트에 사용되며 애플리케이션의 상호작용을 가능하게 한다.

> 리액트 서버 컴포넌트 페이로드란?
>
> 리액트 서버 컴포넌트 페이로드는 리액트 서버 컴포넌트가 렌더된 트리의 간결한 바이너리 표현이다. 이것은 리액트가 클라이언트 브라우저의 돔을 변경할 때 사용된다. RSC Payload는 다음을 포함한다:
>
> - 서버 컴포넌트의 렌더 결과
> - 클라이언트 컴포넌트를 렌더링해야하는 위치 및 해당 자바스크립트 파일에 대한 참조에 대한 플레이스홀더
> - 서버 컴포넌트에서 클라이언트 컴포넌트로 전달되는 모든 프랍들

---

### [Server Rendering Strategies](https://nextjs.org/docs/app/building-your-application/rendering#server-rendering-strategies)

3 가지 서버 렌더링의 하위 집합이 있다: 정적, 동적, 그리고 스트리밍

[Static Rendering (Default)](https://nextjs.org/docs/app/building-your-application/rendering#static-rendering-default)

정적 렌더링은, 라우트가 빌드 시 , 데이터 재검증 시 렌더 된다. 결과는 캐시되고 cdn에 푸쉬된다. 이 최적화는 렌더 작업의 결과를 유저와 서버 요청 사이에 공유할 수 있게 한다.

정적 렌더링은 데이터가 개인화 되지 않고 빌드 타임시 알게되는, 정적 블로그 포스트나 제품 페이지에 유용하다.

[Dynamic Rendering](https://nextjs.org/docs/app/building-your-application/rendering#dynamic-rendering)

동적 렌더링은, 각 유저의 요청시에 라우트가 렌더된다.

동적 라우팅은 개인화된 정보나 요청 시에만 정보를 알 수 있는, 쿠키나 url 서치 파람스를 사용하는 경우에 유용하다.

> 동적 라우트와 캐시된 데이터
>
> 대부분의 웹사이트에서, 라우트는 완전히 정적이거나 동적일 수 없고 스펙트럼이다. 예를 들어, 이커머스 페이지가 있을 때, 인터벌에 의해 재검증된 캐시된 제품 데이터와 캐시되지 않은 개인화된 소비자 데이터를 가질 수 있다.
>
> 넥스트에서, 캐시되고 캐시되지 않은 데이터를 갖는 동적 렌더 라우트를 가질 수 있다. 이것은 RSC Payload와 데이터가 서로 다르게 캐시되기 때문이다. 이것은 동적 렌더의 요청시 마다 성능에 문제를 일으키는 페치를 걱정하지 않아도 되게 해준다.

[Switching to Dynamic Rendering](https://nextjs.org/docs/app/building-your-application/rendering#switching-to-dynamic-rendering)

렌더 중, 동적 함수나 캐시되지 않은 데이터 요청이 발견되면, 넥스트는 전체 라우트를 동적 렌더링으로 교체한다. 테이블은 동적 함수와 데이터 캐싱이 정적 혹은 동적 렌더링에 어떻게 영향을 미치는지 정리한다.

![Untitled](https://github.com/yujiseok/nextstep-with-next/assets/83855636/805c4b46-d459-4842-ae5a-f990df627840)

완전히 정적이려면, 데이터는 반드시 캐시되어야 한다. 하지만, 캐시와 캐시되지 않은 데이터를 사용해 동적 렌더를 가질 수 있다.

개발자로서, 정적과 동적을 고를 필요 없다. 넥스트가 자동으로 최적의 렌더 전략을 기능과 api를 기반으로 선택해준다. 대신에, 언제 캐시하고 특정 데이터를 재검증 할지, ui의 어떤 부분을 스트림할지 정해야한다.

[Dynamic Functions](https://nextjs.org/docs/app/building-your-application/rendering#dynamic-functions)

동적 함수는 요정 시에만 알 수 있는 정보 예를 들어, 유저 쿠키, 현재 요청 헤더, 또는 url 서치 파람 등 에 달려있다. 넥스트의 동적 함수는 다음과 같다:

- `[cookies()](https://nextjs.org/docs/app/api-reference/functions/cookies)` and `[headers()](https://nextjs.org/docs/app/api-reference/functions/headers)`: 서버 컴포넌트에서의 사용은 전체 라우트를 요청 시 동적 렌더한다.
- `[useSearchParams()](https://nextjs.org/docs/app/api-reference/functions/use-search-params)`:
  - 클라이언트 컴포넌트에서, 정적 렌더를 무시하고 부모 서스펜스에 가까운 모든 클라이언트 컴포넌트를 렌더한다.
  - `[useSearchParams()](https://nextjs.org/docs/app/api-reference/functions/use-search-params)`을 사용하는 클라이언트 컴포넌트를 서스펜스로 감싸는 것을 추천한다. 이것은 클라이언트 컴포넌트를 정적 렌더할 수 있도록 해준다.
- `[searchParams](https://nextjs.org/docs/app/api-reference/file-conventions/page#searchparams-optional)`: 페이지의 프랍을 사용하면 요청 시 페이지가 동적 렌더로 선택된다.

위의 어떤 함수를 쓰던, 요청 시에 전체 라우트를 동적 렌더한다.

[Streaming](https://nextjs.org/docs/app/building-your-application/rendering#streaming)

![Untitled](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Fsequential-parallel-data-fetching.png&w=3840&q=75&dpl=dpl_GGqC9f3YC9X7o2mZgeZfBtLubEXa)

스트리밍을 통해, 라우트는 요청시 서버에 렌더된다. 이 작업은 청크로 나누고 준비가 되면 클라이언트로 스트림된다. 이것은 유저가 전체 페이지가 렌더되기 전 프리뷰를 볼 수 있게한다.

![Untitled](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Fserver-rendering-with-streaming.png&w=3840&q=75&dpl=dpl_GGqC9f3YC9X7o2mZgeZfBtLubEXa)

스트리밍은 낮은 우선 순위 ui에 효율적이고, 느린 데이터 페치에 의존한 ui는 전체 라우트 렌더를 막을 것이다. 예를 들어 제품 페이지의 리뷰

넥스트에서, 스트림 라우트 새그먼트는 로딩.js를 사용하고 ui 컴포넌트는 리액트의 서스펜스를 사용한다.
