## 2.17 같은 이름의 함수를 여러 번 선언할 수 있다

자바스크립트에서는 함수의 매개변수에 개수와 타입이 고정되어 있지 않다.<br />
하지만 타입스크립트는 매개변수에 어떤 타입과 값이 들어올 지 미리 타입 선언해야 한다.

```ts
function add(x: string | number, y: string | number): string | number {
  return x + y; // Operator '+' cannot be applied to types 'string | number' and 'string | number'.
}
```

> x + y를 할 수 없다는 에러가 발생한다.<br /> 애초에 매개변수 x와 y를 모두 string | number로 타이핑했기에 x가 문자열이면서 y가 숫자일 수 있게 된다.

<br />

### 위와 같은 문제에 필요한 기법이 "오버 로딩(overloading)"!

오버 로딩(overloading)은 호출할 수 있는 함수의 타입을 미리 여러 개 타이핑해두는 기법이다.
아래와 같이 코드를 변경할 수 있다.

```ts
function add(x: number, y: number): number;
function add(x: string, y: string): string;
function add(x: any, y: any) {
  return x + y;
}

add(1, 2);
add(1, "2"); // Error!
add("1", 2); // Error!
```

> No overload matches this call. Overload 1 of 2, '(x: number, y: number): number', gave the following error. Argument of type 'string' is not assignable to parameter of type 'number'. Overload 2 of 2, '(x: string, y: string): string', gave the following error. Argument of type 'number' is not assignable to parameter of type 'string'.

- 처음 두 선언은 타입만 있고 함수의 구현부(implementation)은 없다.
- 마지막 선언은 구현부는 있으나 매개변수의 타입이 any이다.
- 함수의 두 오버로딩 중 어디에도 해당하지 않아 에러가 발생했다 알려준다.

<br />

### 오버로딩을 선언하는 순서도 타입 추론에 영향을 끼친다!

```ts
function a(b: string): string;
function a(b: string | null): number;

function a(b: string | null): string | number {
  if (b) {
    return "string";
  } else {
    return 1;
  }
}

const result = a("what?"); // result :string
```

> 여러 오버로딩에 해당할 수 있는 경우는 제일 먼저 선언된 오버로딩에 해당된다. 따라서 result는 첫 번째 오버로딩의 반환값 타입인 string이다.

**오버로딩 순서를 바꾸면, result는 "number"로 바뀐다.** <br />
실제로는 result가 string이므로 에러가 발생하지만 결과값은 바뀐다.
오버로딩의 순서는 좁은 타입부터 넓은 타입 순으로 오게 해야 문제가 없다.

<br />

### 인터페이스, 타입 별칭 또한 오버로딩 표현이 가능하다.

```ts
// 인터페이스 오버로딩
interface Add {
  (x: number, y: number): number;
  (x: string, y: string): string;
}

// 타입 오버로딩
type Add1 = (x: number, y: number) => number;
type Add2 = (x: string, y: string) => string;
type Add = Add1 & Add2;

const add: Add = (x: any, y: any) => x + y;

add(1, "2"); // 동일 에러 발생!
```

> 인터페이스 외에도 타입 별칭으로 각각의 함수를 선언한 뒤 & 연산자로 하나로 묶으면 오버로딩과 같은 역할을 한다. <br /> 유니언이나 옵셔널 매개변수를 활용할 수 있다면 오버로딩을 쓰지 않는 게 좋다.

<br />
<br />

## 2.18 콜백 함수의 매개변수는 생략 가능하다

인수로 제공하는 콜백 함수의 매개변수에는 타입을 표기하지 않아도 된다.

```ts
function a(callback: (error: Error, result: string) => void) {}

a((e, r) => {});
```

> 함수를 선언할 때 이미 콜백 함수에 대한 타입을 표기했기 때문에 (e, r) => {} 함수는 callback 매개변수의 타입으로 추론된다.

<br />

### 문맥적 추론(Contextual Typing)?

매개변수 e는 Error타입, r은 string 타입이 되는 현상을 문맥적 추론이라 한다.

### 또, 콜백 함수에 대해 더 알아보자!

- 콜백 함수 매개변수는 함수를 호출할 때 사용하지 않아도 된다.
- 콜백 함수에 매개변수 자리가 없어도 호추할 수 있다. (옵셔널로 만드는 실수를 하지 말자!)
- 옵셔널로 만들 경우, 각 타입에 undefined가 추가되어 의도와는 달라진다.
- 콜백 함수의 반환값이 void이면 어떠한 반환값이 와도 된다.

<br />

### forEach 메서드로 살펴보자.

```ts
[1, 2, 3].forEach((item, index, array) => {
  console.log(item, index, array);
});
```

> (method) Array<number>.forEach(callbackfn: (value: number, index: number, array: number[]) => void, thisArg?: any): void

- forEach 메서드의 콜백 함수는 `callbackfn` 타입이다.
- 콜백 함수의 매개변수에 타입을 표기할 필요가 없고 매개변수도 전부 옵셔널이다.
- `callbackfn` 반환값 타입이 void라서 반환값이 없어도 되고 있어도 상관없다.

<br />
<br />

## 2.19 공변성과 반공변선을 알아야 함수끼리 대입할 수 있다

어떤 함수는 다른 함수에 대입할 수 있는데 대입이 불가능한 경우도 있다.
이를 이해하기 위해 공변성, 반공변성이라는 개념을 알아야 한다.

- 공변성: A -> B일 때 T<A> -> T<B>인 경우
- 반공변성: A -> B일 때 T<B> -> T<A>인 경우
- 이변성: A -> B일 때 T<A> -> T<B>도 되고 T<B> -> T<A>도 되는 경우
- 무공변성: A -> B일 때 T<A> -> T<B>도 안 되고 T<B> -> T<A>도 안 되는 경우

> 기본적으로 타입스크립트는 공변성을 갖고 있지만 함수의 매개변수는 반공변성을 갖고 있다. (TS Config 메뉴에서 strictFunctionTypes 옵션이 체크되어야 한다.)

<br />

### 실제 코드로 테스트해보자!

```ts
function a(x: string): number {
  return 0;
}

type B = (x: string) => number | string;
let b: B = a;
```

- a 함수를 b 타입에 대입할 수 있다. (현재 두 타입의 차이는 반환값뿐)
- 함수의 반환값 타입을 보면 b가 a보다 넓은 타입이다.
- a는 number를 반환하고, b는 number | string을 반환한다.(a 반환값을 b 반환값에 대입 가능)

> 반환값에 대해서는 항상 공변성을 가진다.

### 객체의 메서드를 타이핑할 때도 타이핑 방법에 따라 변성이 정해진다!

```ts
interface SayMethod {
  say(a: string | number): string;
}
interface SayFunction {
  say(a: string | number): string;
}
interface SayCall {
  say: {
    (a: string | number): string;
  };
}

const sayFunc = (a: string) => "hello";

const MyAddingMethod: SayMethod = {
  say: sayFunc, // 이변성
};
const MyAddingFunction: SayFunction = {
  say: sayFunc, // 반공변성
};
const MyAddingCall: SayCall = {
  say: sayFunc, // 공변성
};
```

> "함수(매개변수): 반환값"으로 선언한 것은 매개변수가 이변성을 가지기 때문이고, "함수: (매개변수) => 반환값"으로 선언한 것은 반공변성을 가진다.

<br />
<br />

## 2.20 클래스는 값이면서 타입이다

타입스크립트는 name, age, married 같은 멤버를 클래스 내부에 한 번에 적어야 한다.
멤버의 타입은 생략할 수 있고, 타입스크립트가 생성자 함수를 통해 알아서 추론한다.

```ts
class Person {
  name: string;
  age;
  married;
  constructor(name: string, age: number, married: boolean) {
    this.name = name;
    this.age = age;
    this.married = married;
  }
}
```

### 클래스 표현식으로도 선언할 수 있다!

표현식으로 선언하되 멤버는 항상 constructor 내부와 짝이 맞아야 한다.

```ts
const person = class {
  name;
  age;
  married;
  constructor(name: string, age: number, married: boolean) {
    this.name = name;
    this.age = age;
    this.married = married;
  }
};
```

### 생성자 내부에 선언할 때 주의할 것

생성자 내부에 할당 없이 멤버로만 선언하면 생성자 안에서 할당되지 않았다는 에러가 발생한다.
멤버를 선언하지 않고 생성자에서만 만들면 해당 속성이 클래스 안에 없다고 에러가 발생한다.

조금 더 엄격하게, 클래스 멤버가 제대로 들었는지 검사하려면 `인터페이스 + implements 예약어`s를 사용한다.

```ts
interface Human {
  name: string;
  age: number;
  married: boolean;
  sayName(): void;
}

class Person implements Human {
  name;
  age;
  married;
  constructor(name: string, age: number, married: boolean) {
    this.name = name;
    this.age = age;
    this.married = married;
  }
} // Class 'Person' incorrectly implements interface 'Human'. Property 'sayName' is missing in type 'Person' but required in type 'Human'.
```

> 타입스크립트는 생성자 함수 방식으로 객체를 만드는 것을 지원하지 않는다! 따라서, 클래스가 new를 붙여 호출할 수 있는 유일한 객체이다.

### 클래스 자체의 타입이 필요하다면 "typeof 클래스 이름"으로 타이핑한다.

```ts
const P: typeof Person = Person;
const person2 = new P("nero", 32, true);
```

### 클래스 멤버 중 수식어 속성을 알아보자!

클래스 멤버로는 옵셔널이나 readonly뿐만 아니라 public, protected, private 수식어가 추가되었다.

- public 속성: 선언한 자신의 클래스, 자손 클래스, new 호출로 만들어낸 인스턴스에서 속성을 사용할 수 있다. (자손 클래스란 extends로 상속받은 클래스를 의미한다.)
- protected 속성: 자신의 클래스와 자손 클래스에서는 속성을 사용할 수 있으나 인스턴스에서는 사용할 수 없다.
- private 속성: 자신의 클래스에서만 속성을 사용할 수 있다. value 속성은 Child 클래스나 child.value에서 에러가 발생한다.

| 수식어    | 자신 class | 자손 class | 인스턴스 |
| --------- | ---------- | ---------- | -------- |
| private   | O          | X          | X        |
| protected | O          | O          | X        |
| public    | O          | O          | O        |

