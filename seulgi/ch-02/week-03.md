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

- 공변성: A -> B일 때 T"A" -> T"B"인 경우
- 반공변성: A -> B일 때 T"B" -> T"A"인 경우
- 이변성: A -> B일 때 T"A" -> T"B"도 되고 T"B" -> T"A"도 되는 경우
- 무공변성: A -> B일 때 T"A" -> T"B"도 안 되고 T"B" -> T"A"도 안 되는 경우

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

> public 수식어는 생략해도 되므로 사용하지 않고, private 수식어는 private field로 대체한다. <br />
> 따라서 protected 수식어만 명시적으로 사용한다.

### 인터페이스 속성은 protected, private가 불가능하다.

implements한 클래스에서도 인터페이스 속성은 모두 public이어야 하고, 속성이 protected이거나 private인 경우는 에러가 발생한다.

### 클래스에서 오버라이드를 할 때는?

명시적으로 오버라이드할 떄는 앞에 override 수식어를 붙여야 하는데 수식어를 붙이면 부모 클래스의 메서드가 바뀔 때 확인할 수 있는 장점이 있다.

```ts
class Human {
  sleep() {
    console.log("잠온다");
  }
}

class Employee extends Human {
  override sleep() {
    console.log("졸리다");
  }
}
```

- 또한 클래스의 생성자 함수에도 오버로딩을 적용할 수 있다.
  - 일반 함수와 비슷하게 타입 선언을 여러 번 하면 된다.
  - 다만 함수의 구현부는 한 번만 와야 한다.
  - 그 구현부에서는 여러 번 타입 선언한 것들에 대해 모두 대응할 수 있어야 한다.
- 클래스 속성에도 인덱스 시그니처를 사용할 수 있다.(static 속성 포함)
  - 클래스, 인터페이스 메서드에서는 this를 타입으로 사용 가능하다.
- 인터페이스로 클래스 생성자를 타이핑 가능하다.(메서드처럼 앞에 new 연산자를 추가)

```ts
interface Human {
  new (name: string, age: number): {
    name: string;
    age: number;
  };
}
```

<br />
<br />

## 2.20.1 추상 클래스(abstract class)

implements보다 조금 더 구체적으로 클래스 모양을 정의할 수 있는 추상 클래스가 있다.
abstract class로 선언하고, abstract 클래스 속성과 메서드는 abstract일 수 있다.
추상 클래스는 실제 자바스크립트 코드로 변환 가능하다.(implements와 다른 점)

```ts
abstract class AbstractPerson {
  name: string;
  age: number;
}
```

> 객체 타이핑을 위해 인터페이스, 클래스 사용은 취향일 수 있고 자바스크립트 변환 시에도 코드로 남아야 하는 경우는 클래스 아니라면 인터페이스를 사용하면 된다.

<br />
<br />

## 2.21 enum은 자바스크립트에서도 사용할 수 있다

- enum(열거형)은 여러 상수를 나열하는 목적으로 쓰인다.
- enum 내부에 존재하는 이름은 멤버(member)라고 부른다.
- enum은 자바스크립트로 변환할 때 코드에 남는다.
- enum은 멤버의 순서대로 0부터 숫자를 할당하고, 0대신 다른 숫자를 할당할 때는 = 연산자를 쓴다.

```ts
enum Level {
  INTERMEDIATE,
  NOVICE = 3,
  MASTER,
}
```

- 문자열도 할당 가능한데, 다만 한 멤버를 문자열로 할당하면 그 다음부터는 전부 직접 값을 할당해야 한다.
- enum타입의 속성은 값으로도 활용 가능하다.
- enum타입은 브랜딩에 활용하기 좋다.

```ts
enum Money {
  WON,
  DOLLAR,
}

function wonOrDollar(param: Won | Dollar) {
    if (param.type Money.WON) {
        param;
    } else {
        param;
    }
}
```

- enum 타입을 사용하되, 자바스크립트 코드가 생성되지 않게 할 수 있는데 const enum을 사용하면 된다.

<br />
<br />

## 2.22 infer로 타입스크립트의 추론을 직접 활용하자

infer예약어는 타입스크립트의 타입 추론 기능을 극한까지 활용하는 기능이다. 컨디셔널 타입과 함께 사용한다. 타입스크립트가 스스로 추론하려는 부분을 infer로 만들어도 된다.

```ts
type El<T> = T extends (infer E)[] ? E : never;
```

- 서로 다른 타입 변수를 여러 개 동시에 사용할 수 있다.
- 혹은, 같은 타입 변수를 여러 곳에 사용할 수도 있다.
- 반환값의 타입이 매개변수의 타입과 부분집합인 경우에만 그 둘의 교집합이 된다.
  - 그 외의 경우는 모두 never가 된다.
- 매개변수에 같은 타입 변수를 선언하면 인터섹션이 된다는 사실을 바탕으로 유니언을 인터섹션으로 만드는 타입을 작성할 수 있다.

<br />
<br />

## 2.23 타입을 좁혀 정확한 타입을 얻어내자

타입스크립트가 코드를 파악해서 타입을 추론하는 것을 제어 흐름 분석(Control Flow Analysis)라고 한다.

다만 제어 흐름 분석은 완벽하지 않고, 항상 typeof를 사용할 수 있는 것이 아니기에 다양한 타입 좁히기 방법을 알아두어야 한다.

1. null도 "object"인 것처럼 오류가 있을 수 있다. "value === null" 등의 자바스크립트 문법을 활용한다.
2. boolean값은 true or false로 분기 처리한다.(꼭 유니언타입을 쓰지 않아도 된다.)
3. 배열은 Array.isArray를 활용할 수 있다.
4. 클래스는 instaceof 연산자로 구분하고, 함수도 instanceof Function으로 구분할 수 있다.
5. in연산자를 활용해 속성 타입 좁히기가 가능하다.

> 타입 좁히기는 자바스트립트 문법에서 사용해서 진행해야 한다. 자바스크립트에서도 실행할 수 있는 코드여야 하기 때문이다.

6. 브랜드 속성을 사용하면 객체 구분이 쉬워진다.
7. 직접 타입 좁히기 함수를 만들 수도 있다.

```ts
function isMoney(param: Money | Liter): param is Money {
  if (param.__type === "money") {
    return true;
  } else {
    return false;
  }
}
```

> 반환값에 타입으로 param is Money 타입과 같은 것을 타입 서술 함수로 부른다. param is Money에 is라는 특수한 연산자를 사용했는데, 이렇게 하면 isMoney의 반환값이 true일 때 매개변수의 타입도 is 뒤에 적은 타입으로 좁혀진다.

**"최대한 기본적 타입 좁히기를 먼저 시도하고, 정 안 될 떄 타입 서술을 사용하는 것이 좋다"**

<br />
<br />

## 2.24 자기 자신을 타입으로 사용하는 재귀 타입이 있다

```ts
type Recursive = {
  name: string;
  children: Recursive[];
};

const recur1: Recursive = {
  name: "test",
  children: [],
};

const recur2: Recursive = {
  name: "test",
  children: [
    {
      name: "test",
      children: [],
    },
    {
      name: "test",
      children: [],
    },
  ],
};
```

Recursive 객체 타입을 선언했는데, Recursive 객체의 속성 타입으로 다시 Recursive를 사용하고 있는 것처럼 자기 자신을 다시 사용하는 타입을 **재귀 타입** 이라 한다.

- 컨디셔널 타입에도 사용할 수 있으나 타입 인수로 사용하는 것은 불가능하다.
- 재귀 타입을 선언할 때 에러가 발생하기보다는 재귀 타입을 사용할 때 에러가 발생한다.

```ts
type InfiniteRecur<T> = { item: InfiniteRecur<T> };
type Unwrap<T> = T extends { item: infer U } ? Unwrap<U> : T;
type Result = Unwrap<InfiniteRecur<any>>; //Type instantiation is excessively deep and possibly infinite.
```

> InfiniteRecur 타입이 무한하므로 Unwrap 타입은 유한한 시간 안에 InfiniteRecur를 처리할 수 없고, 타입스크립트는 이를 파악할 수 있으므로 에러를 표시한다.

<br />
<br />

## 2.25 정교한 문자열 조작을 위해 템플리 리터럴 타입을 사용하자

템플릿 리터럴 타입은 특수한 문자열 타입으로 백틱(`)과 보간(${})을 사용하는 자바스크립트의 템플릿 리터럴과 사용법이 비슷하다.

```ts
type Literal = "literal";
type Template = `template ${Literal}`; // type Template = "template literal";
```

- 값 대신 타입을 만들기 위해 사용한다.
- 문자열 타입 안에 다른 타입을 변수처럼 넣을 수 있다.
- 템플릿 리터럴 타입을 사용하면 문자열 변수를 엄격하게 관리할 수 있다.
- 문자열 조합을 표현할 때 편리하다.

```ts
type City = "seoul" | "ulsan";
type ID = `${City}:${City}`;
```

- 제네릭 및 infer와 함께 사용하면 더 강력하게 사용 가능하다.

<br />
<br />

## 2.26 추가적인 타입 검사에는 satisfies 연산자를 사용하자

satisfies 연산자가 타입스크립트 4.9버전에 추가되었고, 타입 추론을 그대로 활용하면서 추가로 타입 검사를 하고 싶을 때 사용한다.

```ts
const universe = {
  sun: "star",
  sriius: "star", // 오타임
  earth: { type: "planet", parent: "sun" },
} satisfies {
  [key in "sun" | "sirius" | "earth"]:
    | { type: string; parent: string }
    | string;
}; // Error!
```

> 타입을 타입 추론된 것을 그대로 사용하면서, 각각의 속성들은 satisfies에 적은 타입으로 다시 한번 검사하고, 오타가 난 것들을 확인할 수 있다.

<br />
<br />

## 2.27 타입스크립트는 건망증이 심하다

```ts
try {
} catch (error) {
  if (error as Error) {
    error.message; // "error" is of type "unknown"
  }
}
```

> if문에서 Error타입을 강제 주장했지만 unknown이라 나오는 이유는 as로 강제 주장한 것이 일시적이기 때문이다. if문 내 참/거짓을 판단할 때만 주장한 타입이 사용되고 판단 후에는 원래 타입으로 돌아간다.

### 어떻게 해당 문제를 해결할까?

```ts
try {
} catch (error) {
  const err = error as Error;
  if (err) {
    err.message;
  }
}
```

주장한 타입을 계속 기억할 수 있도록 변수로 만든다. 타입을 주장할 때는 그 타입이 일시적이므로 변수에 담아야 오래 기억한다.

```ts
try {
} catch (error) {
  if (error instanceof Error) {
    err.message;
  }
}
```

> 가장 좋은 방법은 as를 쓰지 않고 타입 추론을 하는 방법이나 클래스의 인스턴스인 경우에만 가능하다는 단점이 있다.

<br />
<br />

## 2.28 원시 자료형에도 브랜딩 기법을 사용할 수 있다

원시 자료형 타입에 브랜딩 속성을 추가하는 기법을 사용하면 string, number 같은 원시 자료형 타입도 더 세밀하게 구분할 수 있다.

아래의 예시처럼 숫자 타입은 있지만 KM, Mile같은 타입이 없기 때문에 브랜딩 기법을 사용해서 더 구체적으로 타입을 정할 수 있다.

```ts
type Brand<T, B> = T & { __brand: B };
type KM = Brand<number, "km">;
type Mile = Brand<number, "mile">;

function kmToMile(km: KM) {
  return (km * 0.62) as Mile;
}

const km = 3 as KM;
const mile = kmToMile(km);
const mile2 = 5 as Mile; // Error!
kmToMile(mile2);
```

> Argument of type 'Mile' is not assignable to parameter of type 'KM' 같은 타입 불일치 문제가 발생하고 있다. 다른 T타입의 속성과 겹치지 않을 이름으로 사용하고 둘 다 number라도 구별되게 브랜드 속성을 추가하면 된다.

**브랜딩 기법을 활용하여 number 타입을 KM, Mile 타입으로 세분화하고 타입을 더 정밀하게 활용할수록 안정성도 더 올라갈 수 있다.**
