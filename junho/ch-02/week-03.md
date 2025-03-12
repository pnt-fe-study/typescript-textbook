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
    - 일반 함수와 비슷하게 타입 선언을 여러 번 하면 됩니다.
    - 함수의 구현부는 한 번만 나와야 하고, 구현부에서 여러 번 타입 선언한 것들에 대해 모두 대응할 수 있어야 합니다.
- 클래스나 인터페이스의 메서드에서는 this를 타입으로 사용할 수 있습니다.
  - ```ts
    class Person {
        age: number;
        married: boolean;
        constructor(age: number, married: boolean) {
            this.age = age;
            this.married = married;
        }
    
        sayAge() {
            console.log(this.age);
        }
        // this: this
    
        sayMarried(this: Person) {
            console.log(this.married);
        }
        // this: Person
    
        sayCallback(callback: (this: this) => void) {
            callback.call(this);
        }
        // this: this
    }
    ```
    - 기본적으로 this는 클래스 자신이지만, sayMarried 메서드처럼 명시적으로 this를 타이핑할 수 있습니다.
    - sayCallback에서 콜백 함수의 this는 콜백 함수의 this 타입이 Person 인스턴스가 됩니다.
      - ```ts
        class A {
            callbackWithThis(cb: (this: this) => void) {
                cb.call(this);
            }
        
            callbackWithoutThis(cb: () => void) {
                cb();
            }
        }
        
        new A().callbackWithThis(function() {
            this;
        })
        // this: A
        
        new A().callbackWithoutThis(function() {
            this;
        })
        // 'this' implicitly has type 'any' because it does not have a type annotation.
        ```
        - 콜백 함수에서 this를 사용하고 싶다면 this를 타이핑해야 하고, 그 this가 클래스 자신이라면 this: this로 타이핑하면 됩니다.

## 2.20.1 추상 클래스
- 추상 클래스는 implements보다 조금 더 구체적으로 클래스의 모양을 정의하는 방법입니다.
  - ```ts
    abstract class AbstractPerson {
        name: string;
        age: number;
        married: boolean = false;
        abstract value: number;
    
        constructor(name: string, age: number, married: boolean) {
            this.name = name;
            this.age = age;
            this.married = married;
        }
    
        sayName() {
            console.log(this.name);
        }
    
        abstract sayAge(): void;
        abstract sayMarried(): void;
    }
    
    class RealPerson extends AbstractPerson {
        sayAge() {
            console.log(this.age);
        }
    }
    // Non-abstract class 'RealPerson' is missing implementations for the following members of 'AbstractPerson': 'value', 'sayMarried'.
    ```
    - 속성과 메서드가 abstract인 경우 실제 값은 없고 타입 선언만 되어 있습니다.
    - RealPerson 클래스는 AbstractPerson 클래스를 상속하며, 이때 반드시 abstract 속성이나 메서드를 구현해야 합니다.
- implements와 다르게 abstract 클래스는 실제 자바스크립트 코드로 변환됩니다.

## 2.21 enum은 자바스크립트에서도 사용할 수 있다
- enum은 자바스크립트에는 없는 타입이지만, 자바스크립트의 값으로 사용할 수 있는 특이한 타입입니다.
- enum은 여러 상수를 나열하는 목적으로 사용합니다.
  - ```ts
    enum Level {
    NOVICE,
    INTERMEDIATE,
    ADVANCED,
    MASTER
    }  
    ```
- 문자열도 할당 가능합니다.
- enum 타입의 속성은 값으로도 활용할 수 있습니다.
- enum은 타입 안전성을 완전히 보장하지 않습니다.
  - ```ts
    enum Role {
    USER,
    GUEST,
    ADMIN
    }

    console.log(Role[3]);
    // undefined
    ```
- const enum을 사용하면 자바스크립트를 생성하지 않습니다.
  - ```ts
    const enum Money {
        WON,
        DOLLAR
    }
    
    Money.WON;
    // (enum member) Money.WON = 0
    
    Money[Money.WON];
    // A const enum member can only be accessed using a string literal.
    ```

## 2.22 infer로 타입스크립트의 추론을 직접 활용하자
- infer 예약어는 타입스크립트의 타입 추론 기능을 극한까지 활용하는 기능입니다.
- 컨디셔널 타입과 함꼐 사용합니다.
  - ```ts
    // 배열이 있을 때 배열의 요소 타입을 얻어내고 싶은 상황
    type El<T> = T extends (infer E)[] ? E : never;

    type Str = El<string[]>
    // type Str = string
    
    type NumOrBool = El<(number | boolean)[]>
    // type NumOrBool = number | boolean
    ```
    - 컨디셔널 타입에서 타입 변수는 참 부분에서만 사용 가능합니다.(거짓에서 사용 시 에러 발생)
- 서로 다른 타입 변수를 여러 개 동시에 사용할 수 있습니다.
  - ```ts
    type MyPAndR<T> = T extends (...args: infer P) => infer R ? [P, R] : never;
    type PR = MyPAndR<(a: string, b: number) => string>
    // type PR = [[a: string, b: number], string]
    ```
- 같은 타입 변수를 여러 곳에 사용할 수 있습니다.
  - ```ts
    type Union<T> = T extends {a: infer U, b: infer U} ? U : never;
    type Result = Union<{a: 1 | 2, b: 2 | 3}>;
    // type Result = 1 | 2 | 3
    
    type Intersection<T> = T extends {
        a: (pa: infer U) => void,
        b: (pb: infer U) => void,
    } ? U : never;
    type Result2 = Intersection<{ a(pa: 1 | 2): void, b(pb: 2 | 3): void}>
    // type Result2 = 2
    ```
    - 기본적으로 같은 이름의 타입 변수는 유니온이 되지만, 매개변수는 반공변성을 가지고 있기 때문에 인터섹션이 됩니다.

## 2.23 타입을 좁혀 정확한 타입을 얻어내자
- instanceof 연산자를 이용해서 클래스를 구분할 수 있습니다.
  - ```ts
    class A {}
    class B {}
    function classAorB(param: A | B) {
        if (param instanceof A) {
            param; // (parameter) param: A
        } else {
            param; // (parameter) param: B
        }
    }
    ```
- 타입 좁히기는 자바스크립트에서 실행하기 때문에 자바스크립트 문법을 사용해서 진행해야 합니다.
- 브랜드 속성을 사용하면 객체의 구분이 쉬워집니다.
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
    
    function moneyOrLiter(param: Money | Liter) {
        if (param.__type === 'money') {
            param; // (parameter) param: Money
        } else {
            param; // (parameter) param: Liter
        }
    }
    ```
    - 타입 좁히기 함수를 만들 수도 있습니다.
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
        
        function isMoney(param: Money | Liter): param is Money {
            if (param.__type === 'money') {
                return true;
            } else {
                return false;
            }
        }
        
        function moneyOrLiter(param: Money | Liter) {
            if (isMoney(param)) {
                param; // (parameter) param: Money
            } else {
                param; // (parameter) param: Liter
            }
        }
        ```

## 2.24 자기 자신을 타입으로 사용하는 재귀 타입이 있다
- 자기 자신을 타입으로 다시 사용하는 타입을 재귀 타입이라고 부릅니다.
- 컨디셔널 타입에도 사용할 수 있습니다.
  - ```ts
    type ElementType<T> = T extends any[] ? ElementType<T[number]> : T;
    ```
- 타입 인수로 사용하는 것은 불가능합니다.
  - ```ts
    type T = number | string | Record<string, T>;
    // Type alias 'T' circularly references itself.
    ```
    - 타입 인수를 사용하지 않는 방식으로 변경해야 합니다.
      - ```ts
        type T = number | string | { [key: string]: T };
        ```
- 재귀 타입을 통해 배열을 거꾸로 뒤집는 것도 가능합니다.
  - ```ts
    type Reverse<T> = T extends [...infer L, infer R] ? [R, ...Reverse<L>] : [];
    ```

## 2.25 정교한 문자열 조작을 위해 템플릿 리터럴 타입을 사용하자
- 템플릿 리터럴 타입은 백틱과 보간을 사용하는 특수한 문자열 타입입니다.
  - ```ts
    type Literal = "literal";
    type Tempalte = `template ${Literal}`;
    // type Tempalte = "template literal"
    
    const str: Tempalte = 'template literal';
    ```
- 문자열 타입 안에 다른 타입을 변수로 넣을 수 있습니다.
  - ```ts
    type Tempalte = `template ${number}`;
    let t1: Tempalte = 'template 1';
    t1 = 'template 123';
    t1 = 'template a';
    // Type '"template a"' is not assignable to type '`template ${number}`'.
    ```
- 템플릿 리터럴 타입을 사용하면 `문자열 변수`를 엄격하게 관리할 수 있습니다.
- 문자열의 조합을 표현할 때 편리합니다.
  - ```ts
    type City = 'seoul' | 'suwon' | 'busan';
    type Vehicle = 'car' | 'bike' | 'walk';
    type ID = `${City}:${Vehicle}`;
    const id: ID = 'seoul:walk';
    ```
- 템플릿 리터럴 타입은 재귀 호출이 가능합니다.
  - ```ts
    type RemoveX<Str> = Str extends `x${infer Rest}` ? RemoveX<Rest> : Str extends `${infer Rest}x` ? RemoveX<Rest> : Str;
    type Removed = RemoveX<'xxtestxx'>;
    // type Removed = "test"
    ```

## 2.26 추가적인 타입 검사에는 satisfies 연산자를 사용하자
- satisfies 연산자는 타입 추론을 그대로 활용하면서 추가로 타입 검사를 하고 싶을 때 사용합니다.
  ```ts
  const universe: {
      [key in 'sun' | 'sirius' | 'earth']: { type: string, parent: string} | string
  } = {
      sun: 'star',
      sriius: 'star', // sirius 오타
      earth: { type: 'planet', parent: 'sun'},
  };
  // Object literal may only specify known properties...

  universe.earth.type;
  // Property 'type' does not exist on type 'string | { type: string; parent: string; }'...

  /*
  const universe: {
      sun: string | {
          type: string;
          parent: string;
      };
      sirius: string | {
          type: string;
          parent: string;
      };
      earth: string | {
          type: string;
          parent: string;
      };
  }
  */
  ```
  - 인덱스 시그니처를 사용해 siriius의 오타를 찾을 수 있습니다.
  - 속성을 사용할 때 earth의 타입이 객체라는 사실을 제대로 나타내지 못합니다.
    - ```ts
      const universe = {
          sun: 'star',
          sriius: 'star', // sirius 오타
          earth: { type: 'planet', parent: 'sun'},
      };
      
      /* 
      const universe: {
          sun: string;
          sriius: string;
          earth: {
              type: string;
              parent: string;
          };
      }
       */
      ```
      - 변수를 선언만 했을 땐 타입을 정확히 추론합니다.
      - 이 이점을 누리면서 sriius의 오타를 찾고 싶을 때 satisfies 연산자를 사용합니다.
        - ```ts
          const universe = {
              sun: 'star',
              sriius: 'star', // sirius 오타
              earth: { type: 'planet', parent: 'sun'},
          } satisfies {
              [key in 'sun' | 'sirius' | 'earth']: { type: string, parent: string } | string
          };
          // Object literal may only specify known properties...
          
          /* 
          const universe: {
              sun: string;
              sriius: string;
              earth: {
                  type: string;
                  parent: string;
              };
          }
           */
          
          universe.earth.type
          ```
          - universe의 타입을 추론하면서 sriius 오타를 검사해주고 있습니다.
          - earch의 속성을 에러없이 사용할 수 있습니다.

## 2.27 타입스크립트는 건망증이 심하다
- 타입을 강제할 경우에 흔히하는 실수가 있습니다.
  - ```ts
     try {} catch (error) {
        if (error) {
            error.message;
        }
    }
    // Property 'message' does not exist on type '{}'.
    ```
    - error는 unknown 타입입니다.
    - unknown은 if문을 통과하면 {}타입이 됩니다.
    - {}타입은 속성을 사용할 수 없으므로 구체적으로 타입을 주장하겠습니다.
      - ```ts
        try {} catch (error) {
            if (error as Error) {
                error.message;
            }
        }
        // 'error' is of type 'unknown'.
        ```
        - error를 Error 타입으로 강제했지만, error가 unknown으로 나옵니다.
        - as로 강제 주장은 if문이 참인지 거짓인지를 판단할 때만 주장한 타입이 사용되고, 판단한 후에는 원래 타입으로 돌아갑니다.
        - 주장한 타입을 계속 기억할 수 있게 만들어야 합니다.
          - ```ts
            try {} catch (error) {
                const err = error as Error;
                if (err) {
                    err.message;
                }
            }
            // const err: Error
            ```
            - `타입을 주장할 때는 그 타입이 일시적이므로, 변수에 담에 오래 기억해야 합니다.`

## 2.28 원시 자료형에도 브랜딩 기법을 사용할 수 있다
- ```ts
  type Brand<T, B> = T & { __brand: B };
  type KM = Brand<number, 'km'>;
  type Mile = Brand<number, 'mile'>;
  
  function kmToMile(km: KM) {
      return km * 0.62 as Mile;
  }
  
  const km = 3 as KM;
  const mile = kmToMile(km);
  // const mile: Mile
  
  const mile2 = 5 as Mile;
  kmToMile(mile2);
  // Argument of type 'Mile' is not assignable to parameter of type 'KM'...
  ```
  - KM, Mile은 원래 존재하는 타입이 아니기 때문에 as로 강제 변환해야 합니다.
