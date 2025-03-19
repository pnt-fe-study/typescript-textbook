# 3.1 Partial, Required, Readonly, Pick, Record

Partial, Required, Readonly, Pick, Record는 타입스크립트 Utility Types에서 매핑된 객체 타입을 사용하는 것만 추린 타입이다.

### Partial 함수

기존 객체의 속성을 전부 옵셔널로 만든다.

```ts
type MyPartial<T> = {
  [P in keyof T]?: T[P];
};

type Result = MyPartial<{ a: string; b: number }>;
// type Result = { a?: string | undefined; b?: number | undefined; }
```

1.  keyof T를 통해 T의 모든 키를 가져온다.
2.  [P in keyof T]를 사용하여 가져온 키들(P)을 순회한다.
3.  T[P] → 원래 타입을 유지하면서 각 프로퍼티를 선택적으로(?) 만든다.

- 매핑된 객체 타입으로 기존 객체의 속성을 가져오면서 옵셔널 수식어를 추가로 붙이고 있다.
- 따라서, 모든 객체의 속성이 옵셔널이 된다.

### Required 타입

모든 속성을 옵셔널이 아니게 만들어보자.

```ts
type MyRequired<T> = {
  [P in keyof T]-?: T[P];
};

type Result = MyPartial<{ a?: string; b?: number }>;
// type Result = { a: string; b: number; }
```

### Pick 타입

객체에서 지정한 속성만 추려보자.

```ts
type MyPick<T> = {
  [P in K]: T[P];
};

type Result = MyPick<{ a: string; b: number; c: number }, "a" | "c">;
// type Result = { a: string; c: number }
```

- K타입 매개변수는 T 객체의 속성 이름이어야 하므노 extends keyof T 제약을 준다.
- 객체의 속성 이름이 아닌 것을 무시하고, 추리려면 아래와 같이 수정하면 된다.

```ts
type MyPick<T> = {
  [P in K extends keyof T ? K : never]: T[P];
};

type Result = MyPick<{ a: string; b: number; c: number }, "a" | "c" | "d">;
// type Result = { a: string; c: number }
```

> 매핑된 객체 타입과 컨디셔널 타입을 같이 사용하면 된다. "a" | "c" | "d" 는 제네릭(K)이자 유니언이라 분배법칙이 실행된다.

K extends keyof T ? K : never가 K의 값들을 하나씩 검사하여, T의 키(keyof T)에 속하는 값만 남도록 필터링하는 역할을 한다.

**이 코드의 핵심 개념은**

1. keyof T로 객체의 모든 키를 유니언 타입으로 가져온다.
2. K extends keyof T ? K : never를 통해 K에서 T에 존재하지 않는 키를 never로 제거한다.
3. 조건부 타입의 분배 법칙 덕분에 "d"는 never로 평가되어 제외됨. 4. 결국 "a" | "c"만 남아 새로운 객체 타입을 만든다.

- 다만 위의 방식은 K가 "d"인 경우는 Result가 {}타입이 되어 버린다.

### 분배법칙을 다시 알아본다면?

위의 예시를 기준으로 다시 알아보자.
분배법칙은 유니언 타입이 extends를 만나면 각각 개별적으로 평가된 후 유니언으로 합쳐진다.

즉, 아래처럼 개별적으로 평가된다.

1. "a" extends "a" | "b" | "c" → ✅ "a"
2. "c" extends "a" | "b" | "c" → ✅ "c"
3. "d" extends "a" | "b" | "c" → ❌ never

결과적으로 남는 것은 "a" | "c"이다.

### Record 타입

모든 속성의 타입이 동일한 객체의 타입이 Record타입이다.

```ts
type MyRecord<K extends keyof any, T> = {
  [P in K]: T;
};

type Result = MyRecord<"a" | "b", string>; // Result = {a: string; b: string;}
```

- K extends keyof any를 통해 K에 string | number | symbol로 제약을 건다.
- 제약은 엄격히 걸어야 하는데 속성 이름으로 사용하는 값을 K에 제공할 수 있기 때문이다.

<br />
<br />

# 3.2 Exclude, Extract, Omit, NonNullable

### Exclude

어떠한 타입에서 지정한 타입을 제거하는 타입이다.

```ts
type MyExclude<T, U> = T extends U ? never : T;
type Result = MyExclude<1 | "2" | 3, string>; // Result = 1 | 3
```

- 유니언은 분배법칙으로, 각각의 값들을 확인해 Result 내에 string 타입인 값만 제외된다.

### Extract 타입

어떤 타입에서 지정한 타입만 추출해내는 타입이다.

```ts
type MyExtract<T, U> = T extends U ? T : never;
type Result = MyExtract<1 | "2" | 3, string>; // Result = "2"
```

### Omit 타입

특정 객체에서 지정한 속성을 제거하는 타입이다.

```ts
type MyOmit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
type Result = MyOmit<{ a: "1"; b: 2; c: true }, "a" | "c">; // Result = { b: 2 }
```

- Pick타입과 반대되는 행동을 하고, Pick과 Exclude 타입을 활용한다.
- Exclude<keyof T, K>를 해서 지정한 속성을 제거한다.
- a | b | c 중에 a, c를 지정했으니 "b"만 추려진다.
- 최종적으로 b 속성에만 있는 객체 타입이 남게된다.

### NonNullable 타입

null과 undefined를 제거하는 타입이다.

```ts
// 기존 코드
type MyNonNullable<T> = T extends null | undefined ? never : T;
type Result = MyNonNullable<string | number | null | undefined>; // Result = string | number

// 현재 코드
type MyNonNullable<T> = T & {};
```

- 기존 코드에서는 NonNullable<string> | NonNullable<number> | NonNullable<null> | NonNullable<undefined>라는 분배법칙이 실행된다.
- 여기서, 다시 string | number | never | never 되고 ,최종적으로 string | number가 된다.
- {}는 string, number는 포함하나 null과 undefined는 포함하지 않는다.
- 따라서, T와 {}의 교집합인 string | number가 나오고 간소화될 수 있다.

### 일부 속성만 옵셔널로 만드려면?

옵셔널이 될 속성과 아닌 속성을 구분해야 한다.

```ts
type Optional<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;
type Result = Optional<{ a: "hi"; b: 123 }, "a">;

let test: Result = { b: 123 }; // 선택적(optional) 프로퍼티가 됨
let test1: Result = { c: "hi" }; // Error!
```

1. Pick으로 고른다.
2. Partial을 적용한다.
3. 아닌 속석은 Omit<T, K>로 적용해 & 연산자로 합친다.

<br />
<br />

# 3.3 Parameters, ConstructorParameters, ReturnType, InstanceType

### MyParameters

```ts
type MyParameters<T extends (...args: any) => any> = T extends (
  ...args: infer P
) => any
  ? P
  : never;
```

- T는 함수 타입이어야 한다. (T extends (...args: any) => any 덕분)
- T가 (...args: infer P) => any 형태라면 P를 반환하고, 아니면 never을 반환한다.
- 즉, 이 타입은 함수의 매개변수 타입을 추출하는 역할을 한다.

```ts
type Fn = (x: number, y: string) => boolean;
type Params = MyParameters<Fn>; // [number, string]
```

> Fn의 매개변수 타입 [number, string]이 Params가 된다!

### MyConstructorParameters

```ts
type MyConstructorParameters<T extends abstract new (...args: any) => any> =
  T extends abstract new (...args: infer P) => any ? P : never;
```

- T는 생성자 함수 타입이어야 한다. (T extends abstract new (...args: any) => any)
- T가 abstract new (...args: infer P) => any 형태라면 P를 반환하고, 아니면 never을 반환한다.
- 즉, 생성자 함수의 매개변수 타입을 추출하는 역할을 한다.

```ts
class Person {
  constructor(name: string, age: number) {}
}
```

```ts
type ConstructorParams = MyConstructorParameters<typeof Person>; // [string, number]
```

> Person 클래스의 생성자 매개변수 [string, number]가 추출된다!

### MyReturnType

```ts
type MyReturnType<T extends (...args: any) => any> = T extends (
  ...args: any
) => infer R
  ? R
  : any;
```

- T는 함수 타입이어야 한다.
- T가 (...args: any) => infer R 형태라면 R을 반환하고, 아니면 any를 반환한다.
- 즉, 함수의 반환 타입을 추출하는 역할을 한다.

```ts
type Fn = () => string;
type Return = MyReturnType<Fn>; // string
```

> Fn의 반환 타입 string이 추출된다!

### MyInstanceType

```ts
type MyInstanceType<T extends abstract new (...args: any) => any> =
  T extends abstract new (...args: any) => infer R ? R : any;
```

- T는 생성자 함수 타입이어야 한다.
- T가 abstract new (...args: any) => infer R 형태라면 R을 반환하고, 아니면 any를 반환한다.
- 즉, 클래스의 인스턴스 타입을 추출하는 역할을 한다.

```ts
class Person {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}
```

```ts
type Instance = MyInstanceType<typeof Person>; // Person
```

> Person 클래스의 인스턴스 타입인 Person이 추출된다!

### 여기서 쓰인, extends, infer 등의 역할을 알아보자!

1. `extends (...args: any) => any` : T가 함수 타입인지 확인하는 역할

2. `infer`

- infer P: P를 매개변수 타입으로 추론
- infer R: R을 반환 타입으로 추론

3. `never` : 조건을 만족하지 않을 경우 반환되는 기본 타입

<br />
<br />

# 3.4 ThisType

메서드들에 this를 한 방에 주입하는 타입이다.

```ts
const obj = {
  data: {
    money: 0,
  },
  method: {
    addMoney(amount: number) {
      this.money += amount; // Error!
    },
    useMoney(amount: number) {
      this.money -= amount; // Error!
    },
  },
};

// Error Message!
// Property 'money' does not exist on type '{ addMoney(amount: number): void; useMoney(amount: number): void; }'.
// Property 'money' does not exist on type '{ addMoney(amount: number): void; useMoney(amount: number): void; }'.
```

- this는 obj 객체가 아니라 data, methods 객체를 합친 타입이다.

### this.data.money가 아닌 this.money로 접근하려면?

```ts
type Data = { money: number };
type Methods = {
  addMoney(amount: number): void;
  useMoney(amount: number): void;
};
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
      this.money += amount; // ✅
    },
    useMoney(amount: number) {
      this.money -= amount; // ✅
    },
  },
};
```

> 메서드에 일일이 타이핑하지 않고 메서드를 담고 있는 객체 타입인 Methods에 ThisType<Data & Methods>를 인터섹션 하면 된다.

<br />
<br />

# 3.5 forEach 만들기
