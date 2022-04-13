---
title: "TypeScript Tip #7"
date: 2022-04-13T23:11:35+09:00
tags: 
  - "TypeScript"
---
## 背景
今回もmattさんの記事をまとめていく。  
<a href="https://twitter.com/mpocock1/status/1502264005251018754" target="_blank">TypeScript Tip #7</a>

---
## 概要
オブジェクト中身を順番に処理していく時には、以下のコードを用いることがある。
```ts
const myObject = {
  a: 1,
  b: 2,
  c: 3
}

Object.keys(myObject).forEach((key) => {
  console.log(myObject[key])
  // Element implicitly has an 'any' type because expression of type 'string' can't be used to index type '{ a: number; b: number; c: number; }'.
  //  No index signature with a parameter of type 'string' was found on type '{ a: number; b: number; c: number; }'.(7053)
})
```
しかし、上記のコードではconsole.logで型エラーが発生する。これは`myObject[key]`のkeyが存在しないオブジェクトのキーを指定しまう可能性があるとして、エラーが発生している。  
参考記事：<a href="https://zenn.dev/katoaki/articles/37a8cff3a8a32a" target="_blank">TypeScriptでブラケット記法を使うときにハマったこと</a>

今回は、この場合の対応策を考えていく。

---
## 本題
実際のサンプルコードは以下になる。
```ts
const myObject = {
    a: 1,
    b: 2,
    c: 3
}

const objectKeys = <Obj>(obj: Obj): (keyof Obj)[] => {
  return Object.keys(obj) as (keyof Obj)[];
}

objectKeys(myObject).forEach((key) => {
  console.log(myObject[key]);
})
```
1. 今回は`objectKeys`関数を作成し、引数`obj`を型推論させた`Obj`を作成する。
例えば、上記のmyObject定数を型推論させると以下になる。
```ts
const myObject = {
    a: 1,
    b: 2,
    c: 3
}

type MyObject = typeof myObject
// type MyObject = {
//   a: number;
//   b: number;
//   c: number;
// }
```
よって、上記でいう`MyObject`が型`Obj`になる。また、ジェネリクスで定義しているため、関数全体で使用できる型になっている。

2. 二ヶ所で`keyof Obj`を使用しているが、これはオブジェクトのkeyを切り取った、ユニオンのリテラル型になる。
例えば、MyObjectの型を定義すると以下になる。
```ts
type KeyOf = keyof MyObject

🔽

type KeyOf = keyof {
    a: number;
    b: number;
    c: number;
}

🔽

type KeyOf = "a" | "b" | "c"
```
3. ここまでで、定数myObjectを型定義をあてはまると、以下になる
```ts
objectKeys(myObject)

const objectKeys = <Obj>(obj: Obj): (keyof Obj)[] => {
  return Object.keys(obj) as (keyof Obj)[];
}

🔽

type Obj = {
    a: number;
    b: number;
    c: number;
}
const objectKeys = <Obj>(obj: Obj): ("a" | "b" | "c")[] => {
    return Object.keys(obj) as ("a" | "b" | "c")[];
    // 型アサーションでa, b, cのどれかが含まれている配列であることを宣言している。
}

console.log(Object.keys(myObject))
//  ["a", "b", "c"] 
```

4. 上記の配列を作成することによって、foreach分の引数の値に型定義がきちんとされ、オブジェクトのキーとしてきちんと利用することができる。
```ts
objectKeys(myObject).forEach((key) => {
  // ここのkeyは以下の様に、型定義が聞いてくれる。
  // key: "a" | "b" | "c"
  console.log(myObject[key]);
})
```

---
## まとめ
今回はforEach分の例にしているが、通常のオブジェクトの操作をしている時に出てくる対処策は以下の記事にかいていたので、  
併せて確認するとさらに勉強になった。
参考記事：<a href="https://zenn.dev/katoaki/articles/37a8cff3a8a32a" target="_blank">TypeScriptでブラケット記法を使うときにハマったこと</a>


---
## 参考文献
<a href="https://twitter.com/mpocock1/status/1502264005251018754" target="_blank">TypeScript Tip #7</a>  
<a href="https://zenn.dev/katoaki/articles/37a8cff3a8a32a" target="_blank">TypeScriptでブラケット記法を使うときにハマったこと</a>

