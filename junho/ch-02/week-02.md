## 객체의 속성과 메서드에 적용되면 특징을 알자
- 객체의 속성도 옵셔널이나 readonly 수식어가 가능합니다.
- 객체 리터럴을 대입했냐, 변수를 대입했냐에 따라 타입 검사 방식이 달라집니다.
  - ```ts
    interface Example {
      hello: string;
    }

    // 객체 리터럴
    const example: Example = {
      hello: 'hi',
      why: '나만 에러야',
    }
    // Type '{ hello: string; why: string; }' is not assignable to type 'Example'. Object literal ...

    const obj = {
      hello: 'hi',
      why: '나는 에러 아냐',
    }

    // 변수 대입
    const example2: Example = obj;
    ```
    - 참고로 에러 메시지에는 여러 에러 메시지가 동시에 표시될 수 있습니다.
    - 두 번째 에러가 첫 번째 에러보다 더 구체적인 에러입니다.
    - 에러를 분석할 때 에러 메시지가 여러 개 있다면 위에서 아래로 읽으면서 구체적인 에러의 위치를 찾아나가면 됩니다.
  - 객체 리터럴을 대입하면 `잉여 속성 검사`가 실행됩니다.
  - `잉여 속성 검사`는 타입 선언에서 선언하지 않은 속성을 사용할 때 에러를 표시하는 것을 의미합니다.

### 인덱스 접근 타입
- ```ts
  type Animal = {
    name: string;
  }

  type N1 = Animal['name'];
  // type N1 = string

  type N2 = Animal["name"];
  // type N2 = string
  
  type N3 = Animal.name;
  // Cannot access 'Animal.name' becuase ...
  ```
  - 자바스크립트에서 객체의 속성에 접근하듯 접근합니다.
  - 다만, 객체.속성 방식은 사용할 수 없습니다.
  - 객체 속성의 타입에 접근하는 방식을 `인덱스 접근 타입`이라고 부릅니다.
- 키의 타입은 `keyof` 객체 타입이고 값의 타입은 객체 타입[키의 타입] 입니다.
  - ```ts
    const obj = {
      hello: 'world',
      name: 'zero',
      age: 28,
    };
    type Kyes = keyof typeof obj;
    // type Keys = 'hello' | 'name' | 'age'

    type Values = typeof obj[Keys];
    // type Values = string | number
    ```
    - obj는 값이기에 타입 자리에 바로 쓸 수 없습니다.
    - typeof 연산자로 타입을 만들어 주었습니다.
- keyof의 특성 몇 가지를 확인합니다.
  - keyof any는 string | number | symbol 입니다.
  - 배열에 keyof를 적용하면 number | 배열 속성 이름 유니언 | 배열 인덱스 문자열 유니언이 됩니다.
- 튜플과 배열에도 인덱스 접근 타입을 사용할 수 있습니다.
  - ```ts
    type ...

    type Arr2 = (string | boolean)[];
    type El = Arr2[number];
    // type El = string | boolean;
    ```
    - El 타입처럼 배열[number] 인덱스 접근 타입으로 배열 요소들의 타입을 모두 가져올 수 있습니다.
- 객체의 메서드를 선언할 때는 세 가지 방식으로 할 수 있습니다.
  - ```ts
    interface Example {
      a(): void;
      b: () => void;
      c: {
        (): void;
      }
    }
    ```

### 매핑된 객체 타입
- 매핑된 객체 타입이란 기존의 다른 타입으로부터 새로운 객체 속성을 만들어내는 타입을 의미합니다.
- 인터페이스에서는 쓰지 못하고 타입 볊칭에서만 사용할 수 있습니다.
  - ```ts
    type HelloANdHi = {
      [key in 'hello' | 'hi']: string;
    };
    /*
    type HelloAndHi = {
      hello: string;
      hi: string;
    }
    */
    ```
    - in 연산자를 사용해서 인덱스 시그니처가 표현하지 못하는 타입을 표현합니다.
    - in 연산자 오른쪽에는 유니언 타입이 와야 합니다.
- 튜플과 배열도 객체이므로 매핑된 객체 타입을 적용할 수 있습니다.
- 다른 타입으로부터 값을 가져오면서 수식어를 붙일 수도 있습니다.
  - ```ts
    interface Original {
      name: string;
      age: number;
      married: boolean;
    }
    type Copy = {
      readonly [key in keyof Original]?: Original[key];
    }
    

    interface Original {
      name: string;
      age: number;
      married: boolean;
    }
    type Copy = {
      readonly [key in keyof Original]?: Original[key]
    }
    /*
    type Copy = {
        readonly name?: string | undefined;
        readonly age?: number | undefined;
        readonly married?: boolean | undefined;
    }
    */
    ```
- 수식어 앞에 -를 붙이면 해당 수식어가 제거된 채로 속성을 가져옵니다.
  - ```ts
    interface Original {
      readonly name?: string;
      readonly age?: number;
      readonly married?: boolean;
    }
    type Copy = {
      -readonly [key in keyof Original]-?: Original[key];
    }
    /*
    type Copy = {
        name: string;
        age: number;
        married: boolean;
    }
    */
    ```
    
