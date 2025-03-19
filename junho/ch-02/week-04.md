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
- Diff
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
- exclude
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
- 두 타입이 동일하다는 것을 판단하는 방법을 알아보겠습니다.
- Equal
  - ```ts
    type Equal<A, B> = [A] extends [B] ? [B] extends [A] ? true : false : false; 
    ```
    - 다만 any 타입과 다른 타입을 구별하지 못합니다.
      - ```ts
        type Equal2<X, Y> = (<T>() => T extends X ? 1 : 2) extends (<T>() => T extends Y ? 1 : 2)
          ? true
          : false
        ```
        - X, Y가 string, any라 가정할 경우 T에 여러 타입을 넣으면서 한 개라도 false가 나오면 대입할 수 없습니다.
- NotEqual
  - ```ts
    type NotEqual<X, Y> = Equal<X, Y> extends true ? false : true;
    ```

## 2.30 타입스크립트의 에러 코드로 검색하자

## 2.31 함수에 기능을 추가하는 데코레이터 함수가 있다
- 데코레이터는 클래스의 기능을 증강하는 함수로 여러 함수에서 공통으로 수행되는 부분을 데코레이터로 만들면 좋습니다.
  - ```ts
    function startAndEnd(originalMethod: any, context: any) {
        function replacementMethod(this: any, ...args: any[]) {
            console.log('start');
            const result = originalMethod.call(this, ...args);
            console.log('end');
            return result;
        }
        return replacementMethod;
    }
    
    class A {
        @startAndEnd
        eat() {
            console.log('Eat');
        }
    
        @startAndEnd
        work() {
            console.log('Work');
        }
    
        @startAndEnd
        sleep() {
            console.log('Sleep');
        }
    }
    ```
    - 현재 데코레이터가 any로 타이핑되어 있는데 제대로 타이핑하면 다음과 같습니다.
      - ```ts
        function startAndEnd<This, Args extends any[], Return>(originalMethod: (this: This, ...args: Args) => Return, context: ClassMethodDecoratorContext) {
            function replacementMethod(this: any, ...args: any[]) {
                console.log('start');
                const result = originalMethod.call(this, ...args);
                console.log('end');
                return result;
            }
            return replacementMethod;
        }
        ```
        - 기존 메서드의 this, 매개변수, 변환값을 각각 This, Args, Return 타입 매개변수로 선언했습니다.
        - context는 데코레이터의 정보를 갖고 있는 매개변수입니다.
        - context에는 다음과 같은 종류가 있습니다.
          - ClassDecoratorContext: 클래스 자체를 장식할 때
          - ClassMethodDecoratorContext: 클래스 메서드를 장식할 때
          - ClassGetterDecoratorContext: 클래스 getter를 장식할 때
          - ClassSetterDecoratorContext: 클래스 setter를 장식할 때
          - ClassMemberDecoratorContext: 클래스 멤버를 장식할 때
          - ClassAccessorDecoratorContext: 클래스 accessor를 장식할 때
          - ClassFieldDecoratorContext: 클래스 필드를 장식할 때
        - context 객체는 다음과 같은 타입입니다.
          - ```ts
            type Context = {
              kind: string;
              name: string | symbol;
              access: {
                get?(): unknown;
                set?(value: unknown): void;
                has?(value: unknown): boolean;
              };
              private?: boolean;
              static?: boolean;
              addInitializer?(initializer: () => void): void;
            }
            ```
