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
type IsAny<T> = [T] extends (number & T) ? true : false;
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

예를 들어 A가 `{ name: string, age: number }`, B가  `{ name: string, married: boolean }`인 경우 둘을 차집합(A-B)하면  `{ age: number }`가 나와야 한다. 

```ts
type Diff<A, B> = Omit<A & B, keyof B>;
type R1 = Diff<{ name: string, age: number }, { name: string, married: boolean }>;
```

### Omit

특정 객체에서 지정한 속성을 제거하는 타입이다. A & B는 { name: string, age: number, married: boolean }인데 keyof B는 `name | married`이므로 name과 married 속성을 제거하면 age 속성만 남게 된다.

### Diff 

Diff 타입을 조금 응용하면 대칭차집합도 찾아낼 수 있다. 서로 겹치지 않는 부분을 합쳐놓은 것으로 합집합에서 교집합을 뺀 것으로 볼 수 있다.

```ts
type SymDiff<A, B> = Omit<A & B, keyof (A | B)>;
type R2 = SymDiff<{ name: string, age: number }, {name: string, married: boolean }>
```

> 해당 코드는 객체만 적용 가능하다. 유니언에서는 적용되지 않는다.

```ts
type SymDiffUnion<A, B> = Exclude<A | B, A & B>;
type R3 = SymDiffUnion<1 | 2 | 3, 2 | 3 | 4>;
```

### Exclude

어떤 타입(A | B)에서 다른 타입 (A & B)을 제거하는 타입이다.

```ts

```
