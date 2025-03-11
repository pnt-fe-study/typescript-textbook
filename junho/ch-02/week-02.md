## 2.10 객체의 속성과 메서드에 적용되는 특징을 알자
- 객체의 속성도 옵셔널이나 readonly 수식어가 가능합니다.
- 객체 리터럴을 대입했냐, 변수를 대입했냐에 따라 타입 검사 방식이 달라집니다.
  - ```ts
    interface Example {
      hello: string;
    }

    // 객체 리터럴
    const example: Example = {
      hello: 'hi',
      why: '나만 에러야',
    }
    // Type '{ hello: string; why: string; }' is not assignable to type 'Example'. Object literal ...

    const obj = {
      hello: 'hi',
      why: '나는 에러 아냐',
    }

    // 변수 대입
    const example2: Example = obj;
    ```
    - 참고로 에러 메시지에는 여러 에러 메시지가 동시에 표시될 수 있습니다.
    - 두 번째 에러가 첫 번째 에러보다 더 구체적인 에러입니다.
    - 에러를 분석할 때 에러 메시지가 여러 개 있다면 위에서 아래로 읽으면서 구체적인 에러의 위치를 찾아나가면 됩니다.
  - 객체 리터럴을 대입하면 `잉여 속성 검사`가 실행됩니다.
  - `잉여 속성 검사`는 타입 선언에서 선언하지 않은 속성을 사용할 때 에러를 표시하는 것을 의미합니다.

### 2.10.1 인덱스 접근 타입
- ```ts
  type Animal = {
    name: string;
  }

  type N1 = Animal['name'];
  // type N1 = string

  type N2 = Animal["name"];
  // type N2 = string
  
  type N3 = Animal.name;
  // Cannot access 'Animal.name' becuase ...
  ```
  - 자바스크립트에서 객체의 속성에 접근하듯 접근합니다.
  - 다만, 객체.속성 방식은 사용할 수 없습니다.
  - 객체 속성의 타입에 접근하는 방식을 `인덱스 접근 타입`이라고 부릅니다.
- 키의 타입은 `keyof` 객체 타입이고 값의 타입은 객체 타입[키의 타입] 입니다.
  - ```ts
    const obj = {
      hello: 'world',
      name: 'zero',
      age: 28,
    };
    type Keys = keyof typeof obj;
    // type Keys = 'hello' | 'name' | 'age'

    type Values = typeof obj[Keys];
    // type Values = string | number
    ```
    - obj는 값이기에 타입 자리에 바로 쓸 수 없습니다.
    - typeof 연산자로 타입을 만들어 주었습니다.
- keyof의 특성 몇 가지를 확인합니다.
  - keyof any는 string | number | symbol 입니다.
  - 배열에 keyof를 적용하면 number | 배열 속성 이름 유니언 | 배열 인덱스 문자열 유니언이 됩니다.
- 튜플과 배열에도 인덱스 접근 타입을 사용할 수 있습니다.
  - ```ts
    type ...

    type Arr2 = (string | boolean)[];
    type El = Arr2[number];
    // type El = string | boolean;
    ```
    - El 타입처럼 배열[number] 인덱스 접근 타입으로 배열 요소들의 타입을 모두 가져올 수 있습니다.
- 객체의 메서드를 선언할 때는 세 가지 방식으로 할 수 있습니다.
  - ```ts
    interface Example {
      a(): void;
      b: () => void;
      c: {
        (): void;
      }
    }
    ```

### 2.10.2 매핑된 객체 타입
- 매핑된 객체 타입이란 기존의 다른 타입으로부터 새로운 객체 속성을 만들어내는 타입을 의미합니다.
- 인터페이스에서는 쓰지 못하고 타입 볊칭에서만 사용할 수 있습니다.
  - ```ts
    type HelloAndHi = {
      [key in 'hello' | 'hi']: string;
    };
    /*
    type HelloAndHi = {
      hello: string;
      hi: string;
    }
    */
    ```
    - in 연산자를 사용해서 인덱스 시그니처가 표현하지 못하는 타입을 표현합니다.
    - in 연산자 오른쪽에는 유니언 타입이 와야 합니다.
- 튜플과 배열도 객체이므로 매핑된 객체 타입을 적용할 수 있습니다.
- 다른 타입으로부터 값을 가져오면서 수식어를 붙일 수도 있습니다.
  - ```ts
    interface Original {
      name: string;
      age: number;
      married: boolean;
    }
    type Copy = {
      readonly [key in keyof Original]?: Original[key];
    }
    

    interface Original {
      name: string;
      age: number;
      married: boolean;
    }
    type Copy = {
      readonly [key in keyof Original]?: Original[key]
    }
    /*
    type Copy = {
        readonly name?: string | undefined;
        readonly age?: number | undefined;
        readonly married?: boolean | undefined;
    }
    */
    ```
- 수식어 앞에 -를 붙이면 해당 수식어가 제거된 채로 속성을 가져옵니다.
  - ```ts
    interface Original {
      readonly name?: string;
      readonly age?: number;
      readonly married?: boolean;
    }
    type Copy = {
      -readonly [key in keyof Original]-?: Original[key];
    }
    /*
    type Copy = {
        name: string;
        age: number;
        married: boolean;
    }
    */
    ```

## 2.11 타입을 집합으로 생각하자(유니언, 인터섹션)
- `유니언(|)` 연산자는 합집합 역할을 합니다.
- `인터섹션(&)` 연산자는 교집합 역할을 합니다.
- 원소가 존재하지 않는 집합은 `공집합`이라고 부릅니다.
  - 타입스크립트에서는 `never`가 `공집합`의 역할을 맡습니다.
    - ```ts
      type nev = string & number;
      //type nev = never;
      ```
- 타입스크립트에서 전체집합은 `unknown`입니다.
- 타입스크립트에서는 좁은 타입을 넓은 타입에 대입할 수 있습니다.
  - 반대로 넓은 타입에서 좁은 타입은 대입할 수 없습니다.
- `항상 좁은 타입에서 넓은 타입으로 대입해야 합니다.`
  - ```ts
    type A = string | boolean;
    type B = boolean | number;
    type C = A & B;
    // type C = boolean

    type D = {} & (string | null);
    // type D = string

    type E = string & boolean;
    // type E = never;

    type F = unknown | {};
    // type F = unknown

    type G = never & {};
    // type G = never
    ```
    - 한 가지 특이한 점이 있습니다. null/undefined를 제외한 원시 자료형과 비어 있지 않은 객체를 & 연산할 때는 never가 되지 않습니다.
      - ```ts
        type H = { a: 'b' } & number;
        // type H = { a: 'b'} & number;
        ```
        
## 2.12 타입도 상속이 가능하다
- interface는 extends 예약어를 사용해 상속할 수 있습니다.
  - ```ts
    interface Animal {
      name: string;
    }

    interface Dog extends Animal {
      bark(): void;
    }
    ```
- 타입 별칭에서 & 연산자를 사용해 상속할 수 있습니다.
  - ```ts
    type Animal = {
      name: string;
    }

    type Dog = Animal & {
      bark(): void;
    }
    ```
- 상속 받는다는 것은 `더 좁은 타입`이 된다는 것을 의미한다.
- 타입 별칭이 인터페이스를 상속할 수도 있고, 인터페이스가 타입 별칭을 상속할 수도 있습니다.
  - ```ts
    interface Animal {
      name: string;
    }
    type Dog = Animal & {
    bark(): void;
    }
    ```
- 한 번에 여러 타입을 상속할 수도 있습니다.
  - ```ts
    type Animal = {
      name: string;
    }
    interface Dog extends Animal {
      bark(): void;
    }
    interface Cat extends Animal {
      meow(): void;
    }

    interface DogCat extends Dog, Cat {}
    ```
- 상속할 때 부모 속성의 타입을 변경할 수도 있습니다.
  - ```ts
    interface Merge {
      one: string;
      two: string;
    }
    interface Merge2 extends Merge {
      one: 'h' | 'w';
      two: 123;
    }
    ```
    - 다만 완전히 다른 타입으로 변경하면 에러가 발생합니다.
    - 부모의 속성 타입을 바꾸더라도 부모에 대입할 수 있는 타입으로 바꿔야 합니다.

## 2.13 객체 간에 대입할 수 있는지 확인하는 법을 배우자
- 추상적일수록 넓은 타입이며 구체적일수록 더 좁은 타입입니다.
  - ```ts
    type A = { name: string };
    type B = { name: string, age: number };
    ```
    - B가 더 구체적인 값을 가지고 있어 A가 더 넓은 타입입니다.
- 튜플은 배열보다 좁은 타입입니다.
- 튜플이나 배열에 readonly 수식어가 있으면 더 넓은 타입입니다.(일반 배열은 getter, setter 가능, readonly 배열은 getter만 가능)
- 객체 속성에 옵셔널이 붙으면 더 넓은 타입입니다.
- 배열과 다르게 객체에서는 속성에 readonly가 붙어도 서로 대입할 수 있습니다.

### 2.13.1 구조적 타이밍
- 타입스크립트에서 모든 속성이 동일하면 객체 타입의 이름이 다르더라도 동일한 타입으로 취급합니다.
  - ```ts
    interface Money {
      amount: number;
      unit: string;
    }

    interface Liter {
      amount: number;
      unit: string;
    }

    const liter: Liter = { amount: 1, unit: 'liter' };
    const circle: Money = liter;
    ```
    - 객체를 어떻게 만들었든 간에 구조가 같으면 같은 객체로 인식하는 것을 구조적 타이핑이라고 부릅니다.
- B 타입이 A 타입이기 위한 조건이 충족되면 구조적 타이핑 관점에서 A 타입이라고 볼 수 있습니다.
  - ```ts
    interface A {
        name: string;
    }
    
    interface B {
        nmae: string;
        age: number;
    }
    
    const aObj = {
        name: 'zero',
    }
    
    const bObj = {
        name: 'zero',
        age: 32
    }
    
    const bToA: A = bObj;
    ```
- 구조적으로 다르게 만들기 위해 브랜드 속성을 사용하여 브랜딩 합니다.
  - ```ts
    interface Money {
      __type: 'money';
      amount: number;
      unit: string;
    }

    interface Liter {
      __type: 'liter';
      amount: number;
      unit: string;
    }

    const liter: Liter = { amount: 1, unit: 'liter', __type: 'liter' };
    const circle: Money = liter;
    // Type 'Liter' is not assignable to type 'Money'. Types of property ...
    ```

## 2.14 제네릭으로 타입을 함수처럼 사용하자
- 제네릭을 사용해서 중복을 제거할 수 있습니다.
  - ```ts
    interface Person<N, A> {
      type: 'human',
      race: 'yellow',
      name: N,
      age: A,
    }
    interface Zero extends Person<'zero', 28> {}
    interface Nero extends Person<'nero', 32> {}
    ```
- 타입 매개변수의 개수와 타입 인수의 개수가 일치하지 않으면 에러가 발생합니다.
- 클래스와 타입 별칭, 함수도 제네릭을 가질 수 있습니다.
  - 함수에서는 함수 선언문이냐 표현식이냐에 따라 제네릭 표기 위치가 다르므로 주의해야 합니다.
- interface와 type 간에 교차 사용도 가능합니다. 
- 정리하면 다음과 같은 위치에서 사용할 수 있습니다.
  - interface 이름<타입 매개변수들> {}
  - type 이름<타입 매개변수들> = {}
  - class 이름<타입 매개변수들> {}
  - function 이름<타입 매개변수들>() {}
  - const 이름 = <타입 매개변수들>() => {}
- 타입 매개변수는 기본값을 사용할 수 있습니다.
  - ```ts
    // 타입 매개변수 기본값 설정
    type Box<T = string> = {
      value: T;
    };
    
    // 기본값 사용
    const stringBox: Box = { value: "Hello" }; // T는 기본적으로 string으로 설정됨
    
    // 기본값 덮어쓰기
    const numberBox: Box<number> = { value: 123 }; // T를 number로 명시적으로 설정
    ```
- 타입스크립트는 제네릭에 직접 타입을 넣지 않아도 추론을 통해 타입을 알아낼 수 있습니다.
  - ```ts
    function values<const T>(initial: T[]) {
      return {
          hasValue(value: T) { return initial.includes(value) }
      };
    }
      
    const saveValues = values(["a", "b", "c"]);
      
    saveValues.hasValue("x");
    // Argument of type '"x"' is not assignable to parameter of type '"a" | "b" | "c"'.
    ```
    - 타입 매개변수 앞에 const 수식어를 추가하면 타입 매개변수 T를 추론할 때 as const를 붙인 값으로 추론합니다.

### 2.14.1 제네릭에 제약 걸기
- 타입 매개변수에 extends 문법으로 제약을 사용할 수 있습니다.
  - ```ts
    interface Example<A extends number, B = string> {
        a: A,
        b: B,
    }
    
    type Usecase1 = Example<string, boolean>;
    // Type 'string' does not satisfy the constraint 'number'.
    
    type Usecase2 = Example<1, boolean>;
    type Usecase3 = Example<number>;
    ```
    - 특정 타입 매개변수에 제약이 걸리면 제약에 어긋나는 타입은 입력할 수 없지만 제약보다 더 구체적인 타입은 입력할 수 있습니다.
- 하나의 타입 매개변수가 다른 타입 매개변수의 제약이 될 수 있습니다.
  - ```ts
    interface Example<A, B extends A> {
        a: A,
        b: B,
    }
    
    type Usecase1 = Example<string, number>;
    // Type 'number' does not satisfy the constraint 'string'.
    type Usecase2 = Example<string, 'hello'>;
    type Usecase3 = Example<number, 123>;
    ```
- 제네릭을 사용할 때 타입 매개변수와 제약을 동일하게 생각하는 실수를 흔하게 합니다.
  - ```ts
    interface V0 {
        value: any;
    }
    
    const returnV0 = <T extends V0>(): T => {
        return { value: 'test' };
    }
    
    // Type '{ value: string; }' is not assignable to type 'T'...
    ```
    - T의 값이 { value: any, another: string }이 될 수 있습니다.
    - 
  - ```ts
    function onlyBoolean<T extends boolean>(arg: T = false): T {
        return arg;
    }
    
    // Type 'boolean' is not assignable to type 'T'...
    ```
    - T는 true 또는 false가 될 수 있습니다.
    - ```ts
      onlyBoolean<true>(true);  // ✅ OK
      onlyBoolean<false>(false); // ✅ OK
      onlyBoolean<boolean>(true);  // ✅ OK
      onlyBoolean<boolean>(false); // ✅ OK

      onlyBoolean<true>(); // ❌ 에러 발생!
      ```
      - 모순이 발생할 수 있습니다.

## 2.15 조건문과 비슷한 컨디셔널 타입이 있다
> 특정 타입 extends 다른 타입 ? 참 타입 : 거짓 타입
  - ```ts
    type A1 = string;
    type B1 = A1 extends string ? number : boolean;
    // type B1 = number
    ```
- extends 에약어가 여러 곳에서 사용되어 헷갈릴 수 있습니다. 명시적으로 extends를 사용해야 참이 되는 것은 아닙니다.
  - ```ts
    interface X {
        x: number;
    }
    
    interface XY {
        x: number;
        y: number;
    }
    
    interface YX extends X {
        y: number;
    }
    
    type A = XY extends X ? string : number;
    // type A = string
    
    type B = YX extends X ? string : number;
    //type B = string
    ```
- never는 모든 타입에 대입할 수 있어 모든 타입을 extends할 수 있습니다.
  - ```ts
    type Result = never extends string ? true : false;
    // type Result = true
    ```
- never와 컨디셔널 타입을 사용해 특정 타입 속성을 제거할 수 있습니다.
  - ```ts
    type OmitByType<O,T> = {
        [K in keyof O as O[K] extends T ? never : K]: O[K];
    }
    
    type Result = OmitByType<{
        name: string;
        age: number;
        married: boolean;
        rich: boolean;
    }, boolean>;
    /*
    type Result = {
        name: string;
        age: number;
    }
    */
    ```
    - keyof O를 통해 객체 O의 모든 속성 키(K)를 가져옵니다.
    - O[K] extends T ? never : K 에서 속성의 타입이 T 이면 never 가 됩니다.
    - never로 변한된 속성은 자동으로 삭제됩니다.
    - 남아 있는 속성들만 새로운 객체 타입으로 반환됩니다.
- 컨디셔널 타입을 자바스크립트의 삼항연산자처럼 중첩해서 사용할 수 있습니다.
  - ```ts
    type ChooseArray<A> = A extends string 
    ? string[] 
    : A extends boolean ? boolean[] : never;

    type StringArray = ChooseArray<string>;
    // type StringArray = string[]
    
    type BooleanArray = ChooseArray<boolean>;
    // type BooleanArray = boolean[]
    ```
- 인덱스 접근 타입으로 컨디셔널 타입을 사용할 수 있습니다.
  - ```ts
    type A1 = string;
    type B1 = A1 extends string ? number : boolean;
    type B2 = {
        't': number;
        'f': boolean;
    }[A1 extends string ? 't' : 'f'];
    ```
### 2.15.1 컨디셔널 타입 분배법칙
- 검사하는 타입이 제네릭이면서유니언이면 분배법칙이 실행됩니다.
  - ```ts
    type Start = string | number;
    type Result<K> = K extends string ? K[] : never;
    let n: Result<Start> = ['ht'];
    // let n = string[]
    ```
- boolean에 분배법칙이 적용될 때는 조심해야 합니다.
  - ```ts
    type Start = string | number | boolean;
    type Result<K> = K extends string | boolean ? K[] : never;
    let n: Result<Start> = ['ht'];
    // let n: string[] | false[] | true[]
    ```
    - string[] | boolean[]이 될 것이라는 예상과 달리 string[] | false[] | true[] 가 됩니다.
    - boolean을 true | false 로 인식하기 때문입니다.
- 분배법칙이 일어나는 것을 막고 싶을 수도 있습니다.
  - ```ts
    // 분배법칙으로 발생하는 문제
    type IsString<T> = T extends string ? true : false;
    type Result = IsString<'hi' | 3>;
    // type Result = boolean
    ```
    - Result 타입이 false를 예상했지만, boolean으로 나옵니다.
    - 분배법칙으로 true | false가 나와 boolean이 됩니다.
  - ```ts
    type IsString<T> = [T] extends [string] ? true : false;
    type Result = IsString<'hi' | 3>;
    // type Result = false
    ```
    - 배열로 제네릭을 감싸면 분배법칙이 일어나지 않습니다.
    - 'hi' | 3 과 string을 비교하여 false가 됩니다.
    - 추가로 string에 괄호를 제거하면 어떻게 될까요?
      - ```ts
        type IsString<T> = [T] extends string ? true : false;
        type Result = IsString<'hi' | 3>;
        // type Result = false
        ```
        - ([ 'hi' ] extends string ? true : false) | ([ 3 ] extends string ? true : false)
        - false | false => false
- 컨디셔널 타입에서 제네릭과 never가 만나면 never가 됩니다.
  - ```ts
    ttype R<T> = T extends string ? true : false;
    type RR = R<never>;
    // type RR = never
    ```
- 타입스크립트는 제네릭이 들어있는 컨디셔널 타입을 판단할 때 값의 판단을 뒤로 미룹니다.
  - ```ts
    function test<T>(a: T) {
        type R<T> = T extends string ? T : T;
        const b: R<T> = a;
    }
    // Type 'T' is not assignable to type 'R<T>. 
    ```
    - 변수 b에 매개변수 a를 대입할 때까지 타입스크립트는 R<T>가 T라는 것을 알지 못합니다.

## 2.16 함수와 메서드를 타이핑하자
- 기본값이 제공된 매개변수는 자동으로 옵셔널이 됩니다.
- 나머지 매개변수 문법을 사용하는 매개변수는 항상 배열이나 튜플 타입이어야 합니다.
- 함수 내부에서 this를 사용하는 경우에는 명시적으로 표기해야 합니다.
  - ```ts
    function example1() {
        console.log(this);
    }
    // 'this' implicitly has type 'any' because it does not have a type annotation.
    
    function example2(this: Window) {
        console.log(this);
    }
    // this: Window
    
    function example3(this: Document, a: string, b: 'this') {}
    example3('hello', 'this');
    // The 'this' context of type 'void' is not assignable to method's 'this' of type 'Document'.
    
    example3.call(document, 'hello', 'this');
    ```
    - 매개변수 첫 번째 자리에 this 타입을 표기해야 합니다.
    - this 타입을 표기했다고 해서 this를 쓸 수 있는 것은 아닙니다.
    - this가 Document 타입일 수 없음을 알고 있으며 call 메서드 등을 활용해 this의 값을 명시적으로 document로 지정해주어야 합니다.
- 메서드의 this는 일반적으로 객체 자신을 추론하지만 this가 바뀔 수 있을 때는 명시적으로 타이핑해주어야 합니다.
- 타입스크립트에서는 기본적으로 함수를 생성자로 사용할 수 없습니다.

## 2.17 같은 이름의 함수를 여러 번 선언할 수 있다
- 함수의 타입을 미리 여러 개 타이핑해두는 기법을 오버로딩이라고 합니다.
- 오버로딩을 선언하는 순서가 타입 추론에 영향을 끼칩니다.
  - ```ts
    function example(param: string): string;
    function example(param: string | null): number;
    function example(param: string | null): string | number {
        if (param) {
            return 'string';
        } else {
            return 123;
        }
    }
    
    const result = example('what');
    // const result: string
    ```
    - 'what'은 string 이므로 첫 번째 오버로딩과 두 번째 오버로딩에 해당 됩니다.
    - 오버로딩이 동시에 해당할 경우 먼저 선언된 오버로딩을 따릅니다.
  - ```ts
    function example(param: string | null): number;
    function example(param: string): string;
    function example(param: string | null): string | number {
        if (param) {
            return 'string';
        } else {
            return 123;
        }
    }
    
    const result = example('what');
    // const result: number
    ```
    - 오버로딩의 순서를 바꾸면 result가 number가 됩니다.
    - 다만 실제 result의 반환값은 'string'이므로 에러가 발생합니다.
    - `오버로딩의 순서는 좁은 타입부터 넓은 타입순`으로 오게 해야 문제가 없습니다.
- 인터페이스로 오버로딩을 표현할 수 있습니다.
  - ```ts
    interface Add {
        (x: number, y: number): number;
        (x: string, y: string): string;
    }
    
    const add: Add = (x: any, y: any) => x + y;
    ```
- 타입별칭으로 오버로딩을 표현할 수 있습니다.
  - ```ts
    type Add1 = (x: number, y: number) => number;
    type Add2 = (x: string, y: string) => string;
    type Add = Add1 & Add2;
    const add: Add = (x: any, y: any) => x + y;
    ```
- 지나치게 오버로딩을 활요하면 안 됩니다.
  - ```ts
    function a(param: string): void // 오버로드 시그니처(선언부)
    function a(param: number): void 
    function a(param: string | number) {} // 구현 시그니처(구현부)
    
    function errorA(param: string | number) {
        a(param);
    }
    // No overload matches this call...
    ```
    - 타입 검사는 오직 오버로드 시그니처만 기준으로 수행됩니다.
    - 선언부에 string | number 타입이 없어 에러가 발생합니다.
      
## 2.18 콜백 함수의 매개변수는 생략 가능하다
- ```ts
  function example(callback: (error: Error, result: string) => void) {}
  example((e, r) => {});
  example(() => {});
  example(() => true);
  ```
  - 인수로 제공하는 콜백 함수의 매개변수에는 타입을 표기하지 않아도 됩니다.
  - 이러한 현상을 문맥적 추론이라고 부릅니다.

## 2.19 공변성과 반공변성을 알아야 함수끼리 대입할 수 있다
- 공변성: A -> B 일 때 T<A> -> T<B>인 경우
- 반공변성: A -> B 일 때 T<B> -> T<A>인 경우
- 이변성: A -> B 일 때 T<A> -> T<B>도 되고 T<B> -> T<A>도 되는 경우
- 무공변성: A -> B 일 때 T<A> -> T<B>도 안 되고 T<B> -> T<A>도 안 되는 경우
- 함수의 반환값은 공변성을 가집니다.(좁은 타입을 넓은 타입에 대입할 수 있다)
- 함수의 매개변수는 반공변성을 가집니다.(넓은 타입을 좁은 타입에 대입할 수 있다)

## 2.20 클래스는 값이면서 타입이다
- implements 예약어를 사용하면 더 엄격하게 클래스 멤버를 검사할 수 있습니다.
  - ```ts
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
    }
    // Class 'Person' incorrectly implements interface 'Human'...
    ```
    - sayName 메서드를 구현하지 않아 에러가 발생합니다.
- 클래스의 자체 타입이 필요하다면 `typeof 클래스이름`으로 사용해야 합니다.
- 클래스 멤버에 옵셔널(?), readonly, public, protected, private 수식어가 가능합니다.

| 접근 제한자  | 자기 클래스 | 자식 클래스 | 인스턴스 |
|-------------|------------|------------|----------|
| `public`    | ✅ 가능   | ✅ 가능   | ✅ 가능 |
| `protected` | ✅ 가능   | ✅ 가능   | ❌ 불가능 |
| `private`   | ✅ 가능   | ❌ 불가능 | ❌ 불가능 |

- private과 #의 차이점은 private으로 선언한 속성은 자식 클래스에서 같은 이름으로 선언할 수 없다는 점입니다.
  - ```ts
    class PrivateMember {
        private priv: string = 'priv';
    }
    
    class ChildPrivateMember extends PrivateMember {
        private priv: string = 'priv';
    }
    // Class 'ChildPrivateMember' incorrectly extends base class 'PrivateMember'...
    
    class PrivateField {
        #priv: string = 'priv';
    
        sayPriv() {
            console.log(this.#priv);
        }
    }
    
    class ChildPrivateField extends PrivateField {
        #priv: string = 'priv';
    }
    
    class A extends PrivateMember {
        #priv: string = 'ase';
    }
    
    class B extends PrivateField {
        private priv: string = 'priv';
    }
    ```
- implements에 속하는 속성은 전부 public이어야 합니다.
- 명시적으로 오버라이드할 때는 앞에 override 수식어를 사용해야 합니다.
  - ```ts
    class Human {
        eat() {
            console.log("냠냠")
        }
        sleep() {
            console.log("쿨쿨")
        }
    }
    
    class Employee extends Human {
        worK() {
            console.log("끙차")
        }
        override sleep() {
            console.log("에고고")
        }
    }
    ```
    - override 수식어를 사용하면 부모 클래스의 메서드를 바꿀 때 확인할 수 있다는 장점이 있습니다.
    - 부모 클래스의 메서드를 실수로 변경했거나 오타를 쉽게 확인할 수 있습니다.
- 클래스의 생성자 함수에도 오버로딩을 적용할 수 있습니다.
  - ```ts
    class Person {
        name?: string;
        age?: number;
        married?: boolean;
        constructor();
        constructor(name: string, married: boolean);
        constructor(name: string, age: number, marreid: boolean);
        constructor(name?: string, age?: number | boolean, married?: boolean) {
            if (name) {
                this.name = name;
            }
    
            if (typeof age === 'boolean') {
                this.married = age;
            } else {
                this.age = age;
            }
    
            if (married) {
                this.married = married;
            }
        }
    }
    
    const person1 = new Person();
    const person2 = new Person('nero', true);
    const person3 = new Person('zero', 29, false);
    ```
