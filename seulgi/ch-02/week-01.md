## 2.1 변수, 매개변수, 반환값에 타입을 붙이면 된다

타입스크립트는 기본적으로 변수와 함수의 매개변수, 반환값에 타입을 부여한다.
(타이핑: 타입을 부여하는 행위, typing)

기본 타입으로는 string, number, boolean, null, undefined, symbol, bigint, object가 있다.
자바스크립트의 자료형과 대응하고, 함수와 배열도 객체로 object에 포함된다.

함수의 매개변수의 타입은 매개변수 바로 뒤에 표기하고,
함수의 반환값의 타입은 함수의 매개변수 소괄호 뒤에 표기한다.

```ts
function plus(x: number, y: number): number {
  return x + y;
}
```

<br />

## 2.2 타입 추론을 적극 활용하자

타입스크립트는 어느 정도 변수와 반환값의 타입을 스스로 추론할 수 있다.
다만 매개변수에는 어떤 값이 들어올 지 모르기 때문에 타입을 부여해야 한다.

- 타입을 표기할 떄는 "hello", 123, false와 같은 정확한 값을 입력할 수 있는 리터럴 타입이 있다.
- 타입을 표기할 떄는 더 넓은 타입으로 표기해도 문제가 없다.

### 암묵적이란?

직접 타입을 표기하지 않아서 타입스크립트가 타입을 추런했다는 것.
암묵적 any 때문에 발생하는ㄴ 에러를 implicitAny 에러라 부른다.

### let과 const의 차이?

let과 const는 전혀 다르게 타입을 추론하는데 let으로 선언한 변수는 다른 값을 대입할 수 있기에
<br />
타입을 넓게 추론하는 것을 **타입 넓히기(Type Widening)**이라고 한다.

- null, undefined를 let 변수에 대입할 때는 any로 추론한다.
- const로 선언한 symbol을 변경할 수 없는 symbol로 unique symbol이라 한다.(unique symbol 간 비교 금지)

<br />

## 2.3 값 자체가 타입인 리터럴 타입이 있다

타입스크립트는 자바스크립트이 자유도를 희생하는 대신 타입 안정성을 챙기는 언어다.
원시 자료형에 대한 리터럴 타입 외에도 객체를 표시하는 리터럴 타입이 있다.

```ts
const obj: {name: "zero"}  = {name: "zero"};
const arr: [1, 2, "five"] = [1, 2, "five"];
const func: (amount: number, unit: string) => string = (amount, unit) => amount + unit:
```

- 객체, 배열, 함수를 표시하는 리터럴 타입이다.
- 함수 리터럴 타입은 반환값 표기법이 다른데, 콜론 대신 =>를 사용한다.

### 또 어떤 특징이 있을까?

- 자바스크립트 객체는 const 변수라도 수정할 수 있으므로, 타입스크립트는 수정 가능성을 염두에 두고 타입을 넓게 추론한다.
- 값이 변하지 않는 것이 확실하다면 as const 접미사를 붙이면 된다.
- readonly 수식어가 붙으면 해당 값을 변경할 수 없다.

<br />

## 2.4 배열 말고 튜플도 있다

- 빈 배열은 any[]로 추론된다.

### 타입스크립트의 추론엔 한계가 없을까?

```ts
const array = [123, 5, 46];
array[3].toFixed();
```

array[3]이 undefined인데도 toFixed메서드를 사용할 수 있다.
number[]로 추론하기 때문인데, 이런 문제를 튜플을 사용해 해결할 수 있다.

각 요소 자리에 타입이 고정되어 있는 배열을 특별하게 **튜플** 이라 부른다.

```ts
const tuple: [number, boolean] = [1, false];
```

튜플은 []안에 정확한 타입을 하나씩 입력하면 된다.
표기하지 않은 자리는 undefined 타입이 된다.

### 튜플의 특징?

- 튜플에는 push, pop, unshift, shift와 같은 메서드를 사용할 수 있다.
- push를 사용하는 것을 막으려면 readonly라는 수식어를 붙여야 한다.

### 왜 튜플을 고정된 배열이 아닌 각 요소 자리에 타입이 고정된 배열이라 할까?

```ts
  const strNumBools = [string, number, ...boolean[]] = ['hi', 123, false, true, false];

  // 명시적 추론
  const arr1 = ["hi", true];
  const arr = [46, ...arr1]; // const arr: (string | number | boolean)[]
```

...타입[] 표기를 통해 특정 타입이 연달아 나올 수 있음을 알릴 수 있기 때문이다. 타입이 아니라 값에 전개 문법을 사용해도 타입스크립트는 타입 추론을 해낸다.

### 구조분해 할당도 가능하다!

구조분해 할당에서는 나머지 속성(rest property) 문법을 사용할 수 있다.

```ts
const [a, ...rest] = ["hi", 1, 23, 456];
// const a: string;
// const rest: [number, number, number]
```

## 2.5 타입으로 쓸 수 있는 것을 구분하자

값은 일반적으로 자바스크립트에서 사용하는 값을 가리키고, 타입은 타입을 위한 구문에서 사용하는 타입을 가리킨다. (타입은 값 X)

다만 Date, Math, Error, String, Object, Number, Boolean 등과 같은 내장 객체는 타입으로 사용할 수 있다.

```ts
const date: Date = new Date();
const math: Math = Math;
```

여기서 String, Object, Number, Boolean, Symbol은 타입으로 사용하는 것을 권장하지 않는다.
string, object, number, boolean, symbol을 사용하자!

```ts
function add(x: Number, y: Number) {
  return x + y; // error!!
}

const str1: String = "hello";
const str2: string = str1; // error!!

const obj: Object = "what";
```

> Number 간에는 연산자를 사용할 수 없고, string에 String을 대입할 수 없기 때문이다. <br />Object 타입인데도 문자열 대입이 가능한 문제도 있다.

변수에는 typeof를 앞에 붙여 타입으로 사용할 수 있다.
클래스 이름은 typeof 없이도 타입으로 사용할 수 있다.

```ts
class Person {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}

const person: Person = new Person("zero");
```

<br />

## 2.6 유니언 타입으로 OR 관계를 표현하자

타입스크립트에는 타입을 위한 새로운 연산자(operator)가 있다.
유니언 타입과 유니언 타입을 표기하기 위한 파이프 연산자(|)이다.
(자바스크립트의 비트 연산자와는 다른 역할을 한다.)

### 유니언 타입?

유니언 타입은 하나의 변수가 여러 타입을 가질 수 있는 가능성을 표시하는 것이다.

```ts
let strOrNum: string | number = "hello";
strOrNum = 123;
```

### 유니언 타입의 특징?

1. 타입스크립트는 if문을 인식하는데, 유니언 타입으로부터 정확한 타입을 찾아내는 기법을 **타입 좁히기(type Narrowing)** 이라 한다.
2. 타입 사이에만 | 연산자를 쓸 수 있는 것이 아니라 타입 앞에도 사용할 수 있다. (여러 줄에 걸쳐 유니언 표기할 때 종종 사용)

<br />

## 2.7 타입스크립트에만 있는 타입을 배우자

타입스크립트에는 자바스크립트에 없는 any, unknown, void {}, never 등의 타입들이 있다.

### any

any 타입은 모든 동작을 허용한다.
타입이 any로 추론되면 다음과 같은 implicitAny 에러가 발생한다.

```ts
function plus(x, y) {
  return x + y; // 매개변수 x, y에 대한 implicitAny 에러 발생!
}
```

**any 타입은 타입 검사를 포기한다는 것과 같다. 타입스크립트가 any로 추론하는 타입이 있다면 타입을 직접 표기해야 한다.**

#### any[]로 추론된 배열의 신기한 점?

배열에 push 메서드나 인덱스로 요소를 추가할 때마다 추론하는 타입이 바뀐다. 다만 pop으로 요소를 제거할 때는 이전 추론으로 돌아가지 못한다.

```ts
const arr = [];

arr.push("1");
arr; // string[]

arr.pop();
arr; // string[]
```

any는 숫자나 문자열 타입과 연산할 때 타입이 바뀌기도 한다.

### unknown

any와 비슷하게 모든 타입을 대입할 수 있지만, 그 후 어떠한 동작도 할 수 없다. unknown 외의 타입을 직접 표기할 수 없을 때는 as로 타입을 주장(Type Assertion)할 수 있다.

또 다른 Type Assertion 방법으론 <>을 사용하는 방법도 있다. (React JSX와 충돌 권장X)

#### as, unknown을 사용해 강제 타입 변경도 가능하다

```ts
const a: number = "123" as unknown as number;
```

#### !(non-null assertion) 연산자?

null이 아님을 주장하는 연산자지만, null외로 undefined에도 아님을 주장할 수 있다.

### void

함수의 반환값이 없는 경우 반환값이 void 타입으로 추론된다.
자바스크립트는 반환값이 없는 경우 자동으로 undefined가 반환된다.
타입은 void가 된다.

- 반환값이 void 타입이라고 해서 함수가 undefined가 아닌 다른 값을 반환하는 것을 막지는 않는다.
- void를 반환받은 값의 타입은 void가 되어버린다.

#### void의 두 가지 목적?

1. 사용자가 함수의 반환값을 사용하지 못하도록 제한한다.
2. 반환값을 사용하지 않는 콜백 함수를 타이핑할 때 사용한다.

### {}, Object

null, undefined를 제외한 모든 값을 의미한다.
다만 {}타입인 변수를 실제로 사용하면 에러가 발생하는데 {}타입은 Object 타입과 같아서 이름은 객체지만 객체만 대입할 수 있는 것은 아니다.

```ts
const obj: {} = { name: "zero" };
obj.name; // property name은 type{}에 존재하지 않는다는 에러 발생
```

object타입과는 다른 타입으로 O를 대문자로 쓴다.
실제로 사용할 수 없어 대부분 쓸모가 없는 타입으로 대입은 가능하나 사용할 수 없어 object로 타이핑하는 의미가 무색하다.

### never

어떠한 타입도 대입할 수 없다. 함수 선언문과 함수 표현식일 때 차이가 있는데 함수 선언문은 throw를 하더라도 반환값의 타입이 void이지만 함수 표현식은 never이 된다.

- infinite 함수의 경우도 무한 반복문이 들어 있어 함수가 값을 반환하지 않는데 never가 반환값의 타입이 된다.
- 무한 반복문도 함수 표현식일 때만 never 타입을 반환한다.
- 함수 선언문에서는 반환값 타입이 void로 추론될 때 never로 직접 표기하면 된다.

### 타입 간 대입 가능표

| ->        | any | unknown | {}  | void | undefined | null | never |
| --------- | --- | ------- | --- | ---- | --------- | ---- | ----- |
| any       |     | O       | O   | O    | O         | O    | X     |
| unknown   | O   |         | X   | X    | X         | X    | X     |
| {}        | O   | O       |     | X    | X         | X    | X     |
| void      | O   | O       | X   |      | X         | X    | X     |
| undefined | O   | O       | X   | O    |           | X    | X     |
| null      | O   | O       | X   | X    | X         |      | X     |
| never     | O   | O       | O   | O    | O         | O    |       |

<br />

## 2.8 타입 별칭으로 타입에 이름을 붙이자

자바스크립트에서는 특정 값을 변수에 저장해서 변수의 이름으로 대신 사용한 것처럼 타입스크립트에서도 특정 타입을 특정 이름으로 저장할 수 있다.

이렇게 기존 타입에 새로운 이름을 붙인 것을 **타입 별칭** 이라 한다.

- type 키워드를 사용해서 선언할 수 있다.
- 대문자로 시작하는 단어로 만드는 것이 관습이다.
- 타입 별칭은 주로 복잡하거나 가독성이 낮은 타입에서 사용한다.

<br />

## 2.9 인터페이스로 객체를 타이핑하자

객체 타입에 이름을 붙이는 또 하나의 방법이 인터페이스 선언을 사용하는 것이다.

- 인터페이스의 이름은 대문자로 시작하는 것이 관습이다.
- 인터페이스 속성 마지막에는 콤마, 세미콜론, 줄바꿈으로 구분 가능하다.
- 인터페이스를 한 줄로 입려할 때는 콤마나 세미콜론으로 속성을 구분한다.

### 인덱스 시그니처(Index signature)?

인터페이스 속성 키 자리에 [key: number]라는 문법이 있는데 이 객체의 length를 제외한 속성 키가 전부 number라는 의미다. 이 문법을 인덱스 시그니처라고 부른다.

```ts
interface Arr {
  length: number;
  [key: number]: string;
}
```

일반적으로 객체의 속성 키는 문자열과 심볼만 가능하다.
{}타입(null, undefined를 제외한 모든 타입)처럼 속성이 없는 인터페이스도 비슷한 역할을 하는데 null, undefined를 제외한 값을 대입할 수 있다.

### 인터페이스 선언 병합

인터페이스의 주요한 특징은 타입 별칭과는 다르게 인터페이스끼리는 서로 합쳐진다는 것이다.
같은 이름으로 여러 인터페이스를 선언할 수 있는데 이러면 모든 인터페이스가 하나로 합쳐지는 것을 **선언 병합(Declaration merging)** 이라 한다.

- 추후 다른 사람이 인터페이스를 확장할 수 있도록 하기 위함이다.

### 인터페이스 선언 병합이 가능한 이유?

객체를 수정하게 되면 타입스크립트에서 정의한 객체 타입과 달라져 에러가 발생하는 경우가 많은데 타입스크립트에서도 그 객체에 대한 타입을 수정할 수 있는 기능이 필요하게 되었기 때문이다.

> 다만, 인터페이스 간에 속성이 겹치는데 타입이 다른 경우에는 에러가 발생한다.

```ts
interface Merge {
  one: string;
}
interface Merge {
  one: number; // 같은 타입과 속성이 있어 에러 발생!
}
```

### 네임 스페이스(namespace)

다른 사람이 만든 인터페이스와 내 인터페이스 병합되기 원치 않을 때 네임 스페이스를 사용한다.
네임스페이스 내부 타입을 사용하려면 export해야 한다.

```ts
namespace Example {
  export interface Inner {
    test: string;
  }
}
```

- 네임스페이스를 중첩할 수 있고, 이 경우 내부 네임스페이스를 export해야 사용 가능하다.
- 네임스페이스 자체를 자바스크립트 값으로 사용할 수 있다.
- 네임스페이스도 이름이 겹치면 병합되기에 이를 방지하기 위해 모듈 파일이 있다.

```ts
namespace Example {
  export type test = number;
}

const ex: Example[test] = 123; // 타입 접근 불가능
```

- 네임스페이스 내부 값은 []를 사용해서 접근할 수 있지만 내부 타입은 불가능하다.

<br />
