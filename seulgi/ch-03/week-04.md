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

```
