## 객체의 속성과 메서드에 적용되는 특징을 알자

- 객체의 속성에도 옵셔널(`optional`)이나 `readonly` 수식어가 가능하다.
- 옵셔널 사용 시에는 `undefined` 사용도 허용된다.
- multiple 속성에는 `readonly`와 `?`수식어 등 여러 수식어를 동시에 붙일 수도 있다.

```ts
interface Example {
  hello: string;
  world?: number;
  readonly wow: boolean;
  readonly multiple?: symbol;
}

const example : Example = {
  hello: "hi",
  world: undefined;
  wow: false;
}
```

<br /><br />

### 대입 방식에 따라 객체 타입 검사 방식은 달라진다.

- 객체 리터럴을 대입했냐, 변수를 대입했냐에 따라 타입 검사 방식은 달라진다.
- 인수 자리에 변수로 값을 대입하면 에러가 발생하지 않지만, 객체 리터럴을 대입하면 에러가 발생한다.
- 타입스크립트에서는 객체 리터럴을 대입하면 **잉여 속성 검사**가 실행된다.

  > 잉여 속성 검사(Excess Property Checking)? <br />
  > 타입 선언에서 선언하지 않은 속성을 사용할 때 에러를 표시하는 것.

```ts
interface Money {
  amount: number;
  unit: string;
}

const money = { amount: 1000, unit: "won", error: "에러 아님" };

function addMoney(money1: Money, money2: Money): Money {
  return {
    amount: money1.amount + money2.amount,
    unit: "won",
  };
}

addMoney(money, { amount: 3000, unit: "money", error: "에러" }); // error속성에서 에러남
```

### 구조분해 할당을 올바르게 타이핑하려면?

```ts
const {
  prop: { nested },
}: { prop: { nested: string } } = { prop: { nested: "hi" } };
```

<br /><br />

## 인덱스 접근 타입

- 특정 속성 타입을 별도 타입으로 만들고 싶다면 아래와 같이 작성한다.
- 자바스크립트처럼 객체 속성에 접근하듯 접근하되 "객체.속성"꼴의 방식은 사용할 수 없다.

```ts
type Animal = {
  name: string;
};

type N1 = Animal["name"];
type N2 = Animal["name"];
type N3 = Animal.name; // 사용 불가
```

> 이렇게 객체 속성의 타입에 접근하는 방식을 "인덱스 접근 타입(Indexed Access Type)"이라 한다.

- `keyof`연산자와 `인덱스 접근 타입`을 활용해 키의 타입과 값을 타입을 구할 수 있다.
- 키의 타입은 "keyof 객체*타입"이고, 값의 타입은 "객체*타입[키의_타입]"이다.

```ts
const obj = {
  hello: "world",
};

type Keys = keyof typeof obj;
```

> obj는 값이라 타입 자리에 바로 쓸 수 없어, typeof 연산자를 붙여 타입으로 만든다. Keys 타입에는 obj의 속성 키 타입이 들어 있다.

- keyof any는 string | number | symbol 이다.
- 타입스크립트에서는 배열을 위해 객체의 키를 number까지 허용해주고 있다.
- 배열에 keyof를 적용하면 number | 배열 속성이름 유니언 | 배열 인덱스 문자열 유니언이 된다.

### 투플과 배열에도 인덱스 접근 타입을 쓸 수 있다.

```ts
type Arr2 = (string | boolean)[];
type El = Arr2[number]; // type El = string | boolean;
```

### 인덱스 접근 타입을 활용해 특정 키들의 값 타입만 추릴 수 있다.

```ts
const obj = {
  hello: "world",
  name: "seulgi",
  age: "32",
};
type Values = (typeof obj)["hello" | "name"];
```

### 객체의 메서드를 선언할 때는 세 가지 방식으로 할 수 있다.

```ts
interface Example {
  a(): void;
  b: () => void;
  c: {
    (): void;
  };
}
```

- 메서드(매개변수): 반환값
- 메서드: (매개변수) => 반환값
- 메서드: {(매개변수): 반환값}

<br /><br />

## 매핑된 객체 타입

> 매핑된 객체 타입(Mapped Object Type)? <br />
> 기존의 다른 타입으로부터 새로운 객체 속성을 만들어내는 타입

- in 연산자를 사용해서 인덱스 시그니처가 표현하지 못하는 타입을 표현한다.
- in 연산자 오른쪽에는 유니언 타입이 와야 한다.
- 튜플, 배열도 객체이므로 매핑된 객체 타입을 적용할 수 있다.

```ts
type HelloAndHi = {
  [key in "hello" | "hi"]: string;
};

// 복잡한 상황에 주로 활용
interface Original {
  name: string;
  age: number;
  married: boolean;
}
type Copy = {
  readonly [key in keyof Original]?: Original[key];
};
```

### 수식어 앞에 -를 붙이면?

해당 수식어가 제거된 채로 속성을 가져온다.

```ts
interface Original {
  readonly name?: string;
  readonly age?: number;
  readonly married?: boolean;
}

type Copy = {
  -readonly [key in keyof Original]-?: Original[key];
};
```

### Capitalize ?

속성 이름을 바꿀 수 있고, 문자열의 첫 번째 자리를 대문자화한다.
as 예약어를 통해 속성 이름을 어떻게 바꿀지 정할 수 있다.

```ts
interface Original {
  readonly name: string;
  readonly age: number;
  readonly married: boolean;
}

type Copy = {
  [key in keyof Original as Capitalize<key>]: Original[key];
}; // Name: string, Age: number, Married: boolean;
```

<br /><br />

## 타입을 집합으로 생각하자(유니언, 인터섹션)

- 유니언 연산자(|)는 실제로 합집합 역할을 한다.
- 인터섹션(intersection) & 연산자는 교집합 역할을 한다.
- 공집합 : 원소가 존재하지 않는 집합(never)

### 타입스크립트 타입을 집합 관계로 볼 수 있다.

타입스크립트 전체 집합은 `unknown`이다. 반대로 `never`는 공집합으로 좁다.
항상 좁은 타입에서 넓은 타입으로 대입해야 하며, any타입은 집합 관계를 무시하므로 연산하지 않는 것이 좋다.

### null, undefined를 제외한 원시 자료형과 비어 있지 않은 객체를 & 연산할 때는 never가 되지 않는다.

```ts
type H = { a: "b" } & number; // type H = { a: 'b'} & number;
type I = null & { a: "b" }; // type I = never;
type J = {} & string; // type J = string;
```

<br /><br />

## 타입도 상속이 가능하다.

- 상속을 하면 부모 객체에 존재하는 속성을 다시 입력하지 않아도 되므로 중복이 제거된다.
- extends 예약어를 사용해 기존 타입을 상속할 수 있다.(중복 선언 막기 가능)
- 타입 별칭에서 & 연산을 이용해 상속처럼 작업 가능하다.

```ts
interface Animal {
  name: string;
}

interface Dog extends Animal {
  bark(): void;
}

type Animal = {
  name: string;
};

type Dog = Animal & {
  bark(): void;
};
```

> 타입 별칭이 인터페이스를 상속할 수도, 인터페이스가 타입 별칭을 상속할 수도 있다.

### 한 번에 여러 타입도 상속할 수 있다!

```ts
type Animal = {
  name: string;
};
interface Dog extends Animal {
  bark(): void;
}
interface Cat extends Animal {
  meow(): void;
}
interface DogCat extends Dog, Cat {}
```

### 상속 시 부모 속성의 타입 변경이 가능하다?!

```ts
interface Merge {
  one: string;
  two: string;
}
interface Merge2 extends Merge {
  one: "h" | "w";
  two: 123;
}
```

- 완전히 다른 타입 변경 X
- 부모의 속성 타입을 바꾸더라도 부모 대입 가능한 타입으로 변경해야 한다.

<br /><br />

## 객체 간에 대입할 수 있는지 확인하는 법을 배우자

1. 객체도 동일하게 좁은 타입은 넓은 타입에 대입할 수 있지만, 넓은 타입은 좁은 타입에 대입할 수 없다.
2. 구체적이라는 것은 조건을 만족하기 어렵고 더 좁은 타입이라는 뜻이다.
3. 튜플은 배열보다 좁은 타입이며 튜플이나 배열에 readonly 수식어가 있으면 더 넓은 타입이다.
4. 객체 속성에 옵셔널이 붙으면 더 넓은 타입이다.
5. 배열과 다르게 객체에서는 속성에 readonly가 붙어도 서로 대입 가능하다.

<br /><br />

## 구조적 타이핑

타입스크립트에서는 모든 속성이 동일하면 객체 타입이 이름이 다르더라도 동일한 타입으로 취급한다.

```ts
interface Money {
  amount: number;
  unit: string;
}

interface Liter {
  amount: number;
  unit: string;
}

const liter: Liter = { amount: 1, unit: "liter" };
const circle: Money = liter;
```

> 구조적 타이핑? <br />
> 객체를 어떻게 만들었든 간에 구조가 같으면 같은 객체로 인식하는 것. 서로 대입하지 못하게 하려면 구조적으로 동일하지 않게 만들어야 한다.

### 구조적으로 다르게 만들기 위해서는

브랜드 속성을 사용해 브랜딩한다.

```ts
interface Money {
  __type: "money";
  amount: number;
  unit: string;
}

interface Liter {
  __type: "liter";
  amount: number;
  unit: string;
}

const liter: Liter = { amount: 1, unit: "liter", __type: "liter" };
const circle: Money = liter; // 오류!
```

<br /><br />

## 제네릭으로 타입을 함수처럼 사용하자

제네릭(generic)을 사용하면 타입 중복을 제거할 수 있다. 제네릭 표기는 <>로 하며 인터페이스 이름 바로 뒤에 위치한다.

### 선언한 제네릭을 사용할 때는,

매개변수에 대응하는 실제 타입 인수(Type Argument)를 넣으면 된다.

```ts
interface Person<N, A> {
  type: "human";
  race: "yellow";
  name: N;
  age: A;
}
interface Zero extends Person<"zero", 28> {}
interface Seulgi extends Person<"seulgi", 32> {}
```

### 제네릭의 특징에 대해 알아보자.

- 타입 매개변수의 개수와 타입 인수의 개수가 일치하지 않으면 에러가 발생한다.
- 클래스와 타입 별칭, 함수도 제네릭을 가질 수 있다.
- 함수에서는 함수 선언문이냐 표현식이냐에 따라 제네릭 표기 위치가 다르다.
- interface와 type 간 교차 사용 가능하다.
- 타입 매개변수는 기본값을 사용할 수 있다.
- 타입 매개변수 앞에 const 수식어를 추가하면 타입 매개변수 T를 추론할 때 as const를 붙인 값으로 추론합니다.

### 제네릭을 쓸 수 있는 위치를 알아보자.

```ts
interface 이름<타입 매개변수들> {}
type 이름<타입 매개변수들> = {}
class 이름<타입 매개변수들> {}
function 이름<타입 매개변수들>() {}
const 이름 = <타입 매개변수들>() => {}
```

<br /><br />

## 제네릭에 제약 걸기

타입 매개변수에 extends 문법으로 제약(constraint)을 걸 수 있다. 이 제약에 어긋나는 타입은 입력할 수 없지만 제약보다 더 구체적인 타입은 입력 가능하다.

하나의 타입 매개변수가 다른 타입 매개변수의 제약이 될 수 있다.

```ts
interface Example<A, B extends A> {
  a: A;
  b: B;
}

type Usecase1 = Example<string, number>; // 오류!
type Usecase2 = Example<string, "hello">;
type Usecase3 = Example<number, 123>;
```

### 제네릭에 자주 쓰이는 제약들을 알아보자!

```ts
<T extends object> // 모든 객체
<T extends any[]> // 모든 배열
<T extends (...args: any) => any> // 모든 함수
<T extends abstract new (...args: any) => any> // 생성자 타입
<T extends keyof any> // string | number | symbol
```

### 제네릭 사용 시 자주 하는 실수?

타입 매개변수와 제약을 동일하게 생각하는 실수를 하는데 아래 예시를 봐보자.

```ts
interface V0 {
  value: any;
}

const returnV0 = <T extends V0>(): T => {
  return { value: "test" };
}; // 에러 발생!
```

- T는 정확히 V0이 아니라 V0에 대입할 수 있는 모든 타입이다.
- 이 부분 이해가 안되서 공부를 더 해봐야할 것 같다 🥲

<br /><br />

## 조건문과 비슷한 컨디셔널 타입이 있다

> 컨디셔널 타입(Conditional Type)? <br />
> 조건에 따라 다른 타입이 되는 타입, 특정 타입 extends 다른 타입 ? 참 타입 : 거짓타입

```ts
type A = string;
type B = A extends string ? number : boolean;
// type B: number;
```

- 컨디셔널 타입은 타입 검사를 위해 많이 사용되고, never와도 함께 사용할 때가 있다.

### 컨디셔널 타입을 자바스크립트의 삼항연산자처럼 중첩해서 사용할 수 있다.

```ts
type ChooseArray<A> = A extends string
  ? string[]
  : A extends boolean
  ? boolean[]
  : never;

type StringArray = ChooseArray<string>; // type StringArray =  string[];
type BooleanArray = ChooseArray<boolean>; // type BooleanArray =  boolean;[];
type Never = ChooseArray<number>; // type Never = never;
```

### 인덱스 접근 타입으로 컨디셔널 타입을 사용할 수 있다.

```ts
type A = string;
type B = A extends string ? number : boolean;
type C = {
  t: number;
  f: boolean;
}[A extends string ? "t" : "f"];
```

<br /><br />

## 컨디셔널 타입 분배법칙

컨디셔널 타입, 제네릭과 never의 조합은 더 복잡한 상황에서 진가를 발휘한다. 컨디셔널 타입을 제네릭과 함께 사용하면 아래와 같이 원하는 결과를 얻을 수 있다.

```ts
type Start = string | number;
type Result<K> = K extends string ? K[] : never;

let n: Result<Start> = ["ht"]; // let n = string[];
```

> 검사하려는 타입이 제네릭이면서 유니언이면 분배 법칙이 실행되나 boolean에 분배법칙이 적용될 때는 조심해야 한다.

```ts
type Start = string | number | boolean;
type Result<K> = K extends string | boolean ? K[] : never;

let n: Result<Start> = ["ht"]; // let n: string[] | false[] | true[];
```

> boolean을 true | false 로 인식하기 때문에 string[] | boolean[]과 같은 원하는 결과가 나오지 않는다.

### 배열로 제네릭을 감싸면 분배법칙이 일어나지 않는다.

```ts
type IsString<T> = [T] extends [string] ? true : false;
type Result = IsString<"hi" | 3>; // type Result = false;
```

> 유니언과 제네릭이 만나면서 분배법칙이 실행되어 Result 타입이 boolean으로 나타난다.

    - 현재 코드에서는 ["hi" | 3] 이 [string]을 extends하는 지 검사
    - Result는 false가 된다.

- 타입스크립트는 제네릭이 들어있는 컨디셔널 타입을 판단할 때 값의 판단을 뒤로 미룬다.
- 그 외에 never 또한 분배법칙의 대상이다. never는 공집합과 같으므로 공집합에서 분배법칙을 실행하는 것은 아무것도 실행하지 않는 것과 같다.
- 컨디셔널 타입에서 제네릭 + never => never

<br /><br />

## 함수와 메서드를 타이핑하자

기본값이 제공된 매개변수는 자동으로 옵셔널이 된다. 매개변수는 배열이나 객체처럼 나머지 문법을 사용할 수 있다. 또 이렇게 나머지 매개변수 문법을 사용하는 매개변수는 항상 배열이나 튜플 타입이어야 하는 특징이 있다.

```ts
function A(a: string, ...b: number[]) {}

A("a", 1, 2, 3);

function B(...a: string[], b: number) {} // 나머지 문법은 항상 마지막!
```

### 함수 내부에서 this를 사용하는 경우에는 명시적으로 표기해야 한다.

```ts
function example1() {
  console.log(this);
} // type annotation이 없다는 에러

function example2(this: Window) {
  console.log(this);
} // this = Window

function example3(this: Document, a: string, b: "this") {}
example3("hello", "this"); // The 'this' context of type 'void' is not assignable to method's 'this' of type 'Document'.

example3.call(document, "hello", "this");
```

> 일반적으로는 this가 메서드를 가지고 있는 객체 자신으로 추론되서 타입 명시가 필요없지만 this가 바뀔 수 있을 때는 타이핑해야 한다.

### 타입스크립트는 함수를 생성자로 사용할 수 없다.

대신 `class`를 사용해야 한다.

또 `function`을 생성자로 만드려면 생성자의 타입(`PersonConstructor`)과 인스턴스의 타입(`Person`)을 따로 만들고 생성자 함수도 as known as `PersonConstructor`로 강제로 타입을 지정해야 한다.

```ts
class Person {
  name: string;
  age: number;
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
  sayName() {
    console.log(this.name);
  }
}

const seulgi = new Person("seulgi", 32);
```

<br /><br />
