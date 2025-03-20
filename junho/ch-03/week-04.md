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
- 일부 속성만 옵셔널로 만드는 타입을 만들 수 있습니다.
  - ```ts
    type Optional<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

    type Result = Optional<{ a: 'hi', b: 123 }, 'a'>;
    // type Result = { a?: 'hi', b: 123 }
    ```

## 3.3 Parameters, ConstructorParameters, RuternType, InstanceType
- ```ts
  type MyParameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : never;

  type MyConstructorParameters<T extends abstract new (...args: any) => any> = T extends abstract new (...args: infer P) => any ? P : never;
  
  type MyReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;
  
  type MyInstanceType<T extends abstract new (...args: any) => any> = T extends abstract new (...args: any) => infer R ? R : any;
  ```
  - my를 빼면 모두 lib.es5.d.ts에 존재하는 타입입니다.

## 3.4 ThisType
- ```ts
  const obj = {
      data: {
          money: 0,
      },
      method: {
          addMoney(amount: number) {
              this.money += amount;
          },
          useMoney(amount: number) {
              this.money -= amount;
          }
      }
  }
  // Property 'money' does not exist on type '{ addMoney(amount: number): void; useMoney(amount: number): void; }'.
  ```
  - addMoney나 useMoney에서 this.money에 바로 접근하고 싶은 상황합니다.
    - ```ts
      type Data = { money: number };
      type Methods = {
          addMoney(this: Data & Methods, amount: number): void;
          useMoney(this: Data & Methods, amount: number): void;
      }
      type Obj = {
          data: Data;
          methods: Methods;
      };
      
      
      const obj: Obj = {
          data: {
              money: 0,
          },
          methods: {
              addMoney(amount: number) {
                  this.money += amount;
              },
              useMoney(amount: number) {
                  this.money -= amount;
              }
          }
      }
      ```
      - 메서드에 this를 직접 타이핑하여 에러를 해결했습니다.
      - 다만, 추가할 모든 메서드에 this를 일일이 타이핑해야 하므로 중복이 발생합니다.(this: Data & Methods)
- ThisType 타입을 사용하면 중복을 제거할 수 있습니다.
  - ```ts
    type Data = { money: number };
    type Methods = {
        addMoney(amount: number): void;
        useMoney(amount: number): void;
    }
    type Obj = {
        data: Data;
        methods: Methods & ThisType<Data & Methods>;
    };
    
    
    const obj: Obj = {
        data: {
            money: 0,
        },
        methods: {
            addMoney(amount: number) {
                this.money += amount;
            },
            useMoney(amount: number) {
                this.money -= amount;
            }
        }
    }
    
    const a = obj.methods.addMoney(3);
    ```
    - 메서드를 담고 있는 객체 타입인 Methods에 ThisType <Data & Methods>를 인터섹션하면 this는 Data & Methods가 됩니다.

## 3.5 forEach 만들기
- https://www.typescriptlang.org/play/?ts=5.8.2#code/JYOwLgpgTgZghgYwgAgIJSnAngHgCoB8yA3gFDIXIC2WAYgPZQCiiAFjgNLIC8yA6qAAm9AO4EAFAjgAbaQCNEAawBcycWFbAAzqo4AaZADdVeA8FUgArlTnQDcEwG0AugEoeRQ-WCCDG7egA5gD8uq6qXj4A3KQAvqSkoJCwiCjomLiEJOSUMIwsCKySMvJKMCCq4oYylhAmZiCCEAAeFta2UPYY2E5uHkbevsj+WkGhyHAgWOEDPnEJjgCMegBMegDMzgB0NAzMbOLi7txExLGuMUurG9u7+QdVZvbHpwj0IFr00hBb0vSBj2AejgrnOl2Wa02Ozo90Kh0MZheJDeHy+Pz+AMMoIupCukNuMP2cKqSPWOMcAHJFhS9BSVjSKesKQS9gUiiT+sQUZ9vr9-lUtlppMAkOIAAyubGXMBQWoGNbIRnM6Gsh5Yzk5CjAGBqMBYAAOEHoOsMPG4vApWhloECFPcZEojqMguFoolMUdsWQEGkWhQDqdFEMWzA9FowGaEEE4pxnriOIA9AnkAAFKD0Q1QPWK0PhyOCCnIYQQLTIED0MDe5raSvvYYGlAUqw2aDIAA+yDk9DRkwpW1T6cz2YpuYjUcLxdL5crLRryDresNivgvogfYWEJuWzyRKKMEsIAQYGA7yO2Ud3LRfIBIxx51ISZzmi0heAVH1ruAYGkWGQrDgpaLo2kxYIWthSJYfrIF+Rb0CWZYVn+cCGCgcD1kukzTnAx7vFsG7XFCdy7uI+6HjhIBngGFCXryGLqM+d44rim6EYSbIkQeR4nhR9qasgNHovyt4xLEBjEBMqgUnIhZggkCYAFTIKQIyqFREnIFaUA2iJ5DyQmQA
    
