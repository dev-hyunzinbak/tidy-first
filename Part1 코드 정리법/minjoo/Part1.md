# 코드 정리법

코드 변경을 위해 지저분한 코드를 마주칠 때마다 적용할 수 있는 작은 설계 움직임을 다룬 파트

## 1. 보호 구문

> 💡
> **항상** 그리고 **반드시** 작은 단계를 거쳐 코드를 정리하라

- depth를 늘리지 말 것

  ```jsx
  // ❌
  if (condition1) {
    if (condition2) {
      doSomething();
    }
  }

  // ✅
  if (!condition1) return;
  if (!condition2) return;
  doSomething();
  ```

- 브랜치 문 내부 간소화

  ```jsx
  // ❌
  if (condition) {
    // 매우 긴 코드
    // 여러 줄의 복잡한 로직
    // ...
  }

  // ✅
  if (!condition) return;
  // 메인 로직을 깔끔하게 배치
  ```

### 퀴즈 (with GPT)

GPT에게 보호 구문 스터디를 위한 퀴즈를 `ts` 로 작성해달라 함

```jsx
interface Order {
  isPaid: boolean;
  items: string[];
  shipping: {
    address: string,
    isValid: boolean,
  };
}

function processOrder(order: Order): string {
  if (order.isPaid) {
    if (order.items.length > 0) {
      if (order.shipping.isValid) {
        if (order.shipping.address) {
          return "주문 처리 완료";
        }
      }
    }
  }
  return "주문 처리 실패";
}
```

---

**나의 답**

```jsx
interface Order {
  isPaid: boolean;
  items: string[];
  shipping: {
    address: string,
    isValid: boolean,
  };
}

function processOrder(order: Order): string {
  if (!order.isPaid) return "주문 처리 실패: 미결제";
  if (order.items.length === 0) return "주문 처리 실패: 상품 없음";
  if (!order.shipping.isValid) return "주문 처리 실패: 유효하지 않은 배송지";
  if (!order.shipping.address) return "주문 처리 실패: 받은 배송 주소가 없음";

  return "주문 처리 완료";
}
```

## 2. 안 쓰는 코드

실행되지 않는 코드라면 지워라

커밋 로그로 남으니 복구도 가능하다 지워라

## 3. 대칭으로 맞추기

동료 및 팀원과 코드 스타일을 맞춰라

```jsx
// ❌ 같은 기능(데이터를 한 번만 초기화하는 기능)이지만, 코드 스타일이 다름
let data1: string | null = null;
function getData1() {
  if (data1 === null) {
    data1 = "Fetched Data";
  }
  return data1;
}

let data2: string | null = null;
function getData2() {
  return data2 ?? (data2 = "Fetched Data");
}

let data3: string | undefined;
function getData3() {
  return data3 ? data3 : (data3 = "Fetched Data");
}

// ✅ 한 가지 방식(if 문)으로 일관성 유지
let data: string | null = null;

function getData(): string {
  if (data === null) {
    data = "Fetched Data";
  }
  return data;
}
```

팀원 간 코드 스타일이 다르면 코드의 흐름을 파악하기 어렵다.

**코드 일관성의 중요성과 코드 정리의 필요성을 느끼며 작성하기**
(다른 사람 코드를 많이 읽어봐야겠다)

## 4. 새로운 인터페이스로 기존 루틴 부르기

새로운 기능을 추가할 때, 기존 인터페이스가 아닌 새로운 인터페이스를 추가할 것

이를 **통로 인터페이스**라 하며, 어떤 동작을 변경할 때 한층 더 수월하게 만들어 준다.

아래의 설계를 활용하기

- 거꾸로 설계
  - 루틴의 마지막부터 설계
- TDD 설계
- 도우미 설계
  - 도우미(Helper) 설계를 잘 하면 전체적인 개발 생산성이 향상됨
  - 도메인 로직을 캡슐화하고 재사용성을 높이기 위해 클래스를 확장하라

## 5. 읽는 순서

코드를 읽을 때는 독자의 입장이 되어볼 것

- 기본 요소를 먼저 이해한 다음 구성 방법을 이해할 것인지
- API 를 먼저 이해한 다음 세부 구현을 이해할 것인지

## 6. 응집도를 높이는 배치

결합도가 높은 코드들은 가깝게 배치하라

> 💡 결합도 제거(decoupling) 비용 + 변경 비용 < 결합도(coupling)에 따른 비용 + 변경 비용

**응집도를 높이는 순서로 정리하면 , 코드를 더 쉽게 변경할 수 있다**

응집도가 좋아지면 결합도 덩달아 좋아진다

## 7. 선언과 초기화를 함께 옮기기

변수 선언과 초기화를 동시에 하라

```jsx
const a = 10;
const b = a + 5;
```

변수 사이에는 데이터 종속이 있음을 존중하며, 코드를 사용하는 곳 근처에 변수를 선언하라

## 8. 설명하는 변수

1. 처음에는 간단했던 코드도 기능이 추가되면서 점점 길고 복잡해질 수 있다
2. 의미있는 변수 이름을 만들어 할당하라

```jsx
// ❌
return new Point(
  someObject.width * 2 + someObject.margin,
  someObject.height - someObject.padding
);

// ✅ better
const adjustedWidth = someObject.width * 2 + someObject.margin;
const adjustedHeight = someObject.height - someObject.padding;

return new Point(adjustedWidth, adjustedHeight);

// ✅ best(더욱 의미있는 변수명 사용)
const totalContentWidth = someObject.width * 2 + someObject.margin;
const visibleContentHeight = someObject.height - someObject.padding;

return new Point(totalContentWidth, visibleContentHeight);
```

1. 코드 정리에 대한 커밋과 동작에 대한 커밋을 분리하라

- 코드 정리에 대한 커밋
  - style: 코드 포맷팅 및 스타일 변경
  - refactor: 코드 정리
- 동작에 대한 커밋
  - feat: 새로운 기능 추가
  - fix: 버그 수정

## 9. 설명하는 상수

이해하기 어려운 숫자(매직 넘버)나 반복되는 문자열이 있을 경우, 의미 있는 상수로 변환하라

```jsx
// ❌
if (response.code === 404) {
	console.log("page not found");
}

// ✅
cosnt PAGE_NOT_FOUND = 404;

if (response.code === PAGE_NOT_FOUND) {
	console.log("page not found");
}
```

> 💡
> 연관된 상수들은 개별 변수로 선언하기보다 object로 묶어 관리하는 것이 좋다
> 최상위 스코프로 흩어진 상수들은 유지보수가 어려울 수 있기 때문이다
> 또한 객체를 활용하면 논리적 네임스페이스를 제공하여 가독성을 높이고, 전역 변수를 최소화할 수 있다.

```jsx
const HttpStatus = {
  OK: 200,
  NOT_FOUND: 404,
  INTERNAL_SERVER_ERROR: 500,
};

if (response.status === HttpStatus.NOT_FOUND) {
  console.log("Page not found");
}
```

> 💡
> `Object.freeze()`를 사용하면 객체를 보다 더 `Immutable`하게 만들 수 있다.

> 다만, 중첩된 객체에 관해서는 값이 변경이 될 수 있기 때문에 상수의 무결성이 필수라면 `deepFreeze` 함수를 권장
>
> - [**Object.freeze() - JavaScript - MDN Web Docs**](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)

## 11. 명시적인 매개변수

명시적인 매개변수를 사용하면 코드의 가독성과 유지보수성이 향상된다

- 함수가 어떤 값을 필요로 하는지 명확해짐
- 암묵적인 데이터 전달을 방지하여 예측 가능성 향상
- 코드 분석 및 테스트가 쉬워짐

```jsx
// ❌
type Parmas = { a: number, b: number };

function foo(params: Parmas) {
  console.log(params.a, params.b);
}

params = { a: 1, b: 2 };
foo(params);

// ✅
function foo(a: number, b: number) {
  console.log(a, b);
}

const a = 1;
const b = 2;
foo(a, b);
```

## 11. 비슷한 코드끼리

긴 코드를 읽다가 기능이 구분될 때는 사이에 빈 줄을 삽입하라

소프트웨어 설계는 복리와 같다. 제대로 된 소프트웨어 설계는 유연성을 확보할 수 있지만, 그렇지 못한 설계는 소용돌이에 빠질 수 있다.

## 12. 도우미 추출

목적이 분명하고 독립적인 블록은 도우미로 추출하라

이 때, `helper` 이름은 작동 방식이 아닌 목적에 따라 네이밍한다.

```jsx
// ❌
function calculate() {
  console.log(1 + 1);
  console.log(2 + 2);
}

// ✅
function calculate() {
  addOnePlusOne();
  addTwoPlusTwo();
}

function addOnePlusOne() {
  console.log(1 + 1);
}

function addTwoPlusTwo() {
  console.log(2 + 2);
}
```

## 13. 하나의 더미

코드가 너무 작게 나뉘어져 있으면 오히려 이해하기 어려울 수 있다.

관련된 코드들을 모아서 하나의 더미(`One Pile`)처럼 만들고, 이후에 정리하면 명확성이 높아진다.

다음과 같은 증상이 있을 경우 코드를 하나의 더미로 만들기를 고민해 보아라

- 길고 반복되는 인자 목록
- 반복되는 코드, 그 중에서도 반복되는 조건문
- 도우미에 대한 부적절한 이름
- 공유되어 변경에 노출된 데이터 구조

_"저도 작은 조각들 속에 들어 있는 코드를 이해하고자 갖은 애를 쓰다가 문득 , 내 자신의 능력을 의심하는 데까지 이르기도 합니다. 이때 , 저는 180도 방향을 바꿔서 모든 것을 한데 모으기 시작합니다"_

## 14. 설명하는 주석

가독성을 올릴 수 있는 주석이라면 주석을 작성하는 것을 권장

_"코드를 읽다가 ' 아 , 이건 이렇게 돌아가는 거구나 ! 라는 생각이 드는 순간을 아시죠 ? 바로 그 순간이 소중한 순간입니다. 기록하세요."_

## 15. 불필요한 주석 지우기

불필요한 주석은 유지보수 비용을 증가시킨다.

주석이 없어도 쉽게 이해할 수 있는 코드를 작성하라.
