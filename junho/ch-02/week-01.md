### 타입 추론을 적극 활용하자
- {} 타입은 객체를 의미하는 것이 아니라 null과 undefined를 제외한 모든 타입을 의미합니다.
- let으로 선언한 변수는 다른 값을 대입할 수 있기에 타입을 넓게 추론합니다.
- null과 undefined를 let 변수에 대입할 때는 any로 추론합니다.
- @ts-expect-error는 다음 줄의 코드에 확실한 에러가 있다는 것을 알릴 수 있습니다.

### 값 자체가 타입인 리터럴 타입이 있다
- 타입스크립트는 자바스크립트의 자유도를 희생하는 대신 타입 안전성을 챙기는 언어입니다.
- 자바스크립트의 객체는 const 변수라도 수정할 수 있으므로, 타입스크립트는 수정 가능성을 염두에 두고 타입을 넓게 추론합니다.
- 값이 변하지 않는 것이 확실하다면 as const 접미사를 붙이면 됩니다.
- readonly 수식어가 붙으면 해당 값은 변경할 수 없습니다.

### 배열 말고 튜플도 있다
- 빈 배열은 any[]로 추론 됩니다.
- 각 요소 자리에 타입이 고정되어 있는 배열을 특별하게 튜플이라고 부릅니다.
- 튜플은 push, pop, unshift, shift와 같은 메서드를 사용할 수 있으니 이것까지 막으려면 readonly 수식어를 사용해야 합니다.
  - ```ts
    const tuple: readonly [number, boolean, string] = [1, false, 'hi'];
    ```
- 튜플이 아닌 배열에서도 readonly 수식어를 사용할 수 있습니다.
- 튜플을 길이가 고정된 배열이라고 설정하지 않습니다. ...타입[] 표기를 통해 타입이 연달아 나올 수 있음을 알릴 수 있습니다.
  - ```ts
    const strNumBools = [string, number, ...boolean[]] = ['hi', 123, false, true, false];
    const strNumsBool = [string, ...number[], boolean] = ['hi', 123, 4, 56, false];
    const strsNumBool = [...string[], number, boolean] = ['hi', 'hello', 'wow', 123, false];
    ```
- 타입이 아니라 값이 전개 문법을 사용해도 타입스크립트는 타입 추론을 해냅니다.
  - ```ts
    const arr1 = ['hi', true];
    const arr = [46, ...arr1];
    // const arr: (string | number | boolean)[]
    ```
- 구조분해 할당에서는 나머지 속성을 사용할 수 있습니다.
  - ```ts
    const [a, ...rest] = ['hi', 1, 23, 456];
    // const a: string;
    // const rest: [number, number, number[

    const [b, ...rest2]: [string, ...number[]] = ['hi', 1, 23, 456];
    // const b: string;
    // const rest2: number[]
    ```
- 타입 뒤에 ?가 있으면 옵셔널 수식어로 해당 자리에 값이 있어도 그만, 없어도 그만이라는 의미입니다.
  - ```ts
    let tuple: [number, boolean?, string?] = [1, false, 'hi'];
    tuple = [3, true[;
    tuple = [5];
    ```
