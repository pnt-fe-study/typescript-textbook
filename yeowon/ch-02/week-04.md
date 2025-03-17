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

