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

### 타입으로 쓸 수 있는 것을 구분하자
- Date, Math, Error, String, Object, Number, Boolean 등과 같은 내장 객체는 타입으로 사용할 수 있습니다.
- String, Object, Number, Boolean, Symbol은 타입으로 사용하는 것을 권장하지 않습니다.
  - Number 간에 연산자를 사용할 수 없고, string에 String을 대입할 수 없습니다.
  - Object 타입인데도 문자열 대입이 가능합니다.
- 변수에 typeof를 앞에 붙여 타입으로 사용할 수 있습니다.
- 클래스의 이름은 typeof 없이도 타입으로 사용할 수 있습니다.
  - ```ts
    class Person {
      name: string;
      constructor(name: string) {
        this.name = name;
      }
    }
    const person: Person = new Person('zero');
    ```

### 유니언 타입으로 OR 관계를 표현하자
- 유니언 타입은 하나의 변수가 여러 타입을 가질 수 있는 가능성을 표시하는 것입니다.
- pasreInt는 자바스크립트에서 pasreInt(1)과 parseInt('1')이 모두 정상적으로 동작 하지만, 타입스크립트에서는 parseInt('1')만 가능합니다.
- 타입스크립트는 if문을 인식합니다.
- 유니언 타입으로부터 정확한 타입을 찾아내는 기법을 타입 좁히기라고 부릅니다.
  - ```ts
    let strOrNum: string | number = 'hello';
    strOrNum = 123;

    if (typeof strOrNum === 'number') {
      strOrNum.toFixed();
    }
    ```
- 유니언은 타입 사이에 | 연산자를 쓸 수 있는 것 뿐만 아니라 타입 앞에도 사용할 수 있습니다.
  - ```ts
    type Union = | string | boolean | number | null;
    ```
