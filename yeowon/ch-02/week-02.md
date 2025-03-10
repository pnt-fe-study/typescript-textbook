## 2-10 객체의 속성과 메서드에 적용되는 특징을 알자
- 인터페이스로 선언했든, 타입 별칭으로 선언했든 상관없이 객체의 속성에 공통적으로 적용된다.
- 객체의 속성에도 옵셔널이나 readonly 수식어가 가능하다.
- 잉여속성검사: 타입 선언에서 선언하지 않은 속성을 사용할 때 에러를 표시하는 것
- 객체 리터럴을 대입하면 잉여속성검사가 실행된다.
- 객체 리터럴을 대입했냐, 변수를 대입했냐에 따라 타입 검사 방식이 달라진다.
  ```ts
  interface Example {
    hello: string;
  }

  const example: Example = {
    hello: 'hi',
    why: 'error',
  }

  const obj = {
    hello: 'hi',
    why: 'no error'
  }

  const ex2:Example = obj
  ```
  - 에러 메시지는 여러개가 동시에 표시될 수 있다.
  - 'why'속성이 Example 타입에 존재하지 않는다는 에러가 잉여속성검사의 에러보다 구체적인 에러이다.
- 함수에서도 같은 현상이 발생한다. 인수자리에 변수로 값을 대입하면 에러가 발생하지 않지만, 객체 리터럴을 대입하면 에러가 발생한다.

### 2.10.1 인덱스 접근 타입
- 특정 속성의 타입을 별도 타입으로 만들고 싶다면 어떻게 해야 할까?
  ```ts  
  type Animal = {
    name: string;
  }

  type N1 = Animal['name']; // type N1 = string
  type N2 = Animal['name']; // type N2 = string
  type N3 = Animal.name; // error
  ```
  - 자바스크립트에서 객체의 속성에 접근하듯 접근하되, '객체.속성'꼴의 방식은 사용할 수 없다.
  - 위 코드와 같은 방식을 인덱스 접근 타입 이라고 부른다.

- keyof 연산자와 인덱스 접근 타입을 활용해 키의 타입과 값의 타입을 구할 수 있다.
  ```ts
  const obj = {
    hello: 'world',
    name: 'yeowon',
    age: 31,
  }

  type Keys = keyof typeof obj; // type key = 'hello' | 'name' | 'age'
  type Values = typeof obj[Keys]; // type Values = string | number
  ```
  - 키의 타입은 'keyof 객체_타입' 이고 값의 타입은 '객체_타입[키의_타입]' 이다.
  - keyof의 특성
    - keyof any는 string | number | symbol이며 타입스크립트에서는 배열을 위해 number타입의 키를 허용.
    - 배열에 keyof를 적용하면 number | 배열 속성 이름 유니언 | 배열 인덱스 문자열 유니언이 된다.
  - 튜플과 배열에도 인덱스 접근 타입을 사용할 수 있다.
  - 인덱스 접근 타입을 활용하여 특정 키들의 값 타입만 추릴 수 있다.
  - 객체의 메서드를 선언할 때에는 세 가지 방식으로 할 수 있다.
    - 메서드(매개변수): 반환값
    - 메서드: (매개변수) => 반환값
    - 메서드: { (매개변수): 반환값 }

### 2.10.2 매핑된 객체 타입
- 속성 전부에 타입을 지정하는 대신 일부 속성에만 타입을 부여할 수 있다.
- 매핑된 객체 타입: 기존의 다른 타입으로부터 새로운 객체 속성을 만들어내는 타입을 의미(인터페이스에서는 못쓰고 타입별칭에서만 가능)
  ```ts
  type HelloAndHi = {
    [key in 'hello' | 'hi']: string;
  }
  
  // 위 코드는 아래의 의미와 같음
  type HelloAndHi = {
    hello: string;
    hi: string
  }
  ```
  - in 연산자를 사용해 인덱스 시그니처가 표현하지 못하는 타입(string, number, symbol, 템플릿 리터럴 타입 외)을 표현함
  - in 연산자 오른쪽에는 유니언타입이 와야 함
  - 유니언 타입에 속한 타입이 하나씩 순서대로 평가되어 객체의 속성이 된다.
- 튜플과 배열도 객체이므로 매핑된 객체 타입을 적용할 수 있다.
- 다른 타입으로부터 값을 가져오면서 수식어(readonly, 옵셔널)를 붙일 수 있다.
- 수식어 앞에 - 를 붙여 제거할 수도 있다.

## 2.11 타입을 집합으로 생각하자 (유니언, 인터섹션)
- 유니언 연산자는 실제로 합집합 역할을 하며, & 연산자는 교집합 역할을 한다.
- 타입스크립트에서 원소가 존재하지 않는 공집합의 역할을 하는 것은 never이다.
- unknown은 전체집합으로 벤다이어 그램에서 가장 넓은 영역을 차지한다.
- 항상 좁은 타입에서 넓은 타입으로 대입해야 한다.
- {}타입은 객체를 의미하는것이 아니라 null, undefined를 제외한 모든 값을 의미한다.

## 2.12 타입도 상속이 가능하다.
- 타입스크립트에서 객체 타입 간 상속하는 방법은 extends예약어를 사용하거나 타입 별칭에서는 & 연산자를 사용한다.
- 상속 받는다는 것은 더 좁은타입이 된다는 것을 의미한다.
  ```ts
  interface Animal {
    name: string;
  }

  interface Dog extends Animal {
    bark(): void;
  }

  type Animal = {
    name: string;
  }

  type Dog = Animal & {
    bark(): void;
  }
  ```
  - 타입 별칭이 인터페이스를 상속할 수 있고, 인터페이스가 타입 별칭을 상속할 수도 있다.

## 2.13 객체 간에 대입할 수 있는지 확인하는 법을 배우자
- 좁은 타입 -> 넓은 타입 대입가능, 넓은 타입 -> 좁은 타입 대입 불가
- 넓은 타입이라는 것은 더 추상적인 타입이라는 뜻이고 좁은 타입이라는 뜻은 더 구체적이라는 뜻이다.
- 합집합은 각각의 집합이나 교집합보다 넓은 타입이다.
- 튜플은 배열보다 좁은 타입이다
  ```ts
  let a: ['hi', 'readonly'] = ['hi', 'readonly'];
  let b: string[] = ['hi', 'normal'];

  a = b; // error: 배열은 튜플에 대입할 수 없다
  b = a; // 튜플은 배열에 대입이 가능하다
  ```
- 배열이나 튜플에 readonly 수식어를 붙일 수 있으며 readonly 수식어가 붙은 배열이 더 넓은 타입이다.
   ```ts
  let a: readonly string[] = ['hi', 'readonly'];
  let b: string[] = ['hi', 'normal'];

  a = b; // string[]이 readonly string[]보다 더 좁은 타입이므로 가능
  b = a; // error: readonly string[]이 string[]보다 넓은 타입이므로 대입 불가
  ```
- 튜플을 배열에 대입하려는 경우, 튜플에 readonly수식어가 붙는다면 일반 배열보다 넓은 타입이 되므로 대입이 불가하다.
- 각 객체의 속성이 동일할 때 옵셔널 속성의 객체가 있다면 더 넓은 타입이다.

### 2.13.1 구조적 타이핑
- 모든 속성이 동일하다면 객체 타입의 이름이 다르더라도 동일한 타입으로 취급한다는 것을 구조적 타이핑이라고 한다.
  ```ts
  interface A {
    name: string;
  }

  interface B {
      nmae: string;
      age: number;
  }
  
  const aObj = {
      name: 'yeowon',
  }
  
  const bObj = {
      name: 'yeowon',
      age: 31
  }

  const bToA: A = bObj;
  ```
   - B인터페이스가 A인터페이스이기 위한 모든 조건이 충족되면 구조적 타이핑 관점에서 B인터페이스는 A인터페이스라고 볼 수 있다.
- 서로 대입하지 못하게 하기 위해서는 객체를 구별할 수 있는 브랜드 속성을 만든다
  ```ts
  interface Money {
    __type: 'money';
    amount: number;
    unit: string;
  }

  interface Liter {
    __tpye: 'liter';
    amount: number;
    unit: string;
  }

  const liter: Liter = { amount:1, nuit:'liter', __tpye: 'liter' }
  const circle: Money = liter // __type속성이 다르므로 대입되지 않는다.
  ```

## 2.14 제네릭으로 타입을 함수처럼 사용하자
- 타입 간 중복이 발생할 때 제네릭을 사용하여 중복을 제거할 수 있다.
    ```ts
    interface Yeowon {
      type: 'human',
      race: 'yellow',
      name: 'yeowon',
      age: 31
    }

    interface Zero {
      type: 'human',
      race: 'yellow',
      name: 'zero',
      age: 28
    }

    interface Person<N, A> {
      type: 'human',
      race: 'yellow',
      name: N,
      age: A
    }
    interface Yeowon extends Person<'Yeowon', 31> {}
    interface Zero extends Preson<'zero', 28> {}
    ```
   - 함수의 매개변수에 호출할 떄 넣은 인수가 대응되는것과 유사하게 <> 안에 타입 매개변수를 넣으면 된다.
   - 타입 매개변수의 개수와 타입 인수의 개수가 일치하지 않으면 에러가 발생한다.
   - 인터페이스뿐만 아니라 클래스와 타입별칭, 함수도 제네릭을 가질 수 있다.
     - interface 이름<타입 매개변수들>{...}
     - type 이름<타입 매개변수들> = {...}
     - class 이름<타입 매개변수들> {...}
     - function 이름<타입 매개변수들>(...) {...}
     - const 이름 = <타입 매개변수들>(...) => {...}
- 타입 매개변수에는 기본값을 사용할 수 있다.
- 타입스크립트는 제네릭에 직접 타입을 넣지 않아도 추론을 통해 타입을 알아낼 수 있다.

### 2.14.1 제네릭에 제약걸기
- 타입 매개변수에는 제약을 사용할 수 있다. 타입의 상속을 의미하던 extends와는 사용법이 다르다.
  ```ts
  interface Example<A extends number, B = string>{
    a: A,
    b: B,
  }
  type Usecase1 = Example<string, string> // error: 제약을 충족시키지 못함
  type Usecase2 = Example<1, boolean> // 제약보다 구체적인 타입 입력 가능
  ```
  - 지정한 타입과완전히 다른 타입을 제공할수는 있지만, 제약에 어긋나는 타입은 제공할 수 없다.
- 제네릭에 제약을 사용할 떄 흔이 하는 실수로 타입 매개변수와 제약을 동일하게 생각하는 것
  ```ts
  function onlyBoolean<T extends boolean>(arg: T = false): T {
    return arg;
  } // error:T는 boolean값이 아니다.
  ```
   - T는 정확히 boolean값이 아니라 boolean값에 대입할 수 있는 모든 타입을 의미한다. 따라서 never도 가능하기 때문에 error
  ```ts
  function onlyBoolean(arg: true | false = true): true | false {
    return arg;
  }
  ```
  - 제네릭을 쓰지 않고 위와 같이 만들면 onlyBoolean함수를 유효하게 만들 수 있다.
 
## 2.15 조건문과 비슷한 컨디셔널 타입이 있다.
- 컨티셔널 타입: 조건에 따라 다른 타입이 되는 것
- 타입 검사를 위해서도 많이 사용된다.
  ```ts
  type Result = 'hi' extends string? true : false; // type Result = true
  type Result2 = [1] extends [string] ? true : false;  type Result2 = false
  ```
### 2.15.1 컨디셔널 타입 분배법칙
- 검사하려는 타입이 제네릭이면서 유니언이면 분배법칙이 실행된다.
```ts
  // string | number 타입이 있을 때 string[] 타입 얻기
  type Start = string | number;
  type Result<Key> = Key extends string ? Key[] : never;
  let n: Result<Start> = ['hi']; // let n: string[]
```
  - 분배법칙이 실행되면 Result<string | number>는 Result<string> | Result<number>가 된다.
  - 다만 boolean일 경우 true|false로 인식하기 때문에 string[] | boolean[]이 되지 않고 string[] | false[] | true[]가 된다.
- 분배법칙 일어나는것 막기
  ```ts
    // 'hi' | 3 dl string 인지 검사하는 타입
    type IsString<T> = T extends string ? true : false;
    type Result = IsString<'hi' | 3> // type Result = boolean

    // 분배법칙 막기
    type IsString<T> = [T] extends [string] ? true : false;
    type Result = IsString<'hi' | 3>; // type Result = false;
  ```
  - 유니언과 제네리깅 만나면서 분배법칙이 실행되어 Result 타입이 boolean으로 나타남.
  - 배열로 제네릭을 감싸면 분배법칙이 일어나지 않는다.
 
## 2.16 함수와 메서드를 타이핑하자
### 나머지 매개변수 [ ...(나머지) ]
- 매개변수는 항상 배열이나 튜플 타입이어야 한다. 나머지 매개변수를 한데 묶는 것이라 배열 꼴일 수밖에 없다.
- 나머지 매개변수 문법은 전개 문법과는 달리 매개변수의 마지막 자리에만 위치해야 한다.
- 매개변수 자리에 전개 문법을 사용할 수도 있다.
  ```ts
  function exam(...args: [number, string, boolean]) {} // 튜플 타입 전개
  exam(1, '2', false);

  function exam2(...args: [a: number, b: string, c:boolean]) {} // 각 매개변수의 이름 직접 정하고 싶다면 이렇게
  ```
- 매개변수에 구조분해 할당을 적용할 때 헷갈리는 문법 타이핑
  ```ts
  // 잘못된 문법 : nested속성을 string타입으로 표기한 것이 아니라 string변수로 이름을 바꾼것
  function destructuring({props: {nested: string}}) {} 
  destructuring({props: {nested: 'hi'}});

  // 올바른 문법
  function destructuring({props: {nested}}: {props: {nested:string}}) {}
  destructuring({props: {nested: 'hi'}})
  ```
- 함수 내부에서 this를 사용하는 경우 명시적으로 표기해야 한다. 그렇지 않으면 any로 추론되고 error가 발생한다.
- 타입스크립트에서는 기본적으로 함수를 생성자로 사용할 수 없다. 억지로 만들 순 있으나 부자연스러운 방법이고 대신 class를 권장 한다.
  ```ts
  class Person {
    name: string;
    age: number;
    married: boolean;
    constructor(name: string, age: number, married: boolean) {
      this.name = name;
      this.age = age;
      this.married = married;
    }
    sayName() {
      console.log(this.name);
    }
  }
  const zero = new Person('zero', 28, false);
  
  ```




