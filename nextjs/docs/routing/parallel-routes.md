# Parallel Routes

병렬 라우팅은 한개 또는 여러 페이지를 같은 레이아웃에 동시에 또는 조건부로 랜더하도록 한다.
앱의 매우 동적인 섹션들 대시보드, 피드, 소셜 사이트 등, 병렬 라우팅은 복잡한 라우팅 패턴을 구현하기위해 사용된다.

예를 들어, 팀과 분석 페이지를 동시에 랜더할 수 있다.

![image](https://github.com/yujiseok/nextstep-with-next/assets/83855636/54bed307-1a2a-49db-a2c2-bcbfb8448f42)

병렬 라우팅은 독립적인 에러와 로딩 상태를 각 라우트에 정의할 수 있게 해준다.

![image](https://github.com/yujiseok/nextstep-with-next/assets/83855636/b74b0d11-8fc2-4c14-80af-4e5f6cfbad06)

병렬 라우팅은 슬롯을 기반으로 조건(인증 상태)와 같은 따라 조건부 랜더도 가능하게 해준다. 이것은 같은 URL에서 완벽히 분리된 코드를 가능하게 한다.

![image](https://github.com/yujiseok/nextstep-with-next/assets/83855636/23f70dff-ea1d-4f4f-bf3b-232c9c1183f4)

## Convention

병렬 라우트는 슬롯을 이용해 만들어진다. 슬롯은 @folder와 같은 컨벤션으로 정의되고, 같은 레벨 레이아웃의 프랍으로 전달된다.

> 슬롯은 라우트 세그먼트가 아니라 url 구조에 영향을 끼치지 않는다. 파일 경로 /@team/members는 /members로 접근 가능하다.

예를 들어, 다음 파일 구조는 두 슬롯을 정의한다: @analytics와 @team

![image](https://github.com/yujiseok/nextstep-with-next/assets/83855636/302bc983-e21b-4286-877d-cacf79ddad4b)

위의 폴더 구조는 app/layout.js가 @analytics와 @team 슬롯을 프랍으로 접근할 수 있다는 것을 뜻하고, 병렬적으로 자식 프랍 옆에서 사용 가능하다.

```tsx
export default function Layout(props: {
  children: React.ReactNode;
  analytics: React.ReactNode;
  team: React.ReactNode;
}) {
  return (
    <>
      {props.children}
      {props.team}
      {props.analytics}
    </>
  );
}
```

> good to know: 자식 프랍은 암시적인 슬롯으로 폴더에 매핑될 필요 없다. 이것은 app/page.js는 app/@children/page.js와 동일하다.

## Unmatched Routes

기본적으로, 슬롯과 랜더되는 컨텐트는 현재 url과 상응한다.

상응하지 않는 슬롯, 그 컨텐트는 넥스트가 라우팅 기술과 폴더 구조에 따라 다르게 랜더한다.

### default.js

당신은 default.js 파일을 정의해 넥스트가 현재 url에 맞는 슬롯을 찾지 못했을 때 fallback으로 랜더할 수 있습니다.

다음 폴더 구조를 생각해보자. @team 슬롯은 settings 디렉터리를 갖지만, @analytics는 그렇지 않다.

![image](https://github.com/yujiseok/nextstep-with-next/assets/83855636/af2c7337-4410-4c3f-bbfb-25f4430b7db4)

만약 루트 /에서 /settings로 이동한다면, 컨텐트는 네비게이션 타입과 default.js의 유용성에 따라 랜더된다.

|                 |           With @analytics/default.js            |        Without @analytics/default.js         |
| :-------------: | :---------------------------------------------: | :------------------------------------------: |
| Soft Navigation |  @team/settings/page.js 와 @analytics/page.js   | @team/settings/page.js 와 @analytics/page.js |
| Hard Navigation | @team/settings/page.js 와 @analytics/default.js |                     404                      |

#### Soft Navigation

Soft Navigation - 넥스트는 슬롯의 이전 활성화 상태를 랜더, 현재 url과 맞지 않더라도

#### Hard Navigation

Hard Navigation - 네비게이션은 풀페이지 리로드를 요구 - 넥스트는 우선 맞지 않는 슬롯의 default.js의 랜더를 시도. 만약 없다면, 404 랜더

> 404는 병렬 랜더가 되면 안되는 맞지 않는 라우터가 랜더되지 않도록 도와준다.

## useSelectedLayoutSegment(s)

useSelectedLayoutSegment와 useSelectedLayoutSegments는 슬롯과 활성화된 라우트 세그먼트를 읽게 해주는 parallelRoutesKey를 받는다.

```tsx
"use client";
import { useSelectedLayoutSegment } from "next/navigation";

export default async function Layout(props: {
  //...
  authModal: React.ReactNode;
}) {
  const loginSegments = useSelectedLayoutSegment("authModal");
  // ...
}
```

유저가 url 바에서 @authModal/login 또는 /login로 이동 했을 때, loginSegments는 문자 login과 같게된다.

## Examples

### Modals

병렬 라우팅은 모달을 랜더하는 데 사용된다.

![image](https://github.com/yujiseok/nextstep-with-next/assets/83855636/c8cef478-a1a4-4f1a-8675-59a9d1ce856f)

@authModal 슬롯은 맞는 라우트에 이동했을 때 모달 컴포넌트를 랜더한다.

```tsx
export default async function Layout(props: {
  // ...
  authModal: React.ReactNode;
}) {
  return (
    <>
      {/* ... */}
      {props.authModal}
    </>
  );
}
```

```tsx
import { Modal } from "components/modal";

export default function Login() {
  return (
    <Modal>
      <h1>Login</h1>
      {/* ... */}
    </Modal>
  );
}
```

활성화 되지 않았을 때, 모달이 랜더 되지 않도록 하기 위해 당신은 null을 리턴하는 default.js를 생성할 수 있다.

```tsx
export default function Default() {
  return null;
}
```

#### Dismissing a modal

만약 모달이 클라이언트 네비게이션으로 동작되었다면, 예를 들어 <Link href="/login">, 당신은 router.back()이나 링크 컴포넌트로 모달을 무시할 수 있다.

```tsx
"use client";
import { useRouter } from "next/navigation";
import { Modal } from "components/modal";

export default async function Login() {
  const router = useRouter();
  return (
    <Modal>
      <span onClick={() => router.back()}>Close modal</span>
      <h1>Login</h1>
      ...
    </Modal>
  );
}
```

만약 다른 곳으로 이동하거나 모달을 무시하고 싶다면, catch-all 라우트를 사용할 수 있다.

```tsx
export default function CatchAll() {
  return null;
}
```

### Conditional Routes

병렬 라우트는 조건부 라우팅에 사용될 수 있다. 예를 들어, 인증 상태에 따라 @dashboard 또는 @login 라우트를 랜더할 수 있다.

```tsx
import { getUser } from "@/lib/auth";

export default function Layout({ params, dashboard, login }) {
  const isLoggedIn = getUser();
  return isLoggedIn ? dashboard : login;
}
```

![image](https://github.com/yujiseok/nextstep-with-next/assets/83855636/ee753d16-7b00-4b48-b994-0645ac971030)

---

## What I Learned

병렬 라우트는 정말 처음 들어보는 개념이다.
잘 쓰면 엄청나게 효율적일 것 같다. 특히 로그인 조건에 따라 다른 컴포넌트를 랜더할 수 있다는 것이 좋은 것 같다.
