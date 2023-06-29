# Internationalization

넥스트는 라우팅과 랜더링 컨텐츠를 다국어로 설정할 수 있게 해준다.
사이트를 다양한 로케일에 맞게 만드는 것은 번역된 컨텐츠와 국제화된 라우트를 포함한다.

## Terminology

- Locale: 언어를 설정하기 위한 식별자이자 포맷 설정. 이것은 유저의 선호 언어와 지역을 포함한다.
  - en-US: 미국에서 사용되는 영어
  - nl-NL: 네덜란드에서 사용되는 네덜란드어
  - nl: 네덜란드어, 특별한 지역 없는

## Routing Overview

이것은 유저에게 브라우저에서 로케일을 고르게 한 후, 언어 설정을 하도록 추천한다.
선호 언어를 바꿀 시 애플리케이션의 Accept-Language 해더를 수정할 것이다.

예를 들어, 다음과 같은 라이브러리들을 사용할 때, 요청에 따라 해더(기본 로케일과 지원하는 로케일)에 기반해 어떤 로케일이 선택되었는지 결정한다.

```jsx
import { match } from "@formatjs/intl-localematcher";
import Negotiator from "negotiator";

let headers = { "accept-language": "en-US,en;q=0.5" };
let languages = new Negotiator({ headers }).languages();
let locales = ["en-US", "nl-NL", "nl"];
let defaultLocale = "en-US";

match(languages, locales, defaultLocale); // -> 'en-US'
```

라우팅은 하위 경로(/fr/products) 또는 도메인(my-site.fr/products)을 통해 국제화될 수 있다.
이 정보를 통해, 미들웨어로 리다이렉트를 시킬 수 있다.

```jsx
import { NextResponse } from 'next/server'

let locales = ['en-US', 'nl-NL', 'nl']

// Get the preferred locale, similar to above or using a library
function getLocale(request) { ... }

export function middleware(request) {
  // Check if there is any supported locale in the pathname
  const pathname = request.nextUrl.pathname
  const pathnameIsMissingLocale = locales.every(
    (locale) => !pathname.startsWith(`/${locale}/`) && pathname !== `/${locale}`
  )

  // Redirect if there is no locale
  if (pathnameIsMissingLocale) {
    const locale = getLocale(request)

    // e.g. incoming request is /products
    // The new URL is now /en-US/products
    return NextResponse.redirect(
      new URL(`/${locale}/${pathname}`, request.url)
    )
  }
}

export const config = {
  matcher: [
    // Skip all internal paths (_next)
    '/((?!_next).*)',
    // Optional: only run on root (/) URL
    // '/'
  ],
}
```

최종적으로, 앱의 모든 파일들은 app/[lang]에 하위에 속하게된다.
이것은 넥스트 라우터가 동적으로 다른 로케일을 라우트에서 다룰 수 있다는 것을 가능하게 하고, 모든 레이아웃과 페이지에 lang 파라미터를 전달할 수 있다.

```jsx
// You now have access to the current locale
// e.g. /en-US/products -> `lang` is "en-US"
export default async function Page({ params: { lang } }) {
  return ...
}
```

루트 레이아웃은 새로운 폴더에 중첩될 수 있다.

## Localization

유저의 선호에 따른 컨텐츠를 보여주는 것은 넥스트에 한정되는 것이 아니다.
이 패턴은 모든 웹 앱에서 사용될 수 있다.

영어와 네덜란드어를 컨텐츠를 지원하는 앱을 원한다고 하자.
아마 두 개의 다른 로케일에 따라 매핑된 사전을 유지해야한다.

```json
{
  "products": {
    "cart": "Add to Cart"
  }
}
```

```json
{
  "products": {
    "cart": "Toevoegen aan Winkelwagen"
  }
}
```

getDictionary 함수를 생성해 요청된 로케일에 맞는 번역을 로드할 수 있다.

```js
import "server-only";

const dictionaries = {
  en: () => import("./dictionaries/en.json").then((module) => module.default),
  nl: () => import("./dictionaries/nl.json").then((module) => module.default),
};

export const getDictionary = async (locale) => dictionaries[locale]();
```

현재 선택된 언어를 전달해, 우리는 레이아웃 또는 페이지에서 사전을 페치할 수 있다.

```js
import { getDictionary } from "./dictionaries";

export default async function Page({ params: { lang } }) {
  const dict = await getDictionary(lang); // en
  return <button>{dict.products.cart}</button>; // Add to Cart
}
```

앱 디렉토리의 레이아웃과 페이지는 기본적으로 서버 컴포넌트이기에, 우리는 번역 파일의 클라이언트 번들 사이즈에 고민할 필요가 없다.
이 코드는 오직 서버에서 실행되고, 그 결과 html이 브라우저로 보내진다.

## Static Generation

전달 받은 로케일에 따라 정적 라우트를 설정하고 싶은 경우, generateStaticParams를 어떤 페이지나 레이아웃에서 사용할 수 있다.
이것은 전역적으로 될 수 있다.

```jsx
export async function generateStaticParams() {
  return [{ lang: "en-US" }, { lang: "de" }];
}

export default function Root({ children, params }) {
  return (
    <html lang={params.lang}>
      <body>{children}</body>
    </html>
  );
}
```

---

## What I Learned

다국어 지원을 구현해본 적이 없어서 정확히 어떻게 동작하는지 아직 이해가 필요하지만, 유용한 기능 같다.
