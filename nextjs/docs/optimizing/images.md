# Image Optimization

Web Almanac에 의하면, 이미지는 일반적인 웹사이트 페이지 무게의 큰 부분을 차지하며 웹사이트의 LCP 성능에 상당한 영향을 미칠 수 있습니다.

넥스트의 이미지 컴포넌트는 html의 이미지 태그를 자동 이미지 최적화 기능과 함께 확장했다.

- 사이즈 최적화: 자동으로 각 장치에 따른 알맞는 사이즈 제공, 모던 이미지 포맷인 webp 또는 avif 사용
- 시각적 안정성: 이미지가 로딩중일 때 레이아웃 쉬프트를 자동으로 방지한다.
- 빠른 페이지 로드: 이미지는 선택적 블러 플레이스홀더와 브라우저 기본 지연 로딩을 사용해 뷰포트에 들어갈 때만 로드된다.
- 에셋 유연성: 외부 서버에 저장된 이미지라도 사이즈 조절 가능

---

## Usage

```ts
import Image from "next/image";
```

이미지의 src를 정의할 수 있다.(로컬 또는 리모트)

### Local Images

로컬 이미지를 사용하기 위해서, .jpg, .png 또는 .webp 이미지 파일을 불러온다.

넥스트는 자동으로 불러온 파일의 너비와 높이를 결정한다. 이 값은 이미지 로딩 시 레이아웃 쉬프트를 막는다.

```tsx
import Image from "next/image";
import profilePic from "./me.png";

export default function Page() {
  return (
    <Image
      src={profilePic}
      alt="Picture of the author"
      // width={500} automatically provided
      // height={500} automatically provided
      // blurDataURL="data:..." automatically provided
      // placeholder="blur" // Optional blur-up while loading
    />
  );
}
```

> 동적인 await import() 와 require()는 지원되지 않는다. import는 반드시 정적으로 빌드 시 분석되어야 한다.

### Remote Images

리모트 이미지의 경우, src 프로퍼티는 URL 문자여야한다.

넥스트가 빌드 프로세스 시, 리모트 파일에 접근할 수 없기에, 너비, 높이 그리고 선택적인 blurDataURL을 제공해야한다.

너비와 높이 속성은 이미지에 맞는 종횡비를 추론하게 해주고, 이미지가 로딩 시 레이아웃 쉬프트를 방지한다.
너비와 높이는 렌더돈 이미지 파일의 사이즈를 결정하는 것이 아니다.

```tsx
import Image from "next/image";

export default function Page() {
  return (
    <Image
      src="https://s3.amazonaws.com/my-bucket/profile.png"
      alt="Picture of the author"
      width={500}
      height={500}
    />
  );
}
```

안정적으로 이미지를 최적화 하기 위해선, next.config.js에 지원되는 URL 패턴을 정의해야한다.
자세할수록 악의적인 사용을 방지한다.
예를 들어, 아래의 설정은 AWS S3 버킷의 이미지만을 허용한다.

```js
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "s3.amazonaws.com",
        port: "",
        pathname: "/my-bucket/**",
      },
    ],
  },
};
```

### Domains

때로 넥스트의 빌트인 이미지 최적화 api를 상요해, 리모트 이미지를 최적화하고 싶을 수 있다.
이것을 하기 위해, loader를 기본 설정으로 두고 이미지의 src 프랍에 절대 URL을 제공해라.

악의적인 유저로부터 애플리케이션을 보호하기 위해, 넥스트 이미지 컴포넌트에 사용될 호스트네임을 정의해야한다.

### Loaders

위의 예에서, "/me.png" URL은 로컬 이미지를 제공한다. 이것은 로더 아키텍처 덕에 가능하다.

로더는 함수로 이미지의 url을 생성한다. 이것은 제공된 src를 수정하고, 여러 사이즈에 관한 여러 url을 요청한다.
이런 여러 url은 자동으로 srcset에 사용되고, 방문자는 뷰포트에 맞는 이미지를 제공받는다.

기본 로더는 넥스트 애플리케이션의 빌트인 최적화 api에 사용되어, 웹의 어떤 이미지든 최적화하고, 넥스트 웹 서버에서 즉시 제공한다.
만약 이미지를 cdn또는 이미지 서버에서 제공하고 싶다면, 로더 함수를 정의할 수 있다.

---

## Priority

각 페이지에 lcp 엘리먼트인 이미지에 priority 프로퍼티를 추가해야한다.
이렇게 하면 넥스트는 특별하게 이미지의 우선순위를 정하고, lcp를 향상시킨다.

lcp 엘리먼트는 일반적으로 큰 이미지나 페이지 뷰포트의 보이는 텍스트 블록이다.
next dev를 실행 했을 경우, priority가 없는 lcp 이미지에 대해 콘솔 경고를 볼 수 있다.

```tsx
import Image from "next/image";
import profilePic from "../public/me.png";

export default function Page() {
  return <Image src={profilePic} alt="Picture of the author" priority />;
}
```

---

## Image Sizing

이미지의 가장 일반적인 퍼포먼스 문제는 이미지가 로드되면 다른 요소들을 밀어내는 레이아웃 쉬프트이다.
이 문제는 사용자에게 너무 짜증나, cumulative layout shift라는 자체 코어 웹 바이탈이 존재한다.
이미지 기반의 레이아웃 쉬프트를 피하기 위해선 항상 이미지의 사이즈를 줘야한다.
이것은 브라우저에게 이미지를 로드하기 전 이미지를 위한 충분한 공간을 정확하게 확보할 수 있게한다.

next/image가 좋은 성능을 보장하기 위해 설계 되었기에, 레이아웃 쉬프트에 기여하는 방식으로 사용할 수 없으며, 다음 세 가지 방법 중 하나로 크기를 조정해야한다.

1. 정적 불러오기를 통해, 자동으로
2. 너비와 높이 속성을 통해, 명시적으로
3. fill을 사용해 부모 요소를 채워도록, 암시적으로

> 이미지의 사이즈를 모르면?
> 만약 이미지 사이즈를 모른 상태에서 이미지에 접근하려할 때, 몇 가지 방법이 있다:
>
> fill 사용하기
> fill 프랍은 부모 요소의 사이즈를 사용할 수 있게 한다. 미디어 쿼리 브레이크 포인트와 일치하도록 size 프랍에 따라 이미지 부모 요소 공간을 제공하려면 css를 사용하는 것이 좋다. 또한 object-fit (fill, contain, cover)와 object-position을 사용해 이미지가 어떻게 공간을 차지하는지 정의할 수 있다.
>
> 이미지 일반화
> 제어하는 소스에서 이미지를 제공하는 경우 이미지 파이프라인을 수정해 이미지를 특정 크기로 규정하는 것이 좋다.
>
> api 호출 수정
> 애플리케이션이 api 호출을 사용하여 이미지 url을 검색하는 경우 api 호출을 수정해 url과 함께 이미지 크기를 반환할 수 있다.

---

## Styling

이미지 컴포넌트를 스타일링 하는 것은 일반 이미지 요소와 동일하나, 가이드라인이 존재한다:

- Use className or style, not styled-jsx.
  - 대부분의 경우 className 사용을 추천한다.
  - 또한 style 프랍을 사용해 인라인 스타일을 줄 수 있다.
  - styled-jsx는 사용할 수 없는데, 현재 컴포넌트를 스코프해서
- fill을 사용할 경우 부모 요소는 반드시 postion: relative를 가져야한다.
  - 이것은 이미지 요소를 레이아웃 모드에서 알맞게 렌더하는데 중요하다.
- fill을 사용할 경우 부모 요소는 반드시 display: block을 가져야한다.
  - div 요소를 사용하면 기본이지만, 다른 경우 명시해야한다.
