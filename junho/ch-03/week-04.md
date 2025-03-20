## Partial, Required, Readonly, Pick, Record
- 기존 객체의 속성을 전부 옵셔널로 만드는 Partial 타입입니다.
  - ```ts
    type MyPartial<T> = {
        [P in keyof T]?: T[P];
    };
    
    type Result = MyPartial<{ a: string, b: number}>;
    /* 
    type MyPartial<T> = {
        [P in keyof T]?: T[P];
    };
    
    type Result = MyPartial<{ a: string, b: number}>;
    
    type Result = {
        a?: string | undefined;
        b?: number | undefined;
    }
     */
    ```
- 모든 속성을 옵셔널이 아니게 만드는 Required 타입입니다.
  - ```ts
    type MyRequired<T> = {
        [P in keyof T]-?: T[P];
    };
    
    type Result = MyRequired<{ a?: string, b?: number }>;
    /* 
    type Result = {
        a: string;
        b: number;
    }
     */
    ```
- 객체에서 지정한 속성인 Pick 타입입니다.
  - ```ts
    type MyPick<T, K extends keyof T> = {
        [P in K]: T[P];
    };
    
    type Result = MyPick<{ a: string, b: number, c: number }, 'a' | 'c'>;
    /* 
    type MyPick<T, K extends keyof T> = {
        [P in K]: T[P];
    };
    
    type Result = MyPick<{ a: string, b: number, c: number }, 'a' | 'c'>;
    type Result = {
        a: string;
        c: number;
    }
     */
    ```
    - T에 속하지 않는 key값을 입력할 경우 에러가 발생합니다.
      - ```ts
        type Result = MyPick<{ a: string, b: number, c: number }, 'a' | 'c' | 'd'>;
        /* 
        Type '"a" | "c" | "d"' does not satisfy the constraint '"a" | "c" | "b"'.
          Type '"d"' is not assignable to type '"a" | "c" | "b"'.(2344)
         */
        ```
    - ```ts
      type MyPick<T, K> = {
          [P in (K extends keyof T ? K : never)]: T[P];
      };
      
      type Result = MyPick<{ a: string, b: number, c: number }, 'a' | 'c' | 'd'>;
      /* 
      Type '"a" | "c" | "d"' does not satisfy the constraint '"a" | "c" | "b"'.
        Type '"d"' is not assignable to type '"a" | "c" | "b"'.(2344)
       */
      ```
      - 매핑된 객체 타입과 컨디셔널 타입을 같이 사용하면 됩니다.
- 모든 속성의 타입이 동일한 객체의 타입인 Record를 만들어 보겠습니다.
  - ```ts
    type MyRecord<K extends keyof any, T> = {
        [P in K]: T;
    };
    
    type Result = MyRecord<'a' | 'b', string>;
    /* 
    type MyRecord<K extends keyof any, T> = {
        [P in K]: T;
    };
    
    type Result = MyRecord<'a' | 'b', string>;
    
    type Result = {
        a: string;
        b: string;
    }
     */
   ```
   - K extends keyof any에서 K는 모든 객체의 키 타입(string | number | symbol)을 의미합니다.

## 3.2 Exclude, Extract, Omit, NonNullable
- 어떠한 타입에서 지정한 타입을 제거하는 Exclude 타입을 만들어보겠습니다.
  - ```ts
    type MyExclude<T, U> = T extends U ? never : T;
    type Result = MyExclude<1 | '2' | 3, string>;
    // type Result = 1 | 3
    ```
- 어떠한 타입에서 지정한 타입을 추출하는 Extract 타입을 만들어보겠습니다.
  - ```ts
    type MyAbstract<T, U> = T extends U ? T: never;
    type Result = MyAbstract<1 | '2' | 3, string>;
    // type Result = "2"
    ```
- 특정 객체에서 속성을 제거하는 Omit 타입을 만들어보겠습니다.
  - ```ts
    type MyOmit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
    type Result = MyOmit<{ a: '1', b: 2, c: true }, 'a' | 'c'>;
    // type Result = { b: 2 }
    ```
- 
