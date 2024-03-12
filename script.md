# React 18의 서버 컴포넌트 & NextJS의 서버 컴포넌트

## 서버 컴포넌트가 왜 필요한가

### React 클라이언트 컴포넌트에서 발생하는 문제

- 레이아웃 이동 문제

  - 컴포넌트 구조 및 각 컴포넌트가 호출하는 네트워크 응답에 따라 UI 렌더가 사용자 경험을 저하
  - 하위 컴포넌트는 부모 컴포넌트의 요청과 렌더링 결과에 의존한다.

  ![](https://www.freecodecamp.org/news/content/images/2023/07/layoutshift-1.gif)
  [출처: React 서버 컴포넌트를 사용해야 하는 이유와 방법 - freeCodeCamp](https://www.freecodecamp.org/korean/news/how-to-use-react-server-components/)

- UI 구조에 따른 유지보수 비용 증가

  - 컴포넌트 구조에 따라 api 호출과 엮여있는 로직 분리 비용 발생

  ```javascript
  function Course() {
    const [info, setInfo] = useState({});

    useEffect(() => {
      const allDetails = fetchAllDetails();
      setInfo(allDetails);
    }, []);

    return (
      <CourseWrapper ino={info.wrapperInfo}>
        <CourseList ino={info.listInfo} />
        <Testimonials ino={info.testimonials} />
      </CourseWrapper>
    );
  }
  ```

- 성능 비용 감소의 한계

  - 프로젝트에 종속성이 있는 외부 라이브러리에 의해 자바스크립트 번들을 줄이는 데에 한계가 존재

  ```javascript
  import * as _ from "lodash"; // 71.5k (gzipped: 25.2k)

  const Sample = () => {
    const longText = `대법원장의 임기는 6년으로 하며, 중임할 수 없다. 모든 국민은 언론·출판의 자유와 집회·결사의 자유를 가진다. 법률안에 이의가 있을 때에는 대통령은 제1항의 기간내에 이의서를 붙여 국회로 환부하고, 그 재의를 요구할 수 있다. 국회의 폐회중에도 또한 같다.
  
    형사피고인은 유죄의 판결이 확정될 때까지는 무죄로 추정된다. 대통령의 임기가 만료되는 때에는 임기만료 70일 내지 40일전에 후임자를 선거한다. 대법관은 대법원장의 제청으로 국회의 동의를 얻어 대통령이 임명한다.`;

    const lodashText = _.truncate(longText);
    return <div>{lodashText}</div>;
  };

  export default Sample;
  ```

> 즉 React가 클라이언트단에 집중한 라이브러리이기 때문에 발생하는 문제

> 이런 문제에 대해 서버 사이드 렌더링은 정적 콘텐츠 제공의 속도를 높일 수 있지만 사용자 인터랙션에 따른 다양한 사용자 경험을 제공하기 어렵다. <br/>클라이언트 사이드 렌더링은 사용자의 인터렉션에 따라 다양한 것을 제공할 수 있자만 전자에 비해 느리고 데이터 접근이 어렵다. <br/>두 구조의 장점을 모두 취하고자 리액트 서버 컴포넌트가 등장했다.

[출처: 모던 리액트 Deep Dive: 리액트의 핵심 개념과 동작 원리부터 Next.js까지, 리액트의 모든 것 p.732](https://www.yes24.com/Product/Goods/123161563)

## 서버 컴포넌트란

> 하나의 언어, 하나의 프레임워크, 그리고 하나의 API와 개념을 사용하면서 서버와 클라이언트 모두에서 컴포넌트를 렌더링할 수 있는 기법을 의미한다.

[출처: 모던 리액트 Deep Dive: 리액트의 핵심 개념과 동작 원리부터 Next.js까지, 리액트의 모든 것 p.732](https://www.yes24.com/Product/Goods/123161563)

- 일부 컴포넌트는 클라이언트, 일부 컴포넌트는 서버에서 렌더링된다.
  > ❗️ 클라이언트 컴포넌트는 서버 컴포넌트를 import할 수 없다

### React 18에서의 세 가지 컴포넌트

0. 공용 컴포넌트(shared components)

- 서버, 클라이언트에서 모두 사용 가능하다.
- React는 모든 것을 다 공용 컴포넌트로 판단한다.

1. 서버 컴포넌트

- 요청이 오는 순간 한 번 실행된다.
- useState, useReducer와 같은 상태 관리 훅 사용 불가하다.
- useEffect, useLayoutEffect와 같은 렌더 관련 훅 사용 불가하다.
- DOM API, window, document와 같은 클라이언트 객체에 접근 불가하다.
- 데이터베이스, 내부 서비스, 파일 시스템 등 서버에만 있는 데이터를 async/await으로 접근 가능하다.
- 컴포넌트 자체를 async로 사용 가능하다.
- 다른 서버 컴포넌트를 렌더링하거나 클라이언트 컴포넌트를 렌더링할 수 있다.

2. 클라이언트 컴포넌트

- 서버 컴포넌트, 서버 전용 훅, 유틸리티를 import할 수 없다.
- 대신 클라이언트 컴포넌트가 자식으로 서버 컴포넌트를 갖는 구조는 가능하다.
- 명시적으로 선언하기 위해 파일의 맨 첫 줄에 `'use client'`를 작성한다.

[출처: 모던 리액트 Deep Dive: 리액트의 핵심 개념과 동작 원리부터 Next.js까지, 리액트의 모든 것 p.734](https://www.yes24.com/Product/Goods/123161563)

> 올해 3월 내지 4월에 출시될 React 19에서는 'use client', 'use server'로 명시적으로 구분할 것이라 예상된다. [React Labs: What We've Been Working On – February 2024 - react.dev](https://react.dev/blog/2024/02/15/react-labs-what-we-have-been-working-on-february-2024#new-features-in-react-canary)

### 서버 컴포넌트 렌더링 방식

1. 서버는 렌더링 요청을 받는다.

2. 서버는 element를 JSON으로 root 컴포넌트를 직렬화한다.

- function은 직렬화가 불가능하다.
- 서버 컴포넌트의 경우 결과를 직렬화하는데, 이때, 서버 컴포넌트는 prop으로 이벤트 핸들러를 가질 수 없다.
- 클라이언트 컴포넌트의 경우 직접 직렬화가 불가하여 module reference object를 가리키도록 배치하여 우회한다.
- module reference: React element의 타입

3. 브라우저는 React 트리를 재건축한다.

- 브라우저가 직렬화된 JSON을 받은 후 렌더하기 위해 재건축 단계를 가진다.
- 이때 클라이언트 컴포넌트의 module reference object 자리에 실제 클라이언트 컴포넌트 함수를 대체한다.

[참조: How React server components work: an in-depth guide - Plasmic Blog](https://www.plasmic.app/blog/how-react-server-components-work#life-of-an-rsc-render)

### 서버 컴포넌트의 특징

- 컴포넌트 별로 번들링이 별개로 되어 있어 필요에 따라 컴포넌트를 지연해서 받거나 따로 받을 수 있다.
- code splitting이 서버에서 자동으로 수행되어 성능 이점을 가져갈 수 있다.

```javascript
// AS-IS
import { lazy } from "react";

const OldPhotoRenderer = lazy(() => import("./OldPhotoRenderer.js"));
const NewPhotoRenderer = lazy(() => import("/NewPhotoRenderer.js"));

function Photo(props) {
  if (FeatureFlags.useNewPhotoRenderer) {
    return <NewPhotoRenderer {...props} />;
  } else {
    return <OldPhotoRenderer {...props} />;
  }
}
```

[출처: 모던 리액트 Deep Dive: 리액트의 핵심 개념과 동작 원리부터 Next.js까지, 리액트의 모든 것 p.729](https://www.yes24.com/Product/Goods/123161563)

```javascript
// TO-BE
import OldPhotoRenderer from "./OldPhotoRenderer.js";
import NewPhotoRenderer from "./NewPhotoRenderer.js";

function Photo(props) {
  if (FeatureFlags.useNewPhotoRenderer) {
    return <NewPhotoRenderer {...props} />;
  } else {
    return <OldPhotoRenderer {...props} />;
  }
}
```

### 서버 사이드 렌더링과의 차이

- 서버 사이드 렌더링

  - 응답받은 페이지 전체를 HTML로 렌더링하는 과정을 서버에서 수행한 다음 해당 결과인 HTML 문자열을 클라이언트에 내려준다.
  - 목적: 초기에 인터랙션은 불가능하지만 정적인 HTML을 빠르게 내려주는 데 초점을 두고 있다.

- 서버 컴포넌트
  - 응답받은 페이지에서 React element(JSON)를 직접 반환한다.
  - 목적: 클라이언트에 전송하는 코드의 양을 줄이는 데 초점을 두고 있다.

> 절차와 목적이 다르기 때문에 서로의 대체제가 아니다. 상호보완으로 사용 가하다.

## NextJS에서의 서버 컴포넌트

NextJS 13부터 서버 컴포넌트가 도입되어있고, app 라우터에서 구현돼 있다.

- 각 페이지는 기본적으로 서버 컴포넌트로 작동한다.

```javascript
// page.js
import ClientComponent from './ClientComponent'
import ServerComponent from '/ServerComponent'

export default function Page() {
return (
  <ClientComponent>
    <ServerComponent />
  <ClientComponent>
)
}
```

### 서버 컴포넌트 적용 후의 NextJS

#### `getServerSideProps`, `getStaticProps`, `getInitialProps`의 삭제

- 기존 SSR과 SSG를 위한 함수가 삭제되었고, 표준 API인 fetch 기반으로 데이터 요청 이뤄진다.

```javascript
// AS-IS
export default function Page({ data }) {
  return (
    <main>
      <Children data={data}>
    </main>
  )
}
```

```javascript
export const getServerSideProps = async () => {
  let fetchedData = await fetchAPI();

  return {
    props: { data: fetchedData },
  };
};
```

```javascript
// TO-BE
export default async function Page() {
  // fetchAPI는 fetch 함수로 데이터 요청한다고 가정
  const data = await fetchAPI()

  return (
    <main>
      <Children data={data}>
    </main>
  )
}
```

- fetch API를 확장하여 서버 컴포넌트 트리 내에서 동일한 요청이 있다면 재요청이 발생하지 않도록 요청 중복을 방지

  [참고: Data Fetching, Caching, and Revalidating - nextjs.org ](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching-caching-and-revalidating#caching-data)

#### SSG

- 이전 버전에서는 `getStaticProps`를 활용했으나 13부터는 정적 라우팅에 대해 기본적으로 빌드 타임에 렌더링을 미리 해두고 캐싱하여 재사용할 수 있게끔 해두었다.
- 동적 라우팅에 대해서는 서버에 매번 요청이 올 때마다 컴포넌트를 렌더링하도록 변경했다.
- `getStaticPaths`는 `generateStaticParams`를 사용하면된다.

- 예시
  - `fetch(URL, { cache: 'force-cache' })`: 기본값으로 `getStaticProps`와 유사하게 불러온 데이터를 캐싱하여 해당 데이터로만 관리한다.
  - `fetch(URL, { cache: 'no-store' })`, ``:

## 개인적인 생각

- 단일 목적, UI 특화 라이브러리라는 점이 흐려지고 있다.
- 번들 사이즈를 줄이는 것은 빌드 시간, 비용을 줄이는 것과 직결돼있기 때문에 성과를 보이기에는 좋을 것
