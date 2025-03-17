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
