# 04장: 서버 사이드 렌더링

<br>

- [04장: 서버 사이드 렌더링](#04장-서버-사이드-렌더링)
  - [4.1 서버 사이드 렌더링이란?](#41-서버-사이드-렌더링이란)
    - [4.1.1 싱글 페이지 애플리케이션의 세상](#411-싱글-페이지-애플리케이션의-세상)
    - [4.1.2 서버 사이드 렌더링이란?](#412-서버-사이드-렌더링이란)
    - [4.1.3 SPA와 SSR을 모두 알아야 하는 이유](#413-spa와-ssr을-모두-알아야-하는-이유)
  - [4.2 서버 사이드 렌더링을 위한 리액트 API 살펴보기](#42-서버-사이드-렌더링을-위한-리액트-api-살펴보기)
    - [4.2.1 renderToString](#421-rendertostring)
    - [4.2.2 renderToStaticMarkup](#422-rendertostaticmarkup)
    - [4.2.3 renderToNodeStream](#423-rendertonodestream)
    - [4.2.4 renderToStaticNodeStream](#424-rendertostaticnodestream)
    - [4.2.5 hydrate](#425-hydrate)
    - [4.2.6 서버 사이드 렌더링 예제 프로젝트](#426-서버-사이드-렌더링-예제-프로젝트)
  - [4.3 Next.js 톺아보기](#43-nextjs-톺아보기)
    - [4.3.1 Next.js란?](#431-nextjs란)
    - [4.3.2 Next.js 시작하기](#432-nextjs-시작하기)
    - [4.3.3 Data Fetching](#433-data-fetching)
    - [4.3.4 스타일 적용하기](#434-스타일-적용하기)
    - [4.3.5 \_app.tsx 응용하기](#435-_apptsx-응용하기)
    - [4.3.6 next.config.js 살펴보기](#436-nextconfigjs-살펴보기)

<br>

## 4.1 서버 사이드 렌더링이란?

### 4.1.1 싱글 페이지 애플리케이션의 세상

#### 싱글 페이지 애플리케이션이란?

- 렌더링과 라우팅에 필요한 대부분의 기능을 서버가 아닌 브라우저의 자바스크립트에 의존하는 방식
- 최초에 첫 페이지에서 데이터를 모두 불러온 이후에는 페이지 전환을 위한 모든 작업이 자바스크립트와 브라우저의 `history.pushState`와 `history.replcaeState`로 이뤄지기 때문에 페이지를 불러온 이후에는 서버에서 HTML을 내려받지 않고 하나의 페이지에서 모든 작업을 처리한다

- 사이트에 접속하면 전체 사이트를 모두 볼 수 있지만 실제로 소스 보기로 HTML코드를 봤을 떄 `<body/>` 내부에 아무런 내용이 없다. 이유는 사이트 렌더링에 필요한 `<body/>` 내부의 내용을 모두 자바스크립트 코드로 삽입한 이후에 렌더링하기 때문이다.

- 페이지 전환시에 새로운 HTML 페이지를 요청하는 게 아니라 자바스크립트에서 다음 페이지의 렌더링에 필요한 정보만 HTTP 요청 등으로 가져온 다음, 그 결과를 바탕으로 `<body/>` 내부에 DOM을 추가, 수정, 삭제하는 방법으로 페이지가 전환된다.

### 4.1.2 서버 사이드 렌더링이란?

- 최초에 사용자에게 보여줄 페이지를 서버에서 렌더링해 빠르게 사용자에게 화면을 제공하는 방식

- SPA는 사용자에게 제공되는 JS번들에서 렌더링을 담당하지만 SSR은 **렌더링에 필요한 작업을 모두 서버에서 수행한다.**

#### 서버 사이드 렌더링의 장점

1. 최초 페이지 진입이 비교적 빠르다

- 빠른 FCP
- 만약 최초에 사용자에게 보여줘야 할 화면에 정보가 외부 API 호출에 많이 의지해야 한다고 가정했을 때, 싱글페이지 애플리케이션이라면 사용자가 해당 페이지에 진입하고, JS리소스를 다운로드하고, HTTP요청을 수행한 이후에 이 응답의 결과를 가지고 화면을 렌더링하게 된다. 그러나 이 작업이 서버에서 이뤄진다면 한결 빠르게 렌더링될 수 있다/

- 물론 서버가 사용자에게 렌더링을 제공할 수 있을 정도의 충분한 리소스가 확보돼 있다는 일반적인 가정하에 비교한 것이다. 만약 서버가 사용자를 감당하지 못하고, 리소스를 확보하기 어렵다면 SPA보다 느려질 수 있다.

2. 검색 엔진과 SNS 공유 등 메타데이터 제공이 쉽다
   **검색 엔진이 사이트에서 필요한 정보를 가져가는 과정**

- 1. 검색엔진 로봇이 페이지에 진입
- 2. 페이지가 HTML 정보를 제공해 로봇이 HTML을 다운로드 한다. (단 다운로드만 하고 J코드는 실행하지 않는다)
- 3. 다운로드한 HTML 페이지 내부의 오픈 그래프(Open Graph)나 메타(meta) 태그 정보를 기반으로 페이지의 검색(공유)정보를 가져오고 이를 바탕으로 검색 엔진에 저장한다.

검색 엔진의 페이지 방문과 사용자의 브라우저를 이용한 페이지 방문의 큰 차이점은 페이지 내부에 있는 **JS코드의 실행 여부**다. 브라우저는 해당 페이지를 사용자에게 HTML이나 각종 정보로 제공하기 위해 JS를 실행해야 3. 누적 레이아웃 이동(CLS)이 적다 4. 사용자의 디바이스 성능에 비교적 자유롭다 5. 보안에 좀 더 안전하다

#### 서버 사이드 렌더링의 단점

1. 소스코드를 작성할 때 항상 서버를 고려해야 한다
2. 적절한 서버가 구축돼 있어야 한다
3. 서비스 지연에 따른 문제

- SPA에서는 최초에 어떤 화면이라도 보여준 상태에서, 무언가의 느린 작업이 수행되기전에 '로딩'처리를 해서 로딩이 진행중임을 나타낼 수 있지만 SSR은 서버에서 사용자에게 보여줄 페이지에 대한 렌더링 작업이 끝나기 전까지 사용자에게 그 어떤 정보도 제공할 수 없다.

### 4.1.3 SPA와 SSR을 모두 알아야 하는 이유

#### 서버 사이드 렌더링 역시 만능이 아니다

#### 싱글 페이지 애플리케이션과 서버 사이드 렌더링 애플리케이션

<br>

## 4.2 서버 사이드 렌더링을 위한 리액트 API 살펴보기

### 4.2.1 renderToString

- 인수로 넘겨받은 리액트 컴포넌트를 렌더링하고 HTML문자열로 반환하는 함수

### 4.2.2 renderToStaticMarkup

`renderToStaticMarkup`은 `renderToString`과 매우 유사한 함수로, 두 함수 모두 리액트 컴포넌트를 기준으로 HTML 문자열을 생성한다.

- 두 함수의 주된 차이점은 , `renderToStaticMarkup`이 루트 요소에 추가된 data-reactroot와 같은 리액트에서만 사용하는 추가적인 DOM 속성을 생성하지 않는다는 점이다

### 4.2.3 renderToNodeStream

`renderToNodeStream`은 `renderToString`과 결과물이 완전히 동일하지만 두 가지 차이점이 있다.

첫 번째 차이점은 renderToString과 renderToStaticMarkup 부라우저에서도 실행할 수는 있지만 renderToNodeStream은 브라우저에서 사용하는 것이 완전히 불가능하다는 점이다.

두 번째 차점은 결과물의 타입이다.renderToString은 결과물이 string인 문자열이지만, renderToNodeStream의 결과물은 Node.js의 ReadableStream이다.

### 4.2.4 renderToStaticNodeStream

리액트 자바스크립트에 필요한 리액트 속성이 제공되지 않는, hydrate를 할 필요가 없는 순수 HTML 결과물이 필요할 때 사용하는 메서드다.

### 4.2.5 hydrate

- 앞서 살펴본 두 개의 함수인 `renderToString`과 `renderToNodeStream`으로 생성된 HTML 콘텐츠에 자바스크립트 핸들러나 이벤트를 붙이는 역할을 한다.
- `renderToString`의 결과물은 단순히 서버에서 렌더링한 HTML결과물로 사용자에게 보여줄 수 있지만, 사용자가 페이지와 상호 작용하는 것은 불가능하다.
  hydrate는 이처럼 정적으로 생성된 HTML에 이벤트와 핸들러를 붙여 완전한 웹페이지 결과물을 만든다

### 4.2.6 서버 사이드 렌더링 예제 프로젝트

<br>

## 4.3 Next.js 톺아보기

### 4.3.1 Next.js란?

리액트 기반 서버 사이드 렌더링 프레임워크

### 4.3.2 Next.js 시작하기

#### pacakge.json

프로젝트 구동에 필요한 모든 명령어 및 의존성이 포함되어있다

#### next.config.js

Next.js 프로젝트의 환경 설정을 담당한다.

#### pages/\_app.tsx

애플리케이션의 전체 페이지의 시작점으로 웹 애플리케이션에서 공통으로 설정해야 하는 것들을 여기에서 실행할 수 있다

- 에러 바운더리를 사용해 애플리케이션 전역에서 발생하는 에러 처리
- reset.css 같은 전역 CSS 선언
- 모든 페이지에 공통으로 사용 또는 제공해야 하는 데이터 제공

#### pages/\_document.tsx

- 해당 파일이 없어도 실행에 지장이 없다.
- \_app.tsx가 애플리케이션 페이지 전체를 조기화하는 곳이라면, \_document.tsx는 애플리케이션의 HTML을 초기화하는 곳이다.

**\_app.tsx과의 차이점**

- `<html>`이나 `<body>`에 DOM 속성을 추가하고 싶다면 \_document.tsx을 사용한다.
- \_app.tsx는 렌더링이나 라우팅에 따라 서버나 클라이언트에서 실행될 수 있지만 \_document는 **무조건 서버에서 실행**된다. 따라서 이 파일에서 onClick 같은 이벤트 핸들러를 추가하는 것은 불가능하다. 이벤트를 추가하는 것은 클라이언트에서 실행되는 hydrate의 몫이다.

- \_app.tsx는 Next.js를 초기화하는 파일로, Next.js 설정과 관련된 코드를 모아두는 곳이며, 경우에 따라 서버와 클라이언트 모두에서 렌더링될 수 있다/
- \_document.tsx는 Next.js로 만드는 웹사이트의 뼈대가 되는 HTML 설정과 관련된 코드를 추가하는 곳이며 반드시 서버에서만 렌더링된다.

### 4.3.3 Data Fetching

Next.js에서는 서버 사이드 렌더링 지원을 위한 몇 가지 데이터 불러오기 전략이 있다.

- 이 함수는 `pages/`의 폴더에 있는 라우팅이 되는 파일에서만 사용할수 있으며, 정해진 함수명으로 export되어야 한다. 또한 예약어로 지정되어 있어 파일 외부로 함수를 내보내야 한다.
- 서버에서 미리 필요한 페이지를 만들어서 제공하거나 해당 페이지에 요청이 있을 때마다 서버에서 데이터를 조회해서 미리 페이지를 만들어서 제공할 수 있다.

- HTML을 그릴 때 필요한 데이터를 미리 가져와서 그 결과물을 HTML에 포함시킨다

### 4.3.4 스타일 적용하기

CSS Reset같은 애플리케이션 전체에 공통으로 적용하고 싶은 스타일이 있다면 \_app.tsx를 활용하면 된다.
글로벌 스타일은 다른 페이지나 컴포넌트와 충동할 수 있으므로 반드시 `_app.tsx`에서만 제한적으로 작성해야 한다.

### 4.3.5 \_app.tsx 응용하기

\_app의 getInitialProps를 추가해두고 Next.js에서 라우팅을 반복하는 테스트를 진행해보자

1. 가장 먼저 자체 페이지에 `getInitialProps`가 있는 곳을 방문

- 로그: [서버] /test/GIP에서 /test/GIP를 요청

2. getServerSideProps가 있는 페이지를 <Link>를 이용해서 방분

- 로그: [서버] /test/GSSP에서 /\_next/data/XB5sdmk5n/test/GSSP.json을 요청

3. 다시 1번의 페이지를 <Link>를 이용해서 방문

- 로그: [클라이언트] /test/GSSP에서 undefined를 요청

4. 다시 2번의 페이지를 <Link>를 이용해서 방문

- 로그: [서버] /test/GSSP 에서 /\_next/data/XB5sdmk5n/test/GSSP.json을 요청

`<Link>`나 `router`를 이용하면 이후 라우팅은 클라이언트 렌더링처럼 작동한다.
페이지 방문 최초 시점인 1번은 서버 사이드 렌더링이 전체적으로 작동해야 해서 페이지 전체를 요청했다. 그러나 이후에는 클라이언트 라우팅을 수행하기 위해 해당 페이지가 `getServerSideProps`와 같은 서버 관련 로직이 있다 하더라도 전체 페이지를 가져오는 것이 아닌, 해당 페이지의 getServerSideProps결과를 json파일만을 요청해서 가져오는 것을 확인할 수 있다.

이러한 특성을 잘 활용하면 웹 서비스를 최초에 접근했을 때만 실행하고 싶은 내용을 `app.getInitialProps`내부에 담아 둘 수 있다.

### 4.3.6 next.config.js 살펴보기

Next.js 실행에 필요한 설정을 추가할 수 있는 파일이다.