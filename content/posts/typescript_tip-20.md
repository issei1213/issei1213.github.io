---
title: "TypeScript Tip #20"
date: 2022-04-20T23:30:11+09:00
tags: 
- "TypeScript"
---
## 背景
今回もmattさんのTypeScriptTipsを解説していく。  
<a href="https://twitter.com/mpocock1/status/1516755042816172042" target="_blank">TypeScript Tip #20</a>

---
## 概要
以下のオブジェクトの型が存在している時に、特定のキーのバリューのユニオンを作成する時に使用する。
```typescript
export type Obj = {
    a: 'a',
    a2: 'a2',
    a3: 'a3',
    b: 'b',
    b1: 'b1',
    b2: 'b2'
}

// 上記の型から以下の型を作成したい
type NewUnion = "a" | "a2" | "a3"
```
---
## 本題
以下が実際の型定義になる。順番に解説していく。
```typescript
type ValuesOfKeysStartingWithA<Obj> = {
    [K in Extract<keyof Obj, `a${string}`>]: Obj[K];
}[Extract<keyof Obj, `a${string}`>]

type NewUnion = ValuesOfKeysStartingWithA<Obj>
// type NewUnion = "a" | "a2" | "a3"
```
1. 今回のポイントは``Extract<keyof Obj,`a${string}`>``になるため細かくみていく。
まず、`a${string}`がどのように型定義されるかというと、文字列`a`を含めて文字列リテラル型を定義できる。
```typescript
type TestType = `a${string}`

// Success
const test1: TestType = 'a'
const test2: TestType = 'a1'

// Error
const test3: TestType = '1'  // Type '"1"' is not assignable to type '`a${string}`'.
const test4: TestType = 1    // Type '1' is not assignable to type '`a${string}`'.
```
次に`keyof Obj`は、オブジェクト型のキーのユニオンのリテラル型を作成できる
```typescript
type Keys = keyof Obj
// "a" | "a2" | "a3" | "b" | "b1" | "b2"
```
そこで``Extract<keyof Obj,`a${string}`>``に組み合わせると、以下になる。     
また、`Extract<T,U>`は、`T`型から`U`型に代入可能な型だけを抽出するユーティリティになる。  
参考記事：<a href="https://qiita.com/k-penguin-sato/items/e2791d7a57e96f6144e5#extracttu" target="_blank">【TypeScript】Utility Typesをまとめて理解する</a>
```typescript
Extract<keyof Obj,`a${string}`>
    
🔽

Extract<"a" | "a2" | "a3" | "b" | "b1" | "b2", `a${string}`>
// "a" | "a2" | "a3" | "b" | "b1" | "b2"からa${string}に代入可能な型を抜き出す

🔽

"a" | "a2" | "a3"
```
よって、``Extract<keyof Obj,`a${string}`>``は`"a" | "a2" | "a3"`の型になる。
2. Extract型がわかったので、元のコードに当てはめてみる
```typescript
type ValuesOfKeysStartingWithA<Obj> = {
    [K in "a" | "a2" | "a3"]: Obj[K];
}["a" | "a2" | "a3"]
```
まずはインデックスシグネチャだけを取り出してみる。
```typescript
type ValuesOfKeysStartingWithA<Obj> = {
    [K in "a" | "a2" | "a3"]: Obj[K];
}

🔽

type ValuesOfKeysStartingWithA<Obj> = {
    'a': Obj['a'],
    'a2': Obj['a2'],
    'a3': Obj['a3'],
}

🔽

type ValuesOfKeysStartingWithA<Obj> = {
    'a': 'a',
    'a2': 'a2',
    'a3': 'a3',
}
```
上記に展開したインデックスシグネチャにルックアップ型を追加する。  
ルックアップ型がユニオンになっているので、取り出し型もユニオンになる。
```typescript
type ValuesOfKeysStartingWithA<Obj> = {
    'a': 'a',
    'a2': 'a2',
    'a3': 'a3',
}["a" | "a2" | "a3"]

🔽

"a" | "a2" | "a3"
```
これで特定のキーのバリューのユニオンを作成することができた。  
3. コードをよくみると``Extract<keyof Obj, `a${string}`>``が二ヶ所存在しており、少し冗長である。  
そのため、ジェネリクスで定義してやる書き方が以下になる。
```typescript
type ValuesOfKeysStartingWithA<Obj, _ExtractedKeys extends keyof Obj = Extract<keyof Obj, `a${string}`>> = {
    [K in _ExtractedKeys]: Obj[K];
}[_ExtractedKeys]


type NewUnion = ValuesOfKeysStartingWithA<Obj>
```
ジェネリクスで型`_ExtractedKeys`に``keyof Obj = Extract<keyof Obj, `a${string}`>>``の制約をつけている。  
これはデフォルト型引数を使用しており、`keyof Obj`が存在しない、つまりジェネリクスの第2引数が存在しない場合、
``Extract<keyof Obj, `a${string}`>>``が``_ExtractedKeys``の型になる。  
``Extract<keyof Obj, `a${string}`>>``が``_ExtractedKeys``の型になる。  
参考記事：<a href="https://typescriptbook.jp/reference/generics/default-type-parameter" target="_blank">デフォルト型引数</a>  
よって今回の場合、型`_ExtractedKeys`は`"a" | "a2" | "a3"`に推論される。

---
## まとめ
過去にTipsのコードリーディングをしていたので、大分型の流れを掴めるようになってきた。  
ジェネリクスのデフォルト型引数は久しぶりに復習できてよかった。

---
## 参考文献
<a href="https://twitter.com/mpocock1/status/1516755042816172042" target="_blank">TypeScript Tip #20</a>  
<a href="https://qiita.com/k-penguin-sato/items/e2791d7a57e96f6144e5#extracttu" target="_blank">【TypeScript】Utility Typesをまとめて理解する</a>  
<a href="https://typescriptbook.jp/reference/generics/default-type-parameter" target="_blank">デフォルト型引数</a>

