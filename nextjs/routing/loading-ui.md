- [Loading UI](#loading-ui)
  - [Instant Loading States](#instant-loading-states)
    - [Good to know:](#good-to-know)
  - [Manually Defining Suspense Boundaries](#manually-defining-suspense-boundaries)
  - [What I Learned](#what-i-learned)

# Loading UI

넥스트 13은 리액트의 서스펜스를 활용해 의미있는 로딩 UI를 만들 수 있는 `loading.js`라는 새로운 파일 컨벤션을 소개했습니다.
이 파일 컨벤션으로 서버에서 라우트 세그먼트가 로드될 때, 즉각적인 로딩 상태를 보여줄 수 있습니다.
새로운 컨텐트는 렌더링이 완료되면 자동으로 교체됩니다.

![](https://assets.vercel.com/image/upload/f_auto%2Cq_100%2Cw_1600/v1666704760/nextjs-docs/darkmode/loading-ui.png)

## Instant Loading States

즉각적인 로딩 상태는 이동 중에 보여지는 fallback UI입니다.
당신은 로딩 인디케이터들 스켈레톤, 스피너 또는 작지만 의미있는 커버 사진, 제목 등을 프리 렌더할 수 있습니다. 이것은 유저가 앱이 반응하고 있다는 것을 알려주며 더 나은 유저 경험을 제공합니다.

`loading.js`를 폴더에 추가하여 로딩 상태를 나타냅니다.

![](https://assets.vercel.com/image/upload/f_auto%2Cq_100%2Cw_1600/v1673552977/nextjs-docs/darkmode/loading-folder.png)

```tsx
export default function Loading() {
  // You can add any UI inside Loading, including a Skeleton.
  return <LoadingSkeleton />;
}
```

같은 폴더에서, `loading.js`는 `layout.js`에 중첩됩니다. 이것은 `page.js` 파일을 `Suspense`로 감쌉니다.

![](https://assets.vercel.com/image/upload/f_auto%2Cq_100%2Cw_1600/v1666568376/nextjs-docs/darkmode/loading-diagram.png)

#### Good to know:

- 서버 중심 라우팅이어도, 이동은 즉각적입니다.
- 라우트를 변경하는 것은 모든 컨텐트가 완전히 로드 되는 것을 기다릴 필요 없습니다.
- 새로운 라우트 세그먼트가 로드될 때 공유된 레이아웃들의 상호작용은 남아있습니다.

## Manually Defining Suspense Boundaries

수동으로 Suspense Boundaries를 UI 컴포넌트에 추가할 수 있습니다.

> Recommendation: 넥스트가 성능을 향상시킨 `loading.js` 컨벤션을 사용하세요

---

## What I Learned

로딩을 처리하는 것이 항상 어렵고 귀찮은 일이었는데, 리액트 18 버전 부터 서스펜스 기능이 추가되며 선언적으로 로딩을 처리할 수 있어서 수월하게 로딩을 처리하였는데, 넥스트의 경우 컴포넌트 하나하나 감싸지 않고 `loading.js` 파일로 로딩을 관리할 수 있어서 정말 혁신적인 것 같다.

넥스트를 활용해서 블로그 말고는 프로젝트를 진행한 적이 없는데, 빨리 사용하고 싶다.

출처

- https://beta.nextjs.org/docs/routing/loading-ui
