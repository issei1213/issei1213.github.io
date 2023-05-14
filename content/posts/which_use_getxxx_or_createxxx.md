---
title: "関数命名について getXXX と createXXX の違い"
date: 2023-05-14T23:45:27+09:00
tags: 
  - "週1ゆるアウトプット"
---
## 背景
現在の参画している案件で「週1ゆるアウトプット」という名前でアウトプットしている方がいたので、自分も真似してみようと思う。  
今回は、コードを書いていてふと気になったことがあったので、簡単にまとめていく。

---
## 概要
戻り地がある関数を命名するときに、`createXXX`と`getXXX`のどちらを使うか迷ったので、まとめていく。

---
## 本題
以下はサンプルコードを記載する。
名前のフルネームを作成する関数である。  
`createFullName`と`getFullName`のどちらが適切かをまとめていく。
```typescript
// 1. createXXX
const createFullName = (firstName: string, lastName: string): string => {
  return `${firstName} ${lastName}`
}

// 2. getXXX
const getFullName = (firstName: string, lastName: string): string => {
  return `${firstName} ${lastName}`
}
```

ポイントとしては、`getXXX`は**通常、既存のデータやリソースを取得するときに使用する**ことを想定すること。  
`createXXX`は**新しいデータを作成するときに使用する**。   
なので、今回の場合は、既存のデータやリソースを取得するわけではないので、`createXXX`を使用するのが適切だと判断。

getXXXを使ったサンプルは以下
1. ユーザー情報を取得する場合
```typescript
const getUserInfo = (userId: string) => {
  // ユーザー情報を取得するロジック
};
```

2. 配列から特定の要素を取得する場合
```typescript
const getElementAtIndex = (array: string[], index: number) => {
  return array[index];
};
```
3. オブジェクトのプロパティ値を取得する関数
```typescript
const getPropertyValue = (object: Object, propertyName: string) => {
  return object[propertyName];
};

```

---
## まとめ
今回は、関数の命名について`createXXX`と`getXXX`のどちらを使うか迷ったので、まとめてみた。
`getXXX`は**通常、既存のデータやリソースを取得するときに使用する**ことを想定すること。  
`createXXX`は**新しいデータを作成するときに使用する**。

改めて言語化できたので、スッキリできた。

---
