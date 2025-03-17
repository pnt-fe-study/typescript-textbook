# 2.29 배운 것을 바탕으로 타입을 만들어보자

## 1) 판단하는 타입 만들기

### IsNever

먼저 never인지 판단하는 IsNever 타입이다.

```ts
type IsNever<T> = [T] extends [never] ? true : false;
```

- 배열로 감싸는 이유는 T에 never를 넣을 때 분배법칙이 일어나는 것을 막기 위해서다.

### IsAny

any 타입인지 판단하는 IsAny 타입을 만들어보자.

```ts
type IsAny<T> = [T] extends number & T ? true : false;
```

- 기본적으로 string과 numbersms 겹치지 않지만, T가 any라면 any를 extends 할 수 있게 된다.
- T가 any일 때만 true이므로 any인지 아닌지를 판단하는 역할을 할 수 있다.

### IsArray

배열인지 판단하는 IsArray 타입을 만들어보자.

```ts
type IsArray<T> = IsNever<T> extends true
  ? false
  : T extends readonly unknown[]
  ? IsAny<T> extends true
    ? false
    : true
  : false;
```

- T가 never, any, readonly [] 타입일 때는 false가 되지 않는다.
- IsArray<T> 타입이 복잡한 건 IsArray<never>가 never가 되는 것을 막기 위해 IsNever<T> extends true가 필요하다.
- IsArray<readonly []>가 false가 되는 걸 막기 위해 T extends readonly unknown[]이 필요하다.

### IsTuple

배열 중 튜플만 판단하는 IsTuple을 만들어보자. (튜플이 아닌 배열 타입은 false)

```ts
type IsTuple<T> = IsNever<T> extends true
  ? false
  : T extends readonly unknown[]
  ? number extends T["length"]
    ? false
    : true
  : false;
```

- 튜플과 배열의 차이점은 길이가 고정되어 있다는 것이다.
- 튜플이 아닌 배열은 length가 number이다.
- any는 number extends T["length"]에서 걸러지므로 따로 any를 검사하지 않아도 된다.

### IsUnion

```ts
type IsUnion<T, U = T> = IsNever<T> extends true
  ? false
  : T extends T
  ? [U] extends [T]
    ? false
    : true
  : false;
```

- T extends T를 만든 이유는 분배 법칙을 만들기 위해서다.
- 유니언의 경우 컨디셔널 타입 제네릭과 만나면 분배법칙이 발생한다.
- T가 string이었다면 [U] extends [T]에서 [string] extends [string]이 되므로 true가 되어버려 IsUnion<string>은 false가 된다.

## 2) 집합 관련 타입 만들기

### 차집합을 만들어보자!

예를 들어 A가 `{ name: string, age: number }`, B가 `{ name: string, married: boolean }`인 경우 둘을 차집합(A-B)하면 `{ age: number }`가 나와야 한다.

```ts
type Diff<A, B> = Omit<A & B, keyof B>;
type R1 = Diff<
  { name: string; age: number },
  { name: string; married: boolean }
>;
```

### Omit

특정 객체에서 지정한 속성을 제거하는 타입이다. A & B는 { name: string, age: number, married: boolean }인데 keyof B는 `name | married`이므로 name과 married 속성을 제거하면 age 속성만 남게 된다.

### Diff

Diff 타입을 조금 응용하면 대칭차집합도 찾아낼 수 있다. 서로 겹치지 않는 부분을 합쳐놓은 것으로 합집합에서 교집합을 뺀 것으로 볼 수 있다.

```ts
type SymDiff<A, B> = Omit<A & B, keyof (A | B)>;
type R2 = SymDiff<
  { name: string; age: number },
  { name: string; married: boolean }
>;
```

> 해당 코드는 객체만 적용 가능하다. 유니언에서는 적용되지 않는다.

```ts
type SymDiffUnion<A, B> = Exclude<A | B, A & B>;
type R3 = SymDiffUnion<1 | 2 | 3, 2 | 3 | 4>;
```

### Exclude

어떤 타입(A | B)에서 다른 타입 (A & B)을 제거하는 타입이다.

```ts
type IsSubset<A, B> = A extends B ? true : false;
type R1 = IsSubset<string, string>; // true
type R2 = IsSubset<{ name: string; age: number }, { name: string }>; // true
type R3 = IsSubset<symbol, unknown>; // true
```

### Equal

두 타입이 동일하다는 것을 판단하는 타입도 있다.

```ts
type Equal<A, B> = A extends B ? (B extends A ? true : false) : false;
```

하지만 위의 코드는 허점이 있다. boolean이나 never는 유니언이므로 분배법칙이 일어나 분배법칙이 일어나지 않게 바꿔야한다.

또, any와 다른 타입을 구분하기 위해 위의 코드를 수정해야 한다.

```ts
type Equal2<X, Y> = (<T>() => T extends X ? 1 : 2) extends <T>() => T extends Y
  ? 1
  : 2
  ? true
  : false;
```

- 두 타입이 다를 때 타입이 false가 된다는 것을 확인하면 된다.
- X, Y가 서로 다른 경우에는 대부분 쉽게 `Equal2<X, Y>`를 false로 만드는 T를 찾을 수 있다.

### Equal의 반례 T를 찾아보자

| X      | Y      | T      | (<T>() => T extends X ? 1 : 2) | (T<X>) => T extends Y ? 1 : 2 | extends |
| ------ | ------ | ------ | ------------------------------ | ----------------------------- | ------- |
| string | any    | number | 2                              | 1                             | false   |
| any    | string | number | 1                              | 2                             | false   |
| 1      | number | 2      | 2                              | 1                             | false   |

- Equal2 타입을 사용하면 any는 다른 타입과 잘 구별하나 인터섹션을 인식하지 못한다.
- Equal2<any, unknown>의 경우는 extends를 false로 만드는 기가 없으며 false가 됩니다.

### NotEqual

해당 타입이 아닌지 판단하는 NotEqual 타입도 있다.
Equal 타입 결과를 반대로 적용하면 된다.

```ts
type NotEqual<X, Y> = Equal<X, Y> extends true ? false : true;
```

<br />
<br />

# 2.30 타입스크립트의 에러 코드로 검색하자

에러코드마다 숫자가 표시되는데 유형으로 검색하면 더 정확한 검색 결과를 얻을 수 있다.

<br />
<br />

# 2.31 함수에 기능을 추가하는 데코레이터 함수가 있다

타입스크립트 5.0에서는 데코레이터(decorator) 함수가 정식으로 추가되었다.

### 데코레이터?

클래스의 기능을 증강하는 함수로 여러 함수에서 공통으로 수행되는 부분을 데코레이터로 만들어두면 좋다.
클래스 내 중복이 있는 경우 데코레이터를 사용해 중복을 제거할 수 있다.

```ts
function startAndEnd(method: any, context: any) {
  function replaceMethod(this: any, ...args; any[]) {
    console.log("start");
    const result = method.call(this, ...args);
    console.log("end");

    return result;
}
  return replaceMethod;
}
```

- 기존 메서드의 this, 매개변수, 반환값을 각각 This, Args, Return 타입 매개변수로 선언했다. 이들 타입은 그대로 대체 메서드에 적용된다.
- context는 데코레이터의 정보를 갖고 있는 매개변수이다.

### context의 종류를 알아보자!

- classDecoratorContext: 클래스 자체를 장식할 때
- classMethodDecoratorContext: 클래스 메서드를 장식할 때
- classGetterDecoratorContext: 클래스의 getter를 장식할 때
- classSetterDecoratorContext: 클래스의 setter를 장식할 때
- classMemberDecoratorContext: 클래스 멤버를 장식할 때
- classAccessorDecoratorContext: 클래스 accessor를 장식할 때
- classFieldDecoratorContext: 클래스 필드를 장식할 때

> 어떤 문법을 장식하냐에 따라 context의 타입을 교체하면 된다. 또 데코레이터 유형에 따라 속성이 존재하지 않기도 하고 각각의 속성들을 가지고 있다.

- 데코레이터 자체도 함수이므로 매개변수를 가질 수 있으나 고차함수를 활용해야 한다.
- context에는 addInitializer라는 메서드가 있는데 addInitializer에 등록한 함수는 클래스의 인스턴스를 생성할 때(초기화)에 호출된다.
- 데코레이터를 여러 개 붙일 수도 있다.

### 클래스 데코레이터의 특징을 보자!

클래스 데코레이터의 경우 export나 export default 앞이나 뒤에 데코레이터를 붙일 수 있으나 앞과 뒤에 동시에 붙일 수 없다.

```ts
@Log
export class C {}

export
@Log
class C {}

@Log
export class C {}
```

> 데코레이터가 공식적으로 도입되고 타입 지원됨에 따라 타입스크립트의 클래스에서 더 효과적으로 코드를 작성할 수 있게 되었다.

<br />
<br />

# 2.32 앰비언트 선언도 선언 병합이 된다

타입스크립트에서 남의 라이브러리를 사용할 때 그 라이브러리가 자바스크립트라면 직접 타이핑해야 하는 경우가 생긴다. 그때 사용하는 것이 앰비언트 선언(ambient declaration)이다.

앱비언트 선언은 declare 예약어를 사용한다.

```ts
declare namespace NS {
  const v: string;
}
```

- 내부에 구현부가 없는데 외부 파일에 실제 값이 존재한다고 믿기 때문이다.
  - 그래서, 반드시 해당 값이 실제로 존재하는 지 확인해야 한다.
- namespace와 enum에 declare로 선언 시 내부 멤버의 구현부를 생략할 수 있다.
- 인터페이스와 타입 별칭도 declare로 선언할 수 있다.

```ts
declare interface Int {}
declare type T = number;
```

인터페이스와 타입 별칭은 굳이 declare를 붙일 필요는 없다.

### 선언이 생성하는 개체(타입 또는 값으로 사용할 수 있는지)를 알아보자!

| 유형         | 네임스페이스 | 타입 | 값  |
| :----------- | :----------- | :--- | :-- |
| 네임스페이스 | O            | O    | O   |
| 클래스       | O            | O    | O   |
| enum         | O            | O    | O   |
| 인터페이스   | O            | O    |     |
| 타입 별칭    | O            | O    |     |
| 함수         | O            |      | O   |
| 변수         | O            |      | O   |

### 같은 이름의 다른 선언과 병합이 가능한 지 알아보자!

| 병합 가능 여부 | 네임스페이스 | 클래스 | enum | 인터페이스 | 타입 별칭 | 함수 | 변수 |
| :------------- | :----------- | :----- | :--- | :--------- | :-------- | :--- | :--- |
| 네임스페이스   | O            | X      | X    | O          | X         | O    | X    |
| 클래스         | X            | O      | X    | X          | X         | X    | X    |
| enum           | X            | X      | O    | X          | X         | X    | X    |
| 인터페이스     | O            | X      | X    | O          | X         | X    | X    |
| 타입 별칭      | X            | X      | X    | X          | O         | X    | X    |
| 함수           | O            | X      | X    | X          | X         | O    | X    |
| 변수           | X            | X      | X    | X          | X         | X    | O    |

> 인터페이스, 네임스페이스 병합이나 함수 오버로딩 같이 널리 알려진 경우를 제외하고는 왠만하면 같은 이름으로 여러 번 선언하지 않는 것이 좋다.

아래와 같은 경우는 선언 병합을 활용하면 좋다.

```ts
declare class A {
  constructor(name: string);
}
function A(name: string) {
  return new A(name);
}

new A("chu");
A("chu");
```

- 클래스가 있을 때 new를 붙이지 않아도 되게 하는 코드이다.
- class A는 앰비언트 선언이고 function A는 일반 선언이다.
- declare로 앰비언트 선언한 타입도 병합된다.

```ts
function Ex() {
  return "hello";
}
namespace Ex {
  export const a = "world";
  export type B = number;
}

Ex(); // hello
Ex.a(); // world
const b: Ex.B = 123;
```

> 자바스크립트에서는 함수도 객체이므로 함수에 속성을 추가할 수 있다. 함수, 네임스페이스가 병합될 수 있으므로 위의 코드가 에러가 나지 않는다.

> 함수에 속성이 별도로 있다는 걸 알리려면 함수와 동일한 이름의 namespace를 추가하면 된다.
