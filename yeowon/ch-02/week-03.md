## 같은 이름의 함수를 여러 번 선언할 수 있다
- 타입스크립트에서는 매개변수에 어떤 타입과 값이 들어올지 미리 타입을 선언해야 한다.
- 오버로딩: 호출할 수 있는 함수의 타입을 미리 여러 개 타이핑 해두는 기법
  ```ts
  function add(x: number, y: number): number
  function add(x: string, y: string): string
  function add(x: any, y: any) {
    return x + y;
  }

  add(1, 2); // 3
  add('1', '2') // 12
  add(1, '2') // error
  ```
- 오버로팅을 선언하는 순서가 타입 추론에 영향을 끼친다.
  ```ts
  function example(param: string): string;
  function example(param: string | null): number;
  function example(param: string | null): string | number {
    if (param) {
        return 'string';
    } else {
        return 123;
    }
  }

  const result = example('what'); // ressult: string
  ```
  - 여러 오버로딩에 동시에 해당될 수 있는 경우는 제일 먼저 선언된 오버로딩에 해당된다.
  - 만약 첫번째 example 함수와 두번째 example함수의 순서를 바꾼다면 result가 number로 바뀐다. (실제로는 'what'이 string이므로 에러발생)
- 오버로딩의 순서는 좁은타입부터 넓은타입으로 오게 해야한다.
- 오버로딩할 때 주의할 점 -> 지나치게 활용하면 안됨
  ```ts
  function a(param: string): void 
  function a(param: number): void 
  function a(param: string | number) {} 
  
  function errorA(param: string | number) {
      a(param); // error
  }
  ```
  - erraA의 param이 string | number 인데 a의 param은 string이나 number라서 발생하는 에러
  - 오버로딩을 제거하면 에러메시지가 사라짐
  - 유니언이나 옵셔널 매개변수를 활용할 수 있는 경우는 오버로딩을 쓰지 않는게 좋다.
 
## 콜백 함수의 매개변수는 생략 가능하다
- 기본적으로 함수의 매개변수에는 타입을 표기해야 하나, 인수로 제공하는 콜백 함수의 매개변수에는 타입을 표시하지 않아도 된다.
  ```ts
  function example(callback: (error: Error, result: string) => void) {} // 함수의 매개변수엔 타입 표시
  example((e, r) => {}); // 타입표시 안함
  example(() => {});
  ```
  - 함수를 선언할 때 콜백 함수에 대한 타입을 표시했기 때문에 표시하지 않아도 추론이 가능하다 -> 문맥적 추론
  - 매개변수를 콜백함수에서 사용하지 않아도 된다. 이때 괜히 옵셔널로 만들면 값이 undefined가 되어 의도와 달라질 수 있다.

## 공변성과 반공변성을 알아야 함수끼리 대입할 수 있다
- 공변성: A -> B 일 때 T<A> -> T<B>인 경우
- 반공변성: A -> B 일 때 T<B> -> T<A>인 경우
- 이변성: A -> B 일 때 T<A> -> T<B>도 되고 T<B> -> T<A>도 되는 경우
- 무공변성: A -> B 일 때 T<A> -> T<B>도 안 되고 T<B> -> T<A>도 안 되는 경우
- 기본적으로 타입스크립트는 공변성을 갖지만, 함수의 매개변수는 반공변성을 갖는다.
  ```ts
  // a와 b타입의 차이는 반환값 뿐 b가 a보다 넓은 타입
  function a (x:string):number {
    return 0;
  }

  type B = (x:string) => number | string;
  let b: B = a;
  ```
  - a함수를 b타입에 대입할 수 있다. 이 관계를 a -> b라고 표현, T타입을 함수<반환값>이라고 표현
  - a -> b 일때 T<a> -> T<b>가 되므로 함수의 반환값은 공변성을 갖고 있다.
  ```ts
  // a가 넓은타입
  function a (x:string | number): number {
    return 0;
  }

  type B = (x:string) => number;
  let b: B = a;
  ```
  - b -> a인 상황이고 a를 b에 대입할 수 있으므로 매개변수가 반공변성을 갖는다.
 
## 클래스는 값이면서 타입이다.
- 타입스크립트는 name, age 같은 멤버를 클래스 내부에 한 번 적어야 한다는 것이다.
- 이 멤버들은 항상 constructor 내부와 짝이 맞아야 한다.
- 엄격하게 클래스의 멤버가 들어있는지 검사하기 위해서는 implements예약어를 사용하면 된다.
  ```ts
  interface Human {
    name: string;
    age: number;
    married: boolean;
    sayName(): void;
  }

  class Person implements Human { // error: Human인터페이스의 sayName메서드를 구현하지 않았음
    name;
    age;
    married;
    constructor(name: string, age:number, married: boolean) {
      this.name = name;
      this.age = age;
      this.married = married
    }
  }
  ```
- 타입스크립트는 생성자 함수 방식으로 객체를 만드는 것을 지원하지 않는다.
- 클래스의 이름은 클래스 자체 타입이 아니라 인스턴스 타입이 됨. 자체 타입으로 사용하려면 'typeof 클래스이름' 으로 타이핑
- 클래스 멤버로는 옵셔널, readonly외의 public, protected, private 수식어가 추가됨
| 접근 제한자  | 자기 클래스 | 자식 클래스 | 인스턴스 |
|-------------|------------|------------|----------|
| `public`    | ✅ 가능   | ✅ 가능   | ✅ 가능 |
| `protected` | ✅ 가능   | ✅ 가능   | ❌ 불가능 |
| `private`   | ✅ 가능   | ❌ 불가능 | ❌ 불가능 |

- private 속성을 선언할 때 타입스크립트의 private을 사용할지 #을 사용할지 고민을 위한 차이점
  ```ts
  class PrivateMember {
    private priv: string = 'priv';
  }
  class ChildPrivateMemnber extends PrivateMember { // error
    private priv: string = 'priv'
  }
  
  class PrivateField {
    #priv: string = 'priv';
    sayPriv() {
      console.log(this.#priv);
    }
  }
  class ChildPrivateField extends PrivateField { 
    private priv: string = 'priv'
  }
  ```
  - private 수식어로 선언한 속성은 자손 클래스에서 같은 이름으로 선언할 수 없다.
  - private 수식어는 private filed로 대체
- implements하는 인터페이스의 속성은 전부 public이어야 함
- 명시적으로 오버라이드할 때는 override 수식어를 붙여야 함
  ```ts
  class Human {
    eat() {
      console.log('yamyam');
    }
    sleap() {
      console.log('zzz')
    }
  }
  class Employee extends Human {
    work() {
      console.log('work');
    }
    override sleap() {
      console.log('자고싶다')
    }
  }
  ```
  - override 수식어를 붙이면 부모 클래스의 메서드가 바뀔 때 확인이 가능
- 클래스의 생성자 함수에도 오버로딩을 적용할 수 있음
  ```ts
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
  - 함수의 구현부는 한 번만 나와야 함
  - 여러 번 타입 선언한 것들에 대해 모두 대응할 수 있어야 함
 ### 추상 클래스
 - implements보다 더 구체적으로 클래스의 모양을 정의하는 방법
   ```ts
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
    
    class RealPerson extends AbstractPerson { // error value속성과 sayMarried메서드를 구현하지 않았음
        sayAge() {
            console.log(this.age);
        }
    }
   ```
   - 속성과 메서드가 abstract인 경우 실제 값은 없고 타입 선언만 되어있음
   - RealPerson클래스는 AbstractPerson클래스를 상속하므로 abstract속성이나 메서드를 구현해야 함

## enum은 자바스크립트에서도 사용할 수 있다
- 자바스크립트에는 없는 타입이지만 자바스크립트의 값으로 사용할 수 있는 특이한 타입
- 여러 상수를 나역하는 목적으로 사용되며 enum예약어로 선언
  ```ts
  enum Level {
  NOVICE,
  INTERMEDIATE,
  ADVANCED,
  MASTER
  }  
  ```
  - Level enum 내부에 존재하는 이름을 멤버라고 부른다.
  - 자바스크립트로 변환할 때 사라지지 않고 자바스크립트 코드로 남는다.
- 기본적으로 enum은 멤버의 순서대로 0부터 숫자를 할당하지만 = 연산자를 사용하여 변경이 가능하다.
- 한 멤버를 문자열로 할당하면 그 다음부터는 전부 직접 값을 할당해야 한다.
- enum 타입의 속성은 값으로도 활용 가능하나, 타입으로 사용하는 경우가 더 많다.
  ```ts
  function whatsYourLevel(level: Level) {
    console.log(Level[level]);
  }

  const myLevel = Level.ADVANCED;
  whatsYourLevel(myLevel);
  ```
  - 매개변수의 타입으로 enum사용하면 멤버의 유니언과 비슷한 역할을 함
- 타입스크립트의 enum은 아직 완벽하지 않다.
- const enum을 사용하면 자바스크립트 코드가 생성되지 않게 할 수 있다.
  ```ts
  const enum Money {
    WON,
    DOLLAR,
  }
  Money.WON; //Money.WON = 0
  Money[Money.WON]; // error: Money라는 자바스크립트 객체가 없으므로 불가능
  ```

## inferfh 타입스크립트의 추론을 직접 활용하자
- infer 예약어는 타입스크립트의 타입 추론 기능을 극한까지 활용하는 기능
- 배열이 있을 때 배열의 요소 타입을 얻어내고 싶은 상황에 사용
  ```ts
  type El<T> = T extends (infer E)[] ? E : nenver; // E가 타입변수
  type Str = El<string[]>; // type Str = string
  type NumOfBool = El<(number | boolean)[]>; // type NumOrBool = number | boolean
  ```
  - 타입스크립트에 추론을 맡기고 싶은 부분을 `infer 타입_변수`로 표시
  - 컨디셔널 타입에서 타입변수는 참 부분에서만 쓸 수 있다.
- 타입스크립트는 많은 부분을 스스로 추론할 수 있다.
  ```ts
    type MyParameters<T> = T extends (...args: infer P) => any ? P : never; // 매개변수
    type MyConstructorParameters<T> = T extends abstract new (...args: infer P) => any ? P : never; // 생성자
    type MyReturnType<T> = T extends (...arg: any) => infer R > R : any; // 반환값
    type MyInstanceType<T> = T extends abstract new (...args: any) => infer R ? R : any; // 인스턴스

    type P = MyParameters<(a: stirng, b: number) => string> // type P = [a: string, b: number]
    type R = MyReturnType<(a: string, b: number) => string> // type R = string
    type CP = MyConstructorParameters<new (a: string, b: number) => {}> // type CP = [a: string, b: number]
    type I = MyInstanceType<new (a: string, b: number) => {}> // type I = {}
  ```
  ```ts
  type Union<T> = T extends {a:infer U, b: infer U} ? U : never;
  type Result1 = Union<{a: 1 | 2, b: 2 | 3 }> // type Result1 = 1 | 2 | 3
  ```
  - 반환값 타입을 같은 U타입 변수를 선언
  - 같은 이름의 타입 변수는 서로 유니언이 된다.
  ```ts
  type Intersection<T> = T extends {
    a: (pa: infer U) => void,
    b: (pb: infer U) => void
  } ? U : never;
  type Result2 = Intersection<{a(pa:1|2): void, b(pb: 2|3): void}> // type Result2 = 2
  ```
  - 메서드의 매개변수에 같은 U타입 변수를 선언
  - 매개변수인 경우, 반공변성을 갖고 있으므로 인터섹션이 된다.

## 타입을 좁혀 정확한 타입을 얻어내자
- 타입스크립트가 코드를 파악해서 타입을 추론하는 것을 제어 흐름 분석이라고 부른다.
- 다만, 이 제어 흐름 분석이 완벽하지는 않다는 것을 염두해 두어야 한다.
  ```ts
    function strOrNullOrUndefined(param: string | null | undefined) {
    if(typeof param === undefined) {
      param; // undefined
    } else if (param) {
      param; // string
    } else {
      param; // string | null
    }
  }
  ```
  - 처음 if문에서 undefined 타입이 걸러지지만 두번째에서 string이 걸러지지 않는다. 빈 문자열이 있으므로 else문에서 param이 string일 수 있다.
  - 자바스크립트 유명한 버그 중 typeof null이 'object'이기 때문에 객체와 typeof의 결과가 똑같아서 typeof로는 null을 구분 할 수 없다.
  ```ts
  function strOrNullOrUndefined(param: string | null | undefined) {
    if(param === undefined) {
      param; // undefined
    } else if (param === null) {
      param; // null
    } else {
      param; // string
    }
  }
  ```
  - 타입 좁히기에 꼭 typeof를 써야 할 필요는 없다. 간단하게 가능
  ```ts
  // 두 객체를 구분하는 방법
  interface X {
    width: number;
    height: number;
  }
  interface Y {
    length: number;
    center: number;
  }
  function objXorY(param: X | Y) {
    if(param instanceof X) { // error: X는 타입스크립트의 인터페이스
      param;
    } else {
      param;
    }
  }
  ```
  - 타입 좁히기는 자바스크립트 문법을 사용해야 한다.
  ```ts
  function objXorY(param:X | Y) {
    if('width' in param) {
      param; // param: X
    } else {
      parma; // param: Y
    }
  }
  ```
  - 따라서 이러한 방법으로 두 객체를 구분할 수 있다. (in 연산자는 자바스크립트 문법)
  - 혹은 브랜드 속성을 사용하여 객체를 구분 할 수도 있다.
 
## 자기 자신을 타입으로 사용하는 재귀 타입이 있다
- 재귀타입: 자기 자신을 타입으로 다시 사용하는 타입
  ```ts
  type Recursive = {
    name: string;
    children: Recursive[];
  };
  const recur1: Recursive = {
    name: 'test',
    children: []
  }
    const recur2: Recursive = {
    name: 'test',
    children: [
      {name: 'test2', children: []},
      {name: 'test3', children: []}
    ]
  }
  ```
- 컨디셔널 타입에도 사용할 수 있다.
- 타입 인수로 사용하는 것은 불가능하다.
- 재귀 타입을 사용할 때 발생하는 에러
  ```ts
  type InfiniteRecur<T> = {item: InfinitedRecur<T>};
  type Unwrap<T> = T extends {item: infer U} ? Unwrap<U> : T;
  type Result = Unwrap<InfiniteRecur<any>>; // error
  ```
  - InfiniteRecur 타입은 무한하므로 Unwrap 타입은 유한한 시간 안에 InfiniteRecur 타입을 처리할 수 없다.
 
## 정교한 문자열 조작을 위해 템플릿 리터럴 타입을 사용하자
- 자바스크립트의 템플릿 리터럴과 사용법이 비슷하지만, 값 대신 타입을 만들기 위해 사용한다.
  ```ts
   type Literal = 'literal';
    type Template = `template ${string}`;
    let str: Template = 'template ';
    str = 'template hello';
    str = 'template 123';
    str = 'template'; // error
  ```
  - template문자열 뒤에 띄어쓰기가 없는 마지막 str에만 에러가 발생
  - 템플릿 리터럴 타입을 사용하면 문자열 변수를 엄격하게 관리할 수 있다.
- 템플릿 리터럴 타입은 제네릭 및 infer와 함께 사용하면 더 강력하다.
  ```ts
  // 좌우의 x를 지우는 타입
  type RemoveX<Str> = Str extends `x${infer Rest}`
  ? RemoveX<Rest>
  : Str extends `${infer Rest}x` ? RemoveX<Rest> : Str;
  ```
  1. RemoveX<'xxtestxx'> => infer Rest는 x로 시작하는 문자열이므로 true가 되고 `infer Rest`는 xtestxx가 됨
  2. RemoveX<'xtestxx'> => 위와 동일한 이유로 testxx가 됨
  3. RemoveX<'testxx'> => 좌측 x가 전부 지워졌으므로 `infer Rests`는 false, `${infer Rest}x`를 평가함
  4. RemoveX<'testx'> => `${infer Rest}x`를 평가하여 testx
  5. RemoveX<'test'> => `${infer Rest}` 와 `${infer Rest}x` 모두 false 이므로 test가 됨

## 추가적인 타입 검사에는 satisfies연산자를 사용하자
  - 타입 추론을 그대로 활용하면서 추가로 타입 검사를 하고 싶을 때 사용
    ```ts
    const universe:{
      [key in 'sun' | 'sirius' | 'earth']: {type: string, parent: string} | string
    } = {
      sun: 'star',
      sriius: 'start', // 오타
      earth: {type: 'planet', parent: 'sun'},
    };
    ```
    - 인덱스 시그니처를 사용하면 오타를 잡을 수 있으나, earth의 타입이 객체라는 것을 제대로 잡아내지 못한다.
    - 속성 값의 타입을 객체와 문자열의 유니언으로 표기해놨기 때문에 earth가 문자열일 수도 있다고 인식한다.
  - 타입 추론된 것을 그대로 사용하면서, 각각의 속성들을 검사하는 방법을 위해 `satisfies연산자`를 사용한다.
    ```ts
    const universe = {
      sun: 'star',
      sriius: 'start', // 오타
      earth: {type: 'planet', parent: 'sun'},
    } satisfies {
      [key in 'sun' | 'sirius' | 'earth']: {type: string, parent: string} | string
    }
    ```
    - universe에 마우스오버 해보면 `satisfies연산자`를 사용하기 전과 동일하게 타입을 추론하고 있는것 확인 가능
    - universe.earth.type; earth속성도 에러 없이 쓸 수 있다.

## 타입스크립트는 건망증이 심하다
- 타입을 강제로 주장하는 경우에 흔히 나타나는 실수
  ```ts
  try {} catch (error) {
    if(error as Error) {
      error.message //error: unknown
    }
  }
  ```
  - as로 강제 주장한 것이 일시적이기 때문에 error를 Error타입이라 강제 주장했는데 unknown타입이 나오고 있음
- 주장한 타입을 계속 기억할 수 있게 만드는 방법
  ```ts
  try {} catch (error) {
    const err = error as Error;
    if(err) {
      error.message 
    }
  }
  ```
  - 타입을 주장할 때는 그 타입이 일시적이므로, 변수에 담아야 오래 기억한다.
 
## 원시 자료형에도 브랜딩 기법을 사용할 수 있다
- string, number와 같은 원시 자료형 타입도 더 세밀하게 구분할 수 있다.
  ```ts
  type Brand<T, B> = T & {_brand: B};
  type KM = Brand<number, 'km'>;
  type Mile = Brand<number, 'mile'>;

  function kmtoMile(km: KM) {
    return km * 0.62 as Mile;
  }

  const km = 3 as KM;
  const mile = kmToMile(km);
  const mile2 = 5 as Mile;
  kmToMile(mile2);
  ```
  - 자바스크립트에선 3이라는 숫자가 있을 때 이 숫자가 km 단위인지 mile단위인지 모른다.
  - 브랜딩 기법을 사용하면 더 구체적으로 타입을 정할 수 있다.
  - 브랜딩 기법을 통해 number타입에 각자 다른 브랜드 속성을 추가하면 둘다 number이지만 서로 구분되게 된다.
  - number타입을 KM, Mile타입으로 세분화 하여 안정성 있게 활용할 수 있다.
