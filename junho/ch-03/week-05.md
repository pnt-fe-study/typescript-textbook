## 3.6 map 만들기
- ```ts
  const r1 = [1, 2, 3].myMap(() => {});
  const r2 = [1, 2, 3].myMap((v, i, a) => v);
  const r3 = ['1', '2', '3'].myMap((v) => parseInt(v));
  const r4 = [{ num: 1 }, { num: 2 }, { num: 3 }].myMap(function(v) {
      return v.num;
  });
  
  interface Array<T> {
      myMap<R>(callback: (v: T, i: number, a: T[]) => R): R[];
  }
  ```
