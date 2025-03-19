## 배운 것을 바탕으로 타입을 만들어보자
 ### 판단하는 타입 만들기
  - IsNever
    ```ts
    type IsNever<T> = [T] extends [never] ? true : false;
    ```
  - IsAny
    ```ts
    type IsAny<T> = string extends (number & T) ? true : false;
    ```
  - IsArray
    ```ts
    type IsArray<T> = IsNever<T> extends true 
      ? false
      : T extends readonly unknown[]
        ? IsAny<T> extends true
        : true
      : false;
    ```
    - IsArray<never>가 never가 되는 것을 막기위해 IsNever<T> extends true가 필요
    - IsArray<any>가 boolean이 되는 것을 막기 위해 IsAny<T> extends true가 필요
    - IsArray<readonly []>가 false가 되는 것을 막기 위해 T extends readonly unknown[]이 필요
  - IsTuple
    ```ts
    type IsTuple<T> = IsNever<T> extends true
      ? false
      : T extends readonly unknown[]
        ? number extends T["length"]
          ? false
          : true
        : fasle;
    ```
    - 튜플은 길이가 고정되어 있다. 즉 number extends T["length"]가 false이다.
  - IsUnion
    ```ts
    type IsUnion<T, U = T> = IsNever<T> extends true
      ? false
      : T extends T
        ? [U] extends [T]
          ? false
          : true
        : false;
    ```
    - 유니언의 경우 컨디셔널 타입 제네릭과 만나면 분배법칙이 발생하므로 타입 매개변수를 하나 더 만듦
    - 분배법칙을 일으킨 후 U = T로 원본타입을 담아두어 [U] extends [T]는 false가 된다.

### 집합 관련 타입 만들기
![차집합](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRAlAFhVX2Obj7H8WLuAFxkjZIAJIKrmD2Bm_kqoFKAR-9sZrP5Hsi8MSSfYQgjTrR3pYU&usqp=CAU)   A - B 차집합
```ts
type Diff<A, B> = Omit<A & B, keyof B>;
type R1 = Diff<{ name: string, age: number }, { name: sring, married: boolean }>;
// type R1 = { age: number }
```
**Omit**
- 특정 객체에서 지정한 속성을 제거하는 타입.
- A & B는 { name: string, age: number } 인데 keyof B는 name | married이므로 age속성만 남음

**Diff**

![title](https://upload.wikimedia.org/wikipedia/commons/thumb/f/f2/SetSymmetricDifference.svg/300px-SetSymmetricDifference.svg.png)   대칭 차집합
```ts
type SymDiff<A, B> = Omit<A & B, keyof (A | B)>;
type R2 = SymDiff<{ name: string, age: number }, { name: string, married: boolean }>;
// type R2 = { age: number, married: boolean }
```
- Diff타입을 응용하여 대칭 차집합을 만들 수 있는데, 현재 코드에서 차집합과 대칭차집합은 객체에만 적용 가능

**Exclude**
  - 어떤 타입( A | B )에서 다른 타입 ( A & B )을 제거하는 타입
  - A가 B타입에 대입 가능하면 A는 B의 부분집합이다.
  ```ts
  type IsSubset<A, B> = A extends B ? true : false;
  type R1 = IsSubset<string, string | number>; // true
  type R2 = IsSubset<{ name: string, age:number }, { name: string}>; // true
  type R3 = IsSubset<symbol, unknow>; // true
  ```

**Equal**
 - A가 B의 부분집합이고 B도 A의 부분집합이면 A와 B는 동일하다.
```ts
type Equal<A, B> = A extends B ? B extends A ? true : false : false;

// boolean이나 never는 유니언이므로 분배법칙 발생함
type R1 = Equal<boolean, true | false>; // boolean
type R2 = Equal<never, never>; // never

// 분배법칙이 일어나지 않게 바꾸기
type Equal<A, B> = [A] extends [B] ? [B] extends [A] ? true : false;
```
- Eqaul1 타입은 any와 다른 타입을 구별하지 못한다.
```ts
type Equal2<X, Y>
  = (<T>() => T extends X ? 1 : 2) extends (<T>() => T extends Y ? 1 : 2)
    ? true
    : false

// X와 Y가 같은 타입이면 Equal2<X, Y> 타입이 true가 된다.
// 두 타입이 다를 때 Equal2<X, Y>타입이 false가 된다는 것만 확인하면 된다.
```



| X | Y | T | (<T.>() => T extends X ? 1 : 2) | (<T.>() => T extends Y ? 1 : 2) | extends |
| --- | --- | --- | --- | --- | --- |
| string | any | number | 2 |1 |false|
| any | string | number | 1 |2|false|
| 1 | number | 2 | 2 | 1 | false |
- Equal2 타입을 사용하면 any는 다른 타입과 잘 구별하나 인터섹션을 인식하지 못한다.
- Equal2<any, unknown>의 경우는 extends를 false로 만드는 T가 없음에도 false가 된다.

**NotEqual**
```ts
type NotEqual<X,Y> = Equal<X,Y> extends true ? false : true
// Equal 타입의 결과를 반대로 적용하면 됨
```

## 함수에 기능을 추가하는 데코레이터함수가 있다
- 여러 함수에서 공통으로 수행되는 부분을 데코레이터로 만들어두면 좋다
   ```ts  
  function stratAndEnd(This, Args extends any[], Return) (
    originalMethod: (this: any, ...args: any[]) => Return ,
    context: ClassMethodDecorator<This, (this: This, ...args: Args) => Return>
  ){
    function replacementMethod(this: This, ...args:Args): Return {
        console.log('start');
        const result = originalMethod.call(this, ...args);
        console.log('end');
        return result;
    }  
  return replacementMethod;
  }

  Class A {
    @startAndEnd
    eat() {
      console.log('Eat');  // eat메서드 호출 시 start, Eat, end
    }

  }
  @startAndEnd
  work() {
    console.log('Work');  
  }
  ```
  - ClassDecoratorContext: 클래스 자체를 장식할 때
  - ClassMethodDecoratorContext: 클래스 메서드를 장식할 때
  - ClassGetterDecoratorContext: 클래스의 getter를 장식할 때
  - ClassSetterDecoratorContext: 클래스의 setter를 장식할 때
  - ClassMemberDecoratorContext: 클래스의 멤버를 장식할 때
  - ClassAccessorDecoratorContext: 클래스 accessor를 장식할 떄
  - ClassFieldDecoratorContext: 클래스 필드를 장식할 때

 Context의 객체
 ```ts
 type Context = {
  kind: string; // 데코레이터의 유형 ClassDecoratorContext라면 class, ClassMethodDecoratorContext라면 method
  name: string | symbol; // 장식 대상의 이름
  access: { // get, set, has 등의 접근자를 모아둔 객체
   get?(): unknown;
   set?(value: unknown): void;
   has?(value: unknown): boolean;
  };
 private?: boolean; // private여부
 static?: boolean; // static여부
 addInitializer?(initializer: () => void): void; // 초기화 시 실행되는 함수
 }
 ```
 - 데코레이터 자체도 함수이므로 매개변수를 가질 수 있으나, 고차함수를 이용해야 한다.

## 앰비언트 선언도 선언 병합이 된다
 - 타입스크립트에서 남의 라이브러리를 사용해야 할 때 declare예약어를 사용해 앰비언트 선언을 한다.
   ```ts
   declare namespace NS {
    const v: string;
   };
   declare enum Enum {
    ADMIN = 1
   }
   declare function func(param: number): string;
   declare const variable: number;
   declare class C {
    constructor(p1: string, p2: string);
   };

   new C(func(variable), NS.v);
   ```
   - 코드에 구현부가 없고, 변수에 타입만 있고 값을 대입하지 않고있음
   - 그래도 new C나 func(variable), NS.v처럼 값으로 사용할 수 있는 이유는 외부 파일에 실제 값이 존재하다고 믿기 때문
   - 만약 외부에 값이 없다면 런타임에러가 발생하므로 declare로 앰비언트 선언 시 반드시 해당 값이 실제로 존재하는지 확인해야 함
   - 인터페이스와 타입 별칭도 declare로 선언할 수 있으나, 선언하지 않아도 동일하게 작동하므로 굳이 붙일 필요 없음
   
**선언이 생성하는 개체**
 | 유형 | 네임스페이스 | 타입 | 값 |
 | --- | --- | --- |--- |
 | 네임스페이스 | O |  | O |
 | 클래스 |  | O | O |
 | enum |  | O | O |
 | 인터페이스 |  | O |  |
 | 타입 별칭 |  | O |  |
 | 함수 |  |  | O |
 | 변수 |  |  | O |
- 인터페이스나 네임스페이스는 같은 이름으로 여러 개 존재할 때 병합되고, 여러 번 선언할 수 있는 대표적인 예이다.
- 함수는 오버로딩 되므로 여러 번 선언할 수 있다.
- 모두를 외우기 쉽지 않으므로 인터페이스, 네임스페이스 병합이나 함수 오버로딩 같이 널리 알려진 경우를 제외하고는 웬만하면 같은 이름으로 여러번 선언하지 않는 것이 좋다.

