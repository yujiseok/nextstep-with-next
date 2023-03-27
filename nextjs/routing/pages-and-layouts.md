- [Pages and Layouts](#pages-and-layouts)
  - [Pages](#pages)
  - [Layouts](#layouts)
    - [Root Layout (Required)](#root-layout-required)
    - [Nesting Layouts](#nesting-layouts)
  - [Templates](#templates)
  - [Modifying ](#modifying-)
  - [What I Learned](#what-i-learned)

# Pages and Layouts

넥스트 13에서 소개된 앱 라우터는 쉽게 페이지, 레이아웃, 템플릿 등을 만들 수 있는 새로운 파일 컨벤션들을 소개합니다.
이 페이지는 이런 특별한 파일들을 넥스트 애플리케이션에서 활용하는 법을 알려줍니다.

## Pages

페이지는 각 라우트의 UI입니다. `page.js`파일에서 컴포넌트를 export 해서 페이지를 구성할 수 있습니다.
중첩된 폴더를 사용하면, `page.js`는 라우트를 접근할 수 있도록 해줍니다.

![](https://assets.vercel.com/image/upload/f_auto%2Cq_100%2Cw_1600/v1667559940/nextjs-docs/darkmode/page.png)

```jsx
// `app/page.js` is the UI for the root `/` URL
export default function Page() {
  return <h1>Hello, Next.js!</h1>;
}
```

**Good to know:**

- 페이지는 서브 트리의 leaf입니다.
- `.js`, `.jsx`, `.tsx` 확장자 사용이 가능합니다.
- `page.js`는 라우트 세그먼트가 존재해야 합니다.
- 페이지는 기본적으로 서버 컴포넌트이며, 사용에 따라 클라이언트 컴포넌트로 될 수 있습니다.
- 페이지는 데이터를 페치할 수 있습니다.

## Layouts

레이아웃은 여러 페이지에서 공유되는 UI입니다. 레이아웃은 상태를 유지하며, 상호적으로 유지됩니다. 또 리렌더 되지 않습니다. 레이아웃은 중첩될 수 있습니다.

레이아웃을 `default` export를 통해 정의할 수 있습니다.
이 컴포넌트는 `children`을 prop으로 받아 자식 레이아웃이나 자식 페이지로 채워져 렌더링 합니다.

![](https://assets.vercel.com/image/upload/f_auto%2Cq_100%2Cw_1600/v1667559944/nextjs-docs/darkmode/layout.png)

```tsx
// app/dashboard/layout.tsx

export default function DashboardLayout({
  children, // will be a page or nested layout
}: {
  children: React.ReactNode;
}) {
  return (
    <section>
      {/* Include shared UI here e.g. a header or sidebar */}
      <nav></nav>

      {children}
    </section>
  );
}
```

**Good to Know**

- 최상위 레이아웃은 루트 레이아웃이라고 불립니다. 이 레이아웃은 전체 애플리케이션의 레이아웃을 공유합니다. 루트 레이아웃은 반드시 `html`과 `body` 태그를 포함해야 합니다.
- 어떤 라우트 세그먼트도 선택적으로 레이아웃을 가질 수 있습니다. 이 레이아웃은 세그먼트의 모든 페이지에 적용됩니다.
- 라우트 안의 레이아웃은 기본적으로 중첩됩니다. 각 부모 레이아웃은 자식 레이아웃을 리액트의 `children` prop을 이용해 감쌉니다.
- 라우트 그룹을 통해 특정 레이아웃을 사용할 수 있습니다.
- 레이아웃은 서버 컴포넌트 또는 클라이언트 컴포넌트가 될 수 있습니다.
- 레이아웃은 데이터를 페치할 수 있습니다.
- 레이아웃에서 자식으로 데이터를 넘겨주는 것은 불가능합니다. 하지만 동일한 데이터를 여러 라우트에서 페치할 수 있습니다. 그러면 리액트는 자동적으로 퍼포먼스에 영향없이 중복을 제거합니다.
- `.js`, `.jsx`, `.tsx` 확장자를 사용할 수 있습니다.
- `layout.js`와 `page.js`는 한 폴더에 존재할 수 있으며, 레이아웃이 페이지를 감쌉니다.

### Root Layout (Required)

루트 레이아웃은 `app` 디렉터리의 최상위에서 정의되며 모든 라우트들에 적용됩니다. 이 레이아웃은 서버에서 리턴되는 초기 HTML을 수정할 수 있게 해줍니다.

```tsx
// app/layout.tsx

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

**Good to know**

- `app` 디렉터리는 반드시 레이아웃을 포함해야 합니다.
- 루트 레이아웃은 `<html>`, `<body>` 태그를 반드시 포함해야 합니다. 넥스트가 자동으로 생성하지 않습니다.
- 빌트인 SEO를 통해 `<head>` 태그를 관리할 수 있습니다. 예를 들면, `<title>` 요소와 같은
- 라우트 그룹으로 여러 개의 루트 레이아웃을 만들 수 있습니다.
- 루트 레이아웃은 서버 컴포넌트이며, 선택적으로 클라이언트 컴포넌트로 **사용할 수 없습니다.**

> `pages` 디렉터리에서 마이그레이션할 경우 `_app.js`와 `_document.js`가 루트 레이아웃으로 교체됩니다.

### Nesting Layouts

폴더 (e.g. `app/dashboard/layout.js`) 안에 정의된 레이아웃은 특정 세그먼트(e.g. `acme.com/dashboard`)의 레이아웃이 됩니다. 그리고 세그먼트가 렌더 될 때 작동합니다.
기본적으로 레이아웃의 파일 계층은 중첩됩니다. 즉 자식 레이아웃을 `children` prop으로 감쌉니다.

![](https://assets.vercel.com/image/upload/f_auto%2Cq_100%2Cw_1600/v1666568352/nextjs-docs/darkmode/nested-layout.png)

```tsx
// app/dashboard/layout.tsx

export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return <section>{children}</section>;
}
```

만약 위와 같이 두 개의 레이아웃이 존재한다면, 루트 레이아웃은 대시보드 라우트 세그먼트들을 감싸는 대시보드 레이아웃을 감쌀 것입니다.

이 두 가지 레이아웃들은 아래와 같이 중첩됩니다.

![](https://assets.vercel.com/image/upload/f_auto%2Cq_100%2Cw_1600/v1671641894/nextjs-docs/darkmode/nested-layouts.png)

## Templates

템플릿은 자식 레이아웃과 페이지를 감싸는 것에서 레이아웃과 비슷합니다. 상태를 유지하는 레이아웃과 다르게 네비게이션시 각 자식들에서 새로운 인스턴스를 생성합니다.
즉 유저가 템플릿을 공유하는 라우트들을 네비게이션시 컴포넌트의 새로운 인스턴스가 마운트 됩니다. 돔이 새로 생성되고, 상태는 보존되지 않습니다. 그리고 이펙트들이 re-synchronized 됩니다.

템플릿은 레이아웃이 할 수 없는 특정한 동작들을 수행합니다.

- 입장/퇴장 시 라이브러리를 활용한 애니메이션
- `useEffect`(e.g 페이지뷰 로깅)와 `useState`(e.g 각 페이지 피드백 폼)에 의한 기능
- 프레임워크의 기본적인 동작을 바꿀 때. 예를 들어 서스펜스의 경우 레이아웃을 사용하면 단 한번 폴백을 보여주지만 템플릿의 경우 네비게이션시 항상 보여줍니다.

> 넥스트 팀은 템플릿을 쓸 특별한 이유가 없다면, 레이아웃을 추천합니다.

템플릿은 `template.js`에서 리액트 컴포넌트를 export 시켜서 정의할 수 있습니다. 중첩된 세그먼트들을 `children` prop으로 받아옵니다.

![](https://assets.vercel.com/image/upload/f_auto%2Cq_100%2Cw_1600/v1666568350/nextjs-docs/darkmode/template.png)

```tsx
// app/template.tsx

export default function Template({ children }: { children: React.ReactNode }) {
  return <div>{children}</div>;
}
```

레이아웃과 템플릿을 사용한 결과물은 다음과 같습니다.

```tsx
<Layout>
  {/* Note that the template is given a unique key. */}
  <Template key={routeParam}>{children}</Template>
</Layout>
```

## Modifying <head>

`app` 디렉터리에서, 빌트인 SEO를 이용해 `<head>` 태그를 수정할 수 있습니다.

```tsx
// app/page.tsx
export const metadata = {
  title: "Next.js",
};

export default function Page() {
  return "...";
}
```

`generateMetadata`를 사용해 동적인 값으로 메타데이터를 `fetch`할 수 있습니다.

```tsx
// app/page.tsx
export async function generateMetadata({ params, searchParams }) {
  return { title: "Next.js" };
}

export default function Page() {
  return "...";
}
```

---

## What I Learned

리액트로 공통 레이아웃을 사용하려면, 리액트 라우터 돔과 같은 라이브러리를 설치하고 추가 설정을 해야 하는데 넥스트의 경우 정말 간편하게 공통 레이아웃을 설정할 수 있다. 프레임워크와 라이브러리의 차이가 정말 모호했는데, 공식 문서를 읽으며 그 개념들이 조금씩 구체화된다.

템플릿의 경우 처음 접하였는데, 아직 언제 사용해야 할지 감이 잡히지 않는다. 좀 더 공부를 해야겠다.

대부분의 페이지나 레이아웃에서 서버 컴포넌트와 클라이언트 컴포넌트를 선택해서 사용할 수 있는데, 루트 레이아웃의 경우 오직 서버 컴포넌트만 사용할 수 있다는 것도 처음 알았는데 왜지?라는 생각을 했는데, app 디렉터리는 기본적으로 서버 컴포넌트인데 만약 루트 레이아웃을 클라이언트 컴포넌트로 설정을 하면 전체 애플리케이션이 클라이언트 컴포넌트가 되어서 서버 컴포넌트로 설정한 의미가 없어서 아닐까 하는 생각이 들었다.
이 부분 역시 좀 더 공부를 해봐야 할 것 같다. 특히 서버 컴포넌트가 무엇인지 확실히 해야겠다.

</br>

출처

- https://beta.nextjs.org/docs/routing/pages-and-layouts
