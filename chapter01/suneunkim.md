## 01장 - 리액트 개발을 위해 꼭 알아야 할 JS

### 1.1 자바스크립트 동등 비교

- 자바스크립트의 데이터 타입
  - 원시 타입과 객체 타입
  - 원시 타입 : boolean, null, undefined, number, string, symbol, bigint
  - 객체 타입 : 7가지 원시 타입 이외의 모든 것. 배열, 함수, 정규식, 클래스 포함
- 원시 타입과 객체 타입의 큰 차이점
  - 값을 저장하는 방식이 다름
  - 원시 타입은 불변 형태의 값으로 저장
  - 객체는 프로퍼티를 삭제, 추가, 수정할 수 있으므로 변경 가능한 형태로 저장
  - 또한 값을 복사할 때도 값이 아닌 참조를 전달

### 1.2 함수

- 화살표 함수 특징 1
  - consturctor를 사용할 수 없음
  - arguments가 존재하지 않음 (함수 내 모든 인자를 배열 형태로 저장)
  - 대신에 스프레드 연산자를 활용
- 화살표 함수 특징 2
  - this 바인딩이 일반 함수와 다름
  - this는 자신이 속한 객체나 자신이 생성할 인스턴스를 가리키는 값
  - 상위 스코프의 this를 따름
- 일반 함수는 호출하는 시점에 결정되는 this를 따름(호출될 때 결정됨)

### 1.3 클래스

- JS의 클래스 : 특정한 객체를 만들기 위한 일종의 템플릿 같은 개념
  - 특정한 형태의 객체를 반복적으로 만들기 위함
  - consturctor는 생성자로, 하나만 존재할 수 있으며 생략도 가능
- 인스턴스 메서드
  - 클래스를 선언하고 내부에서 선언한 메서드를 말함
  - new 로 새로 생성한 객체에서 접근할 수 있음
  - 이유는 메서드가 prototype에 선언됐기 때문
- 정적 메서드
  - 클래스 자체에 속하는 메서드로 클래스의 인스턴스 내부에서 사용 할 수 없음
  - static 키워드로 정적 메서드 정의
  - 이 때 this는 생성된 인스턴스가 아닌 클래스 자신을 가리킴
  - 전역에서 사용하는 유틸 함수의 경우 활용할 수 있음

### 1.4 클로저

- 함수와 그 함수가 선언된 렉시컬 스코프와의 조합
  - 함수가 선언될 때의 스코프에 있는 변수에 접근할 수 있게 함
  - **함수가 생성될 당시의 환경을 기억, 함수 외부의 변수에 접근할 수 있는 함수**
- 렉시컬 스코프
  - 렉시컬 스코프는 변수가 코드 내부에서 어디서 선언됐는지를 말함
  - 호출될 때 결정되는 this와 다르게 코드가 작성된 순간에 정적으로 결정됨
  - **함수가 어디에 선언 되었는지에 따라 함수가 접근할 수 있는 변수의 범위 결정**
- 클로저를 활용하면 전역 스코프의 사용을 막을 수 있음
- 리액트에서 대표적으로 useState가 클로저의 원리를 사용하고 있음
- 주의점: 클로저는 외부 함수를 기억하고 이를 내부 함수에서 가져다 쓰는 매커니즘은 성능에 영향을 미친다. (내부 함수는 외부 함수의 선언적인 환경을 기억하고 있어야 하므로 저장함)

<aside>
💡 클로저는 함수형 프로그래밍의 중요한 개념인 부수 효과가 없고 순수해야 한다는 목적을 달성하기 위해 적극적으로 사용되는 개념

</aside>

### 1.5 이벤트 루프와 비동기 통신의 이해

- 싱글 스레드 자바스크립트
  - 코드의 실행이 하나의 스레드애서 순차적으로 이루어 짐
  - 모든 코드는 동기적으로 한 번에 하나씩 처리
- 이벤트 루프 - 비동기 작업을 처리하는 메커니즘
  - 호출 스택은 수행해야 할 코드나 함수를 순차적으로 담아두는 스택
  - 호출 스택이 비어 있는지 확인하는 것이 **이벤트 루프**
  - 비동기 작업이 있으면 태스크 큐로 들어가고 호출 스택에서 제거됨
  - 이벤트 루프는 호출 스택에 실행 중인 코드가 있는지, 그리고 태스크 큐에 대기중인 함수가 있는지 반복해서 확인함
  - 호출 스택이 비었다면 태스크 큐의 대기 중인 작업이 있는지 확인하고 호출 스택에 추가
- 태스크 큐 & 마이크로 태스크 큐
  - 마이크로 태스크 큐는 대표적으로 Promise가 있으며 기존의 태스크 큐와는 다른 태스크를 처리함
  - 마이크로 태스크 큐는 기존 태스크 큐보다 우선권을 가짐
  - setTimeout과 setInterval은 Promise보다 늦게 실행됨

### 1.6 구조 분해 할당

- 전개 구문(Spread Syntax)
  - 배열이나 객체, 문자열과 같이 순회할 수 있는 값을 전개해 간결하게 사용하는 구문
  - 기존 배열에 영향을 미치지 않고 배열을 복사하는 것도 가능(얕은 복사)
  - 값만 복사할 수 있어서 참조가 다름
  ```jsx
  const arr1 = ["a", "b"];
  const arr2 = arr1;

  arr1 === arr2; // true
  // 참조를 복사하기 때문에 true
  ```