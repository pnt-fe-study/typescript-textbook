## 2.29 배운 것을 바탕으로 타입을 만들어보자

### 2.29.1 판단하는타입 만들기
- IsNever
  - ```ts
    type IsNever<T> = [T] extends [never] ? true : false;
    ```
- IsAny
  - ```ts
    type IsAny<T> = string extends (number & T) ? true : false;
    ```
- IsArray
  - ```ts
    type IsArray<T> = IsNever<T> extends true 
        ? false 
        : T extends readonly unknown[]
            ? IsAny<T> extends true
                ? false
                : true
            : false
    ```
    - T extends readonly unknown[] ? true : false 만 사용하지 않는 이유는 never일 때는 never가 any일 때는 boolean이 반환되어 예외 처리를 해줘야 합니다.
- IsTuple
  - ```ts
    type IsTuple<T> = IsNever<T> extends true
      ? false
      : T extends readonly unknown[]
          ? number extends T["length"]
              ? false
              : true
          : false;
    ```
    - 배열의 T["length"]는 number가 되고 튜플의 T["length"]는 구체적인 숫자가 됩니다.
- IsUnion
  - ```ts
    type IsUnion<T, U = T> = IsNever<T> extends true
      ? false
      : T extends T
          ? [U] extends [T]
              ? false
              : true
          : false;
    ```
    - T extends T를 사용하는 이유는 분배법칙을 만들기 위해서 사용합니다.

### 2.29.2 집합 관련 타입 만들기
- 차집합을 만들어 보겠습니다.
  - ```ts
    type Diff<A, B> = Omit<A & B, keyof B>;
    type R1 = Diff<{ name: string, age: number }, { name: string, married: boolean }>;
    // type R1 = { age: number }
    ```
    - Omit 타입은 특정 객체에서 지정한 속성을 제거하는 타입입니다.
    - Diff 타입을 응용하면 대칭차집합을 구할 수 있습니다.
      - ```ts
        type SymDiff<A, B> = Omit<A & B, keyof (A | B)>;
        type R2 = SymDiff<{ name: string, age: number }, { name: string, married: boolean }>;
        // type R2 = { married: boolean, age: number }
        ```
- 유니언에 대칭차집합을 적용하겠습니다.
  - ```ts
    type SymDiffUnion<A, B> = Exclude<A | B, A & B>;
    type R3 = SymDiffUnion<1 | 2 | 3, 2 | 3 | 4>;
    // type R3 = 1 | 4
    ```
    - Exclude는 어떤 타입에서 다른 타입을 제거하는 타입입니다.
      - ```ts
        type IsSubset<A, B> = A extends B ? true : false;
        type R1 = IsSubset<string, string | number>;
        // type R1 = true
        
        type R2 = IsSubset<{ name: string, age: number }, { name: string }>;
        // type R2 = true
        
        type R3 = IsSubset<symbol, unknown>;
        // type R3 = true
        ```
