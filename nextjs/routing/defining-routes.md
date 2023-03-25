- [Defining Routes](#defining-routes)
  - [Creating Routes](#creating-routes)
  - [Creating UI](#creating-ui)
  - [Route Groups](#route-groups)
    - [Convention](#convention)
    - [Example: Organize routes without affecting the URL path](#example-organize-routes-without-affecting-the-url-path)
    - [Example: Opting specific segments into a layout](#example-opting-specific-segments-into-a-layout)
    - [Example: Creating multiple root layouts](#example-creating-multiple-root-layouts)
    - [Good to know](#good-to-know)
  - [Dynamic Segments](#dynamic-segments)
    - [Convention](#convention-1)
    - [Example](#example)
    - [Catch-all Segments](#catch-all-segments)
    - [Optional Catch-all Segments](#optional-catch-all-segments)
    - [TypeScript](#typescript)
  - [What I Learned](#what-i-learned)

# Defining Routes

넥스트에서 라우트를 처리하고 구성하는 법에 대해 다룹니다.

## Creating Routes

`app` 디렉터리안의 폴더들은 라우트로 정의됩니다.

각각의 폴더들은 라우트 세그먼트로 URL 세그먼트에 매핑됩니다. 중첩된 라우트는 폴더 안에 폴더를 만듭니다.

![](https://assets.vercel.com/image/upload/f_auto%2Cq_100%2Cw_1600/v1667553431/nextjs-docs/darkmode/route-segments-to-path-segments.png)

`page.js` 파일은 라우트에 접근할 수 있도록 해줍니다.

![](https://assets.vercel.com/image/upload/f_auto%2Cq_100%2Cw_1600/v1666568301/nextjs-docs/darkmode/defining-routes-page.js.png)

위 예제에서 `/dashboard/analytics` URL 경로에 접근하면, `page.js`파일이 없기에 접근할 수 없습니다.
이 폴더는 컴포넌트, 스타일시트, 이미지, 혹은 .파일들을 저장하는 폴더가 될 수 있습니다.

> `.js`, `.jsx` 또는 `.tsx` 확장자 모두 사용 가능

## Creating UI

특별한 파일 이름들은 각각의 라우트 세그먼트를 위한 UI를 담당합니다. pages는 라우트의 UI를 보여주고 layouts는 공통적인 UI를 보여줍니다.

```js
// app/page.js

export default function Page() {
  return <h1>Hello, Next.js!</h1>;
}
```

## Route Groups

`app` 폴더의 계층은 URL에 바로 매핑됩니다. 하지만 라우트 그룹을 통해 이런 룰을 깰 수 있습니다.
라우트 그룹은 다음과 같이 사용됩니다.

- URL 구조에 영향을 안 주고 라우트를 구성하고 싶을 때
- 레이아웃에 특정한 라우트 세그먼트를 보여주고 싶을 때
- 다양한 루트 레이아웃을 만들기 위해

### Convention

라우트 그룹은 폴더를 소괄호로 감싸서 만들 수 있습니다. `(folderName)`

### Example: Organize routes without affecting the URL path

라우트를 URL에 영향을 주지 않고 구성하고 싶을 때, 연관된 그룹을 생성합니다.
소괄호로 감싸진 폴더는 URL에서 생략됩니다. (e.g. `(marketing)` or (`shop`))

![](https://assets.vercel.com/image/upload/f_auto%2Cq_100%2Cw_1600/v1666568349/nextjs-docs/darkmode/route-group-organisation.png)

`(marketing)`과 (`shop`) 같은 URL 구조를 공유하지만, 서로 다른 레이아웃을 만들 수 있습니다.

![](https://assets.vercel.com/image/upload/f_auto%2Cq_100%2Cw_1600/v1666568349/nextjs-docs/darkmode/route-group-multiple-layouts.png)

### Example: Opting specific segments into a layout

선택적 레이아웃, 새로운 라우트 그룹을 만듭니다. (e.g. `(shop)`) 그리고 같은 레이아웃을 공유하는 파일들을 폴더 안으로 옮깁니다. `(shop)` 폴더 밖에 있는 라우트는 레이아웃을 공유하지 않습니다.

![](https://assets.vercel.com/image/upload/f_auto%2Cq_100%2Cw_1600/v1666568350/nextjs-docs/darkmode/route-group-opt-in-layouts.png)

### Example: Creating multiple root layouts

여러 개의 루트 레이아웃을 만들기 위해, 최상위 `layout.js` 파일을 삭제하고 각각의 라우트 그룹에 `layout.js` 파일을 추가합니다.
이것은 다른 UI와 경험을 주기 위해 유용합니다.
각각의 루트 레이아웃 파일에는 `<html>`과 `<body>` 태그가 필요합니다.

![](https://assets.vercel.com/image/upload/f_auto%2Cq_100%2Cw_1600/v1666568348/nextjs-docs/darkmode/route-group-multiple-root-layouts.png)

`(marketing)`과 `(shop)`은 서로 다른 루트 레이아웃을 갖게 됩니다.

### Good to know

- 라우트 그룹의 이름은 구성외에 특별한 중요성이 없습니다. 그들은 URL 경로에 영향을 주지 않습니다.
- 라우트 그룹 내부의 동일 라우트가 존재하면 안됩니다. 동일한 라우트로 변환되기에 에러를 유발합니다.
- 여러 루트 레이아웃은 전체 페이지 로드를 일으킵니다.

## Dynamic Segments

정확한 세그먼트의 이름을 모르고 동적인 데이터로 라우트를 만들길 원한다면, 동적 세그먼트를 사용할 수 있습니다. 동적 세그먼트는 요청 시 채워지거나, 빌드 시 프리렌더 됩니다.

### Convention

동적 세그먼트는 폴더를 대괄호로 감싸서 생성할 수 있습니다. `[folderName]` 예를 들어`[id]`, `[slug]`

동적 세그먼트는 `params` prop을 레이아웃, 페이지, 라우트 그리고 `generateMetadata` 함수에 넘겨줍니다.

### Example

블로그의 라우트를 예를 들면 `app/blog/[slug]/page.js` 이런 식으로 될 수 있습니다.
`[slug]`가 블로그 포스트들을 위한 동적 세그먼트가 됩니다.

```js
// app/blog/[slug]/page.js

export default function Page({ params }) {
  return <div>My Post</div>;
}
```

| Route                     | Example URL | `params`      |
| ------------------------- | ----------- | ------------- |
| `app/blog/[slug]/page.js` | `/blog/a`   | `{ slug: a }` |
| `app/blog/[slug]/page.js` | `/blog/b`   | `{ slug: b }` |
| `app/blog/[slug]/page.js` | `/blog/c`   | `{ slug: c }` |

params를 생성하기 위해선 generateStaticParams() 함수를 사용해야 합니다.

> Note: 동적 세그먼트는 `pages` 디렉터리의 동적 라우트와 동일합니다.

### Catch-all Segments

동적 세그먼트는 후속 세그먼트들을 `[...folderName]`을 이용해 전부 잡을 수 있습니다.

예를 들어, `app/shop/[...slug]/page.js`는 `/shop/clothes`와 매치될 수도 있고, `/shop/clothes/tops`, `/shop/clothes/tops/t-shirts`에도 매치될 수 있습니다.

| Route                        | Example URL   | `params`                    |
| ---------------------------- | ------------- | --------------------------- |
| `app/shop/[...slug]/page.js` | `/shop/a`     | `{ slug: a }`               |
| `app/shop/[...slug]/page.js` | `/shop/a/b`   | `{ slug:["a", "b"] }`       |
| `app/shop/[...slug]/page.js` | `/shop/a/b/c` | `{ slug: ["a", "b", "c"] }` |

### Optional Catch-all Segments

Catch-all Segments는 `[[...folderName]]`을 사용하면 선택적으로 파라미터를 포함할 수 있습니다.

예를 들어, `app/shop/[[...slug]]/page.js`는 `/shop`에도 매치하고 `/shop/clothes`, `/shop/clothes/tops`, `/shop/clothes/tops/t-shirts`에도 매치될 수 있습니다.

catch-all 과의 차이점은 파라미터가 필요 없는 라우트도 매치가 된다는 점입니다.

| Route                          | Example URL   | `params`                    |
| ------------------------------ | ------------- | --------------------------- |
| `app/shop/[[...slug]]/page.js` | `/shop`       | `{}`                        |
| `app/shop/[[...slug]]/page.js` | `/shop/a`     | `{ slug: a }`               |
| `app/shop/[[...slug]]/page.js` | `/shop/a/b`   | `{ slug:["a", "b"] }`       |
| `app/shop/[[...slug]]/page.js` | `/shop/a/b/c` | `{ slug: ["a", "b", "c"] }` |

### TypeScript

타입스크립트를 사용하면 라우트 세그먼트에 따른 `params`의 타입을 지정해야합니다.

```tsx
// app/blog/[slug]/page.tsx

export default function Page({ params }: { params: { slug: string } }) {
  return <h1>My Page</h1>;
}
```

| Route                               | `params` Type Definition                 |
| ----------------------------------- | ---------------------------------------- |
| `app/blog/[slug]/page.js	`           | `{ slug: string }`                       |
| `app/shop/[...slug]/page.js	`        | `{ slug: string[] }`                     |
| `app/[categoryId]/[itemId]/page.js	` | `{ categoryId: string, itemId: string }` |

---

## What I Learned

파일 기반 라우팅 시스템은 넥스트의 가장 큰 장점이라고 생각한다. 정말 쉽게 라우트 처리를 할 수 있고, 동적 라우팅도 정말 쉽게 할 수 있는 것 같다. 도대체 어떤 흑마술을 부리고 있는 걸까

**Route Groups**, **Catch-all Segments**와 **Optional Catch-all Segments**에 대한 지식은 없었는데, 이번 기회에 알 수 있었다. 아직 완벽하게 이해하진 못했지만, 매우 유용한 기능이 틀림없는 것 같다. 넥스트가 라우팅 처리에 얼마나 많은 공을 들였는지 알 수 있었다.

</br>

출처

- https://beta.nextjs.org/docs/routing/defining-routes
