# Edge and Node.js Runtimes

넥스트의 맥락에서 런타임은 실행 중에 코드에 사용할 수 있는 라이브러리, api 및 일반 기능 세트를 나타낸다.

서버에서, 애플리케이션의 코드가 렌더되는 두 가지 런타임이 존재한다.

- 노드js 런타임은 모든 노드js api와 생태계에 맞는 라이브러리 접근이 가능하다.
- 웹 api에 기반을 둔 엣지 런타임

---

## Runtime Differences

언제 런타임을 선택해야 하는지 많은 고려사항들이 있다.

![](https://github.com/yujiseok/nextstep-with-next/assets/83855636/542e028c-cd17-4cea-b7e4-cc6987afe362)

### Edge Runtime

넥스트에서, 가벼운 엣지 런타임은 노드 api의 하위 집합이다.

엣지 런타임은 동적인 개인화된 컨텐츠나 낮은 레이턴시 그리고 간단한 함수일 경우 이상적이다.
엣지의 속도는 자원을 적게 사용하는 것에서 오는데, 이것은 많은 경우 제한이된다.

예를 들어, 버셀에서 코드가 실행되는 엣지 런타임은 1mb - 4mb를 초과할 수 없다. 이것은 가져와진 패키지, 폰트, 파일 그리고 배포 인프라에 따라 달라진다.

### Node.js Runtime

노드 런타임을 사용하는 것은 모든 노드의 api를 사용할 수 있도록 하고 npm 패키지 역시 사용할 수 있게한다. 하지만, 엣지를 사용하는 것보단 속도가 느리다.

넥스트를 노드에서 배포하는 것은, 관리, 스케일링, 그리고 인프라에 대한 설정이 필요하다. 대신에 서버리스 플랫폼인 버셀에 배포해, 이런 문제를 대신 해결할 수 있다.

### Serverless Node.js

서버리스는 엣지보다 더 확장 가능성이 필요할 경우 더 복잡한 계산 부하를 로드하는 데 적합하다.
버셀의 서버리스 함수로, 임포트된 패키지, 폰트 그리고 파일을 포함해 전체 코드 사이즈가 50mb이다.

엣지를 사용하는 라우트에 비해 단점은, 서버리스의 경우 기능 요청 처리 시작 전에 부팅하는 데 수백 밀리초가 걸릴 수 있다는 것이다. 사이트가 수시하는 트래픽 양에 따라 기능이 자주 따뜻한 상태가 아니기에 이러한 현상이 자주 발생할 수 있다.

---

## Examples

### Segment Runtime Option

각 페이지 라우트 세그먼트들에 런타임을 명시할 수 있다. `runtime` 변수를 선언한 후 내보내할 수 있다.
변수는 반드시 스트링이며 값은 `'nodejs'` 또는 `'edge'` 런타임이어야한다.

```tsx
// page.tsx
export const runtime = "edge"; // 'nodejs' (default) | 'edge'
```

런타임을 레이아웃 레벨에서 선언할 수 있으며, 레이아웃 아래의 모든 런타임을 엣지로 변경한다.

```tsx
export const runtime = "edge"; // 'nodejs' (default) | 'edge'
```

만약 세그먼트의 런타임이 지정되지 않았을 경우, 노드js가 기본 런타임이다.
노드js 런타임을 변경하지 않을 경우 런타임 옵션을 사용할 필요 없다.
