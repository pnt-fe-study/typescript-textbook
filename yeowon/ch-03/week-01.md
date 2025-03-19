## Partial, Required, Readonly, Pick, Record
**Partial함수**: 기존 객체의 속성을 전부 옵셔널로 만듦
```ts
type MyPartial<T> = {
  [P in keyof T]?: T[P];
}

type Result = MyPartial<{ a:string, b:number }>;
// { a?: string | undefined; b?: number | undefined }
```
**Required**: 모든 속성을 옵셔널이 아니게 만듦
```ts
type MyRequired<T> = {
  [P in keyof T]-?: T[P];
}

type Result = MyRequired<{ a?: string, b?: number }>;
// { a: string; b: number }
```
**readonly**: 모든 속성을 readonly로 만들거나 아니게 만듦
  - 모든 속성을 readonly가 아니게 만드려면 -readonly를 대신 적으면 됨
```ts
type MyReadonly<T> = {
  readonly [P in keyof T]: T[P];
}

type Result = MyReadonly<{ a: string, b: number }>;
// { readonly a: string; readonly b: number }
```
**pick**:객체에서 지정한 속성만 추리기
```ts
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P];
}

type Result = MyPick<{ a:string, b: number, c:number }, 'a' | 'c'>
// { a: string; c: number }
```
**객체의 속성이 아닌 경우는 무시하고 지정한 속성 제거하기**
```ts
type MyPick<T, K> = {
  [P in (K extends keyof T ? K : never)]: T[P];
}

type Result = MyPick<{ a: string, b: number, c:number }, 'a' | 'c' | 'd' >;
// { a: string; c: number }
```
  - 'a' | 'c' | 'd'는 제네릭(K)이자 유니언이므로 분배법칙이 실행됨
  - 다만, K가 d인 경우 Result가 null과 undefined를 제외한 모든값을 의미하는 {}타입이 되어버림

**record**: 모든 속성의 타입이 동일한 객체의 타입
```ts
type MyRecord<K extends keyof any, T> = {
  [P in K]: T;
}

type Result = MyRecord<'a' | 'b', string>;
// { a: string; b: string }
```

## Exclude, Extract, Omit, NonNullable
- 모두 분배법칙을 활용하는 타입!
**Exclude**: 어떤 타입에서 지정한 타입을 제거함
```ts
type MyExclude<T, U> = T extends U ? never : T;
type Result = MyExclude<1 | '2' | 3, string>; // 분배법칙 실행
// 1 | 3
```
**Extrac**: Exclude와 정 반대로 컨디셔널의 타입 참, 거짓 부분만 바꾸면 됨
**Omit**: 특정 객체에서 지정한 속성을 제거하는 타입
```ts
type MyOmit<T, K extends keyof an> = Pick<T, Exclude<keyof T, K>>;
type Result = MyOmit<{ a: '1', b: 2, c: true }, 'a' | 'c'>;
// { b: 2}
```
**NonNullable**: null과 undefined를 제거하는 타입
```ts
type MyNonNullable<T> = T extends null | undefined ? never : T;

// old code
type Result = MyNonNullable<string | number | null | undefined>;
// new code
type MyNonNullable<T> = T & {}
// string | number
```
**응용해서 일부 속성만 옵셔널로 만들기**
```ts
type Optional<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>
type Result = Optional<{ a: 'hi', b: 123 }, a>
// { a?: 'hi', b: 123 }
```
  - 옵셔널이 될 속성은 Pick으로 고른 후 Partial을 적용하고 아닌 속성들은 Omit으로 추린 후 & 연산자로 합친다.

## Parameters, ConstructorParameters, Return Type, InstanceType
- infer를 활용한 타입!
```ts
type MyParameters<T extends (...args:any) => any>
  = T extends (...args: infer P) => any ? P : never;

type MyConstructorParameters<T extends abstract new (...args: any) => any>
  = T extends abstract new (...args: infer P) => any ? P : never;

type ReturnType<T extends (...args: any) => any>
  = T extends (...args: any) => infer R ? R : any;

type MyInstanceType<T extends abstract new (...args: any) => any>
  = T extends abstract new (...args: any) => infer R ? R : any;
```

## ThisType
- this를 한 방에 주입하는 타입
```ts
const obj = {
  data: {
    money: 0,
  }
  methods: {
    addMoney(amount: number){
      this.money += amount;
    },
    useMoney(amount: number) {
      this.money -= amount;
    }
  }
}
// 메서드들 this.methods.addMoney~ 로 접근해야 함
```
```ts
type Data = { money: number };
type Methods = {
  addMoney(amount: number): void;
  useMoney(amount: number): void;
};
type obj = {
  data: Data;
  methods: Methods & ThisType<Data & Methods>;
};
const obj: Obj = {
  data: {
    money: 0,
  }
 },
 methods: {
   addMoney(amount) {
      this.money += amount;
    }
   useMoney(amount) {
      this.money -= amount;
    }
 }
```
- 메서드에 직접 this를 타이필 할 수 있지만 일일이 타이핑 해야하는 불편함이 있다. 이럴 때 ThisType타입을 사용하면 중복을 줄일 수 있다.

## forEach 만들기
```ts
[1, 2, 3].myForEach(() => {});

interface Array<T> {
  myForEach(callback: () => void): void;
}
```
```ts
[1, 2, 3].myForEach(() => {});
[1, 2, 3].myForEach((v, i, a) => { console.log(v, i, a) });
[1, 2, 3].myForEach((v, i) => console.log(v));
[1, 2, 3].myForEach((v) => 3);

interface Array<T> {
  myForEach(callback: (v: number, i: number, a: number[]) => void): void;
}
```
```ts
[1, 2, 3].myForEach(() => {});
[1, 2, 3].myForEach((v, i, a) => { console.log(v, i, a) });
[1, 2, 3].myForEach((v, i) => console.log(v));
[1, 2, 3].myForEach((v) => 3);
['1', '2', '3'].myForEach((v) => {console.log(v.slice(0))});

[true, 2, '3'].myForEach((v) => {
    if(typeof v === 'string') {
        v.slice(0); // error
    } else {
        v.toFixed() // 여기 에러가 나타나야 함
    }
})

interface Array<T> {
  myForEach(callback: (v: number, i: number, a: number[]) => void): void;
}
```
- 에러가 발생해야 하는데 발생하지 않는 상황도 문제
- 원인: 각각 요소와 원본 배열의 타입인 매개변수 v와 a가 모두 number로 고정되어 있기 때문 -> number대신 제네릭 사용
```ts
...
[true, 2, '3'].myForEach((v) => {
    if(typeof v === 'string') {
        v.slice(0); 
    } else {
        v.toFixed() // error
    }
})

interface Array<T> {
  myForEach(callback: (v: T, i: number, a: T[]) => void): void;
}
```
**liv.es5.d.ts**
```ts
interface Array<T> {
  forEacth(callbackfn: (value: T, index: number: array:T[] => void, thisArgs?: any): void 
}
```
- thisArgs는 콜백 함수 선언문에서 this를 사용할 때 직접 바꿀 수 있게 하는 부분

```ts
...
[1, 2, 3].myForEach(function() {
  console.log(this); // this: window
});
[1, 2, 3].mtForEach(function() {
  console.log(this) // a: string
}, {a: 'b'});
```
