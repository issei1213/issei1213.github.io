---
title: "TypeScript Tip #5"
date: 2022-04-10T12:43:14+09:00
tags: 
  - "TypeScript"
---
## 背景
今回もmattさんの動画をまとめていく  
<a href="https://twitter.com/mpocock1/status/1500813765973053440" target="_blank">TypeScript Tip #5</a>

---
## 概要
型推論された型に対してextendsを使用するまとめていく。

---
## 本題
実際のサンプルコードは以下。
`getDeepValue`関数の引数にオブジェクトを渡して、そこから型推論をさせている。  
第2引数、第3引数のオブジェクトのキーを指定して、最終的なバリューを取得できるような関数を作成する。
```ts
export const getDeepValue = <
      Obj, 
      FirstKey extends keyof Obj, 
      SecondKey extends keyof Obj[FirstKey]
  >(
      obj: Obj,
      firstKey: FirstKey,
      secondKey: SecondKey
    ): Obj[FirstKey][SecondKey] => {
        return  obj[firstKey][secondKey]
}

const obj = {
  foo: {
    a: true,
    b: 2
  },
  bar: {
    c: 'cool',
    d: 2
  }
}

console.log(getDeepValue(obj, 'bar', 'd'))
```

1. 引数の第2引数`FirstKey`は型推論された`Obj`に制約を設けている。
```ts
FirstKey extends keyof {
    foo: {
        a: boolean;
        b: number;
    };
    bar: {
        c: string;
        d: number;
    };
}

🔽

FirstKey extends "foo" | "bar"

```
よって、FirstKeyは`foo`か`bar`のどちらかになる。


2. 引数の第3引数`SecondKey`は型推論された`Obj`と上記の`FirstKey`のルックアップ型で制約を設けている。  
例えば、第2引数が`foo`の場合、以下の様になる。
```ts
<
      Obj, 
      FirstKey extends 'foo', 
      SecondKey extends keyof Obj['foo']
>
    
🔽

<
    Obj,
    FirstKey extends 'foo',
    SecondKey extends "a" | "b"
>
```
上記の様に型推論され、第2引数の値によって第3引数の値も変更されるため、より安全に型定義できる。

よって、以下の様なパターンは型エラーとなる。
```ts
console.log(getDeepValue(obj, 'foo', 'd'))
// Argument of type '"d"' is not assignable to parameter of type '"a" | "b"'.

console.log(getDeepValue(obj, 'bar', 'a'))
// Argument of type '"a"' is not assignable to parameter of type '"c" | "d"'
```
---
## まとめ
今回は短めだったが、型推論は強力のため、うまく使用すれば汎用的な型定義ができそうだなと感じた。  
意外とkeyofを使用した時に、リテラル型のユニオンが生成されることが抜けていたので勉強になった。

---
## 参考文献
<a href="https://twitter.com/mpocock1/status/1500813765973053440" target="_blank">TypeScript Tip #5</a>

