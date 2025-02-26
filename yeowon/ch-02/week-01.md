### 2-2 타입 추론을 적극 활용하자
 - 암묵적(직접 타입을 표시하지 않아서 타입스크립트가 타입을 추론했다는 의미) any 때문에 발생하는 에러는 implicitAny에러라고 불린다.
 - 'hello', 123, false와 같은 정확한 값을 입력하는 것이 가능하다. -> 리터럴 타입
 - 타입을 표시할 때에는 더 넓은 타입으로 표기하는것이 가능하다.
 - let을 사용하면 const와는 다르게 더 넓게 타입을 추론한다. -> 타입 넓히기

### 2-3 값 자체가 타입인 리터럴 타입이 있다
 - 자바스크립트의 객체는 const 변수라도 수정할 수 있으므로 타입스크립트는 타입을 넓게 추론한다.
 - 값이 변하지 않는 것이 확실하다면 as const라는 특별한 접미사를 붙인다.
   ```ts
    // readonly 수식어가 붙으면 해당 값은 변경 할 수 없다.
    // const obj: { readonly name: 'yeowon' }
   const obj = { name: 'yeowon' } as const; 
   ```

### 2-4 배열 말고 튜플도 있다.
 - 타입스크립트는 배열을 추론할 때 요소들의 타입을 토대로 추론한다. 빈 배열은 any[]로 추론되니 주의해야 한다.
 - 튜플이란, 각 요소 자리에 타입이 고정되어 있는 배열을 의미한다.
 - 추론의 한계
   ```ts
    // arr[3] 은 undefined임에도 toFixed메서드를 붙일 수 있다는 문제
    // arr가 number[]로 추론되면서 arr[3]은 number로 추론되기 때문 -> 튜플로 해결 가능
   const arr = [ 123, 4 ,56 ];
   arr[3].toFixed();

    // 위의 문제를 튜플로 해결하기 -> []안에 정확한 타입을 하나씩 입력하면 해결됨(튜플)
   const tuple:[ number, boolean, string ] = [ 1, false, 'hi' ];
   tuple[0] = 3;
   tuple[2] = 5; // Type 'number' is no assignable to type 'string'
   ```
 - 튜플에서는 push, pop, unshift 메서드를 막지 않으므로, readonly 수식어를 붙여주어야 튜플 수정이 불가능하다.
 - 튜플 외의 특별한 표기로 옵셔널체이닝이 존재하는데, 해당 자리에 값이 있어도그만, 없어도 그만 이라는 의미로 사용되며 표기는 ? 이다.

### 2-5 타입으로 쓸 수 있는 것을 구분하자
 - 타입을 값으로 사용할 수 없다.
 - 타입으로 사용할 수 있는 값: 대부분의 리터럴 값, Date, Math, Errorm String, Object, Number, Boolean등의 내장객체(권장되진x)
 - 타입으로 사용할 수 없는 값: 변수의 이름, 함수의 호출

### 2-6 유니언 타입으로 OR 관계를 표현하자
 - 유니언 타입은 하나의 변수가 여러 타입을 가질 수 있는 가능성을 표시하는 것
 - 함수의 매개변수나 반환값에서도 쓰인다.
   ```ts
   function returnNumber(value: string | number): number {
     return parseInt(value) // error
   }
   // 가능
   returnNumber(1)
   returnNumber('1')
   ```
- 타입스크립트는 if문을 인식한다.(타입 좁히기)
  ```ts
  let strOrNum: string | number = 'hello';
  strOrNum = 123;

  if (typeof strOrNum === 'number') {
    // if문 안에서 strOrNum의 타입은 number
    strOrNum.toFixed();
  }
    // if문 밖에선 strOrNum의 타입은 string
  ```

### 2-7 타입스크립트에만 있는 타입을 배우자
 #### any
  - 타입스크립트가 타입을 검사하지 못하므로 지양해야 할 타입이다.
  - any 타입을 통해 파생되는 결과물도 any 타입이 된다.
  - 배열에 push메서드나 인덱스로 요소를 추가할 때마다 추론하는 타입이 바뀐다
    ```ts
    const arr = []; // any
    arr.push('1');
    arr; // const arr: string[]
    arr.push(3);
    arr; // const arr: (string | number)[]
    
    // 다만, pop으로 요소를 제거할 땐 이전 추론으로 되돌아가지 못함
    arr.pop();
    arr; // const arr: (string | number)[]
    
    const arr2 = [];
    const arr3 = arr2.concat('123'); // error: Variable 'arr2' implicitly has an 'any[]' type.
    ```
 #### unknown
  - any와 비슷하게 모든 타입을 대입할 수 있지만 그 후 어떠한 동작도 수행할 수 없게 된다.
  - 주로 try catch문에서 많이 보게 된다.
    ```ts
    try {
      // ...
    } catch (e) {
      console.log(e.message) // e is of type unknown
      const error = e as Error; // as로 타입을 주장할 수 있다.
      console.log(error.message) // success
    }
    ```
  - 타입을 강제할 때 as 연산자를 주로 사용하나, ! 연산자를 사용해도 된다.(null 뿐만 아니라 undefined도 아님을 주장하는 연산자)
 #### void
  - 함수의 반환값이 없는 경우 반환값이 void타입으로 추론된다.
    ```ts
    // 함수의 반환값이 3(number)인데 void를 반환함.
    const func: () => void = () => 3;

    // void를 반환받은 값의 타입은 void
    const value = func();

    // 반환값의 타입만 따로 표기하는 경우엔 반환값을 무시하지 않음 (3을 무시하지 않아 에러가 발생)
    const func2 = (): void => 3; // number is no assignable to type void

    // 반환값의 타입이 void와 다른 타입의 유니언이면 반환값을 무시하지 않음
    const func3: () => void | undefined = () => 3; // number is no assignable to type void
    ```
 - 사용목적: 사용자가 함수의 반환값을 사용하지 못하도록 제한, 반환값을 사용하지 않는 콜백 함수를 타이필 할 때 사용
 #### {}, object
  - null과 undefined를 제외한 모든 값을 의미한다.
  - 이름은 객체이지만 객체만 대입할 수 있는 타입이 아니다.
  - 대입은 가능하지만 사용할 수 없으므로 object로 타이핑하는 의미가 무색하다.
 #### never
  - 어떠한 타입도 대입할 수 없다.
  - void타입은 never타입에 대입할 수 없다.
    ```ts
    // 함수 선언문
    const neverFunc1() {
      throw new Error('error'); // 반환값이 void
    }
    const result1: never = neverFunc1(); // void is assignable to type never

    // 함수 표현식
    const neverFunc2 = () => {
      throw new Error('error'); // 반환값이 never
    }
    const result2 = neverFunc2(); // const result2: never
    ```

### 2-8 타입 별칭으로 타입에 이름을 붙이자
 - 타입 별칭은 type 키워드를 사용하여 선언할 수 있으며 대문자로 시작하는 단어로 만드는게 관습이다.
   ```ts
   type Person = {
    name: string;
    age: number;
    married: boolean;
   }

   const person: Person = {
    name: 'yeowon',
    age: 31,
    married: false,
   }
   ```

### 2-9 인터페이스로 객체를 타이핑하자
 - 객체 타입에 이름을 붙이는 또하나의 방법으로 인터페이스 선언을 사용하는 것이다. (type Person 대신 interface Person)
 - 인터페이스 속성 키는 객체의 lenght를 제외한 속성 키가 전부 number라는 의미
 - 속성이 없는 인터페이스는 object의 null과 undefined를 제외한 모든 타입을 의미하는 것과 비슷한 역할을 함.
   ```ts
   interface NoProps {}
   const obj: NoProps = {
    why : 'no error',
   }
   const what: NoProps = '이게 되네';
   const omg: NoProps = null; // null 과 Undefined를 제외한 값만 대입할 수 있다.
   ```
 #### 인터페이스 선언 병함
  - 인터페이스 끼리는 서로 합칠 수 있다.
  - 같은 이름으로 여러 인터페이스 선언 가능 -> 선언 병합
  - 다만, 같은 이름의 인터페이스 간 속성이 겹치는데 타입이 다르다면 에러가 발생한다.
 #### 네임스페이스
  - 남이 만든 인터페이스와 의도치 않게 병합된다는 단점을 대비하여 존재하는 것이 네임스페이스이다.
    ```ts
    namespace Exam {
      interface Inner {
        test: string;
      }
      export type test2 = number; // export를 사용하여 네임스페이스 내부 타입을 사용한다.
    }

    const ex1: Exam.Inner = {
      test: 'hello',
    }
    const ex2: Exam.test2 = 123;
    ```
