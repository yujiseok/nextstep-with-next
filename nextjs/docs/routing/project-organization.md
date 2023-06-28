# Project Organization and File Colocation

라우팅 폴더와 파일 컨벤션과는 별개로, 넥스트는 프로젝트 파일을 어떻게 배치하는지 의견이 없다.

이 페이지에서는 기본 동작과 프로젝트를 정리하는데 기능을 소개한다.

- 안전한 기본 배치
- 기능에 따른 정리
- 전략에 따른 정리

## Safe colocation by default

앱 디렉토리에서, 중첩된 폴더의 계층이 라우트 구조를 정의한다.

각 폴더는 url 경로와 상응하는 라우트 세그먼트를 나타낸다.

하지만, page.js 또는 route.js가 추가되지 않으면 접근할 수 없다.

![label](https://nextjs.org/_next/image?url%253D%252Fdocs%252Fdark%252Fproject-organization-not-routable.png%2526w%253D1920%2526q%253D75%2526dpl%253Ddpl_9Zb4JtAnDQzpKHqt16ocfto3Jnya)

또한, 라우트가 접근이 가능해도, 오직 page.js 또는 route.js에 반환된 컨텐츠만이 클라이언트에 전달된다.

![label](https://nextjs.org/_next/image?url%253D%252Fdocs%252Fdark%252Fproject-organization-routable.png%2526w%253D1920%2526q%253D75%2526dpl%253Ddpl_9Zb4JtAnDQzpKHqt16ocfto3Jnya)

이것은 프로젝트의 파일들이 앱 디렉토리의 라우트 세그먼트 안에 직접적으로 있어야 실수로 라우팅 되지 않고, 안전하다는 것을 뜻한다.

![label](https://nextjs.org/_next/image?url%253D%252Fdocs%252Fdark%252Fproject-organization-colocation.png%2526w%253D1920%2526q%253D75%2526dpl%253Ddpl_9Zb4JtAnDQzpKHqt16ocfto3Jnya)

> Good to know:
>
> - 이것은 pages 디렉토리 안에 어떤 파일도 라우트가 될 수 있었던 page 디렉토리와 다르다.
> - 앱에서 프로젝트 파일들을 배치할 수 있지만, 그럴 필요는 없다. 원하는 경우 앱 디렉토리 외부에 보관할 수 있다.

## Project organization features

넥스트는 당신의 프로젝트를 정리하기 위해 몇 가지 기능을 제공한다.

### Private Folders

프라이빗 폴더는 언더스코어를 접두사로 붙여 생성할 수 있다.
`_folderName`

이것은 프라이빗 구현 정보로 라우팅 시스템으로 간주되면 안 되므로, 폴더 및 모든 하위 폴더가 라우팅에서 제외된다.

![label](https://nextjs.org/_next/image?url%253D%252Fdocs%252Fdark%252Fproject-organization-private-folders.png%2526w%253D1920%2526q%253D75%2526dpl%253Ddpl_9Zb4JtAnDQzpKHqt16ocfto3Jnya)

앱 디렉토리가 기본적으로 안전하게 배치되어, 프라이빗 폴더들은 배치될 필요 없다. 하지만 그들은 유용하다:

- UI 로직을 라우팅 로직과 분리할 때
- 넥스트 생태계와 프로젝트 파일에 맞는 정리
- 코드 에디터에서 분류와 그룹화
- 미래의 넥스트 파일 컨벤션과 충돌 방지

> Good to know:
>
> - 프레임워크의 컨벤션이 아닌, 프라이빗 폴더를 만들고 언더스코어 패턴을 따를 수 있다.
> - 언더스코어로 시작하는 url 세그먼트를 `%5F` 접두사로 생성할 수 있다. `5FfolderName`
> - 프라이빗 폴더 컨베션을 따르지 않는다면, 넥스트의 특별한 파일 컨벤션을 살펴봐라.

### Route Groups

라우트 그룹은 폴더를 소괄호로 감싸 생성한다.
`(folderName)`

이것은 라우트의 url 경로에 속하지 않고 정리하기 위한 것을 나타낸다.

![label](https://nextjs.org/_next/image?url%253D%252Fdocs%252Fdark%252Fproject-organization-route-groups.png%2526w%253D1920%2526q%253D75%2526dpl%253Ddpl_9Zb4JtAnDQzpKHqt16ocfto3Jnya)

라우트 그룹은 다음 경우 유용하다:

- 라우트를 그룹화 할때 e.g 사이트 섹션, 의도, 팀
- 같은 세그먼트 레벨에서 레이아웃 공유

### src Directory

넥스트는 애플리케이션의 코드를 선택적으로 src 디렉토리에 둘 수 있다.
이렇게 하면, 대부분의 프로젝트 루트의 파일과 애플리케이션 코드가 분리된다.

![label](https://nextjs.org/_next/image?url%253D%252Fdocs%252Fdark%252Fproject-organization-src-directory.png%2526w%253D1920%2526q%253D75%2526dpl%253Ddpl_9Zb4JtAnDQzpKHqt16ocfto3Jnya)

### Module Path Aliases

넥스트는 모듈 경로 별칭을 제공해 깊게 중첩된 파일의 가독성을 쉽게 한다.

```jsx
// before
import { Button } from "../../../components/button";

// after
import { Button } from "@/components/button";
```

## Project organization strategies

넥스트 프로젝트의 폴더를 구성하는데 있어서 옳고 틀림은 없다.

이 섹션의 리스트는 일반적인 전략에 높은 수준의 개요이다.
가장 간단한 방법은 당신과 당신 팀에 적합한 전략을 선택하고 프로젝트의 일관성을 유지하는 것이다.

> Good to know: 우리는 components와 lib 폴더를 사용하는데, 프레임워크에 중요하지 않으며 ui, utils, hooks, styles, etc 와 같은 이름도 가능하다.

### Store project files outside of app

이 전략은 앱 디렉토리가 순수하게 라우팅을 처리할 수 있도록 모든 코드가 프로젝트 루트에 존재하는 것이다.

![label](https://nextjs.org/_next/image?url%253D%252Fdocs%252Fdark%252Fproject-organization-project-root.png%2526w%253D1920%2526q%253D75%2526dpl%253Ddpl_9Zb4JtAnDQzpKHqt16ocfto3Jnya)

### Store project files in top-level folders inside of app

앱 디렉토리안에 코드를 저장하는 방식이다.

![label](https://nextjs.org/_next/image?url%253D%252Fdocs%252Fdark%252Fproject-organization-app-root.png%2526w%253D1920%2526q%253D75%2526dpl%253Ddpl_9Zb4JtAnDQzpKHqt16ocfto3Jnya)

### Split project files by feature or route

이 전략을 전역으로 공유 되는 코드는 앱 루트에 두고 세부적인 기능은 그것을 사용하는 라우트 세그먼트에 저장하는 방식이다.

[label](https://nextjs.org/_next/image?url%253D%252Fdocs%252Fdark%252Fproject-organization-app-root-split.png%2526w%253D1920%2526q%253D75%2526dpl%253Ddpl_9Zb4JtAnDQzpKHqt16ocfto3Jnya)

---

## What I Learned

폴더 구조나, 파일의 컨벤션 등이 정해진 것을 보면서 프레임워크(넥스트)와 라이브러리(리액트)의 차이를 제법 알 수 있게 되었다.
물론 폴더 구조를 구성하는 것은 자유라고 되어있지만, 어느 정도의 틀이 있기에 폴더 구조를 고민하는 것에서 좀 더 해방될 수 있는 것 같다.
