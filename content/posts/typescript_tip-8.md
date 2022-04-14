---
title: "TypeScript Tip #8"
date: 2022-04-15T00:14:16+09:00
tags: 
  - "TypeScript"
  - 'React'
---
## 背景
今回もmattさんのTypeScriptTipsをまとめていく。  
<a href="https://twitter.com/mpocock1/status/1503352924537339904" target="_blank">TypeScript Tip #8</a>

---
## 概要
Reactのジェネリクスを使用して、動的で柔軟なコンポーネントを作成することができる。

---
## 本題
以下のコードはReactのサンプルのコードである。  
`Table`コンポーネントの`props`に存在する`items`はオブジェクトの配列になっている。  
オブジェクトはプロパティ`id`のみを保持している。
```tsx
import React from 'react'

interface TableProps {
    items: { id: string }[]
    renderItem: (item: { id: string }) => React.ReactNode
}

export const Table = (props: TableProps) => {
    return null
}

const Component = () => {
    return (
        <Table
            items={[
                {
                    id: '1',
                }
            ]}
            renderItem={(item) => <div>{ item.id }</div>}
        ></Table>
    )
}
```
上記の場合、props`items`にオブジェクトの`id`以外のプロパティを定義したい場合、`TableProps`にも追加をしてやる必要がある。
以下はそのサンプルコード
```tsx
import React from 'react'

interface TableProps {
    items: { id: string, name: string }[]
    // 🔼プロパティnameを追加している
    renderItem: (item: { id: string, name: string }) => React.ReactNode
    // 🔼プロパティnameを追加している
}

export const Table = (props: TableProps) => {
    return null
}

const Component = () => {
    return (
        <Table
            items={[
                {
                    id: '1',
                    name: 'issei'
                    // 🔼propsにnameを追加している
                }
            ]}
            renderItem={(item) => <div>{ item.id }</div>}
        ></Table>
    )
}
```

プロパティを追加する度に型定義を追加してやる必要があるため、これをジェネリクスを利用して定義を動的に変更させる。
ジェネリクスを利用したサンプルコードは以下。
```tsx
import React from 'react'

interface TableProps<TItem> {
    items: TItem[]
    renderItem: (item: TItem) => React.ReactNode
}

export function Table<TItem>(props: TableProps<TItem>) {
    return null
}

const Component = () => {
    return (
        <Table
            items={[
                {
                    id: '1',
                    name: 'Matt'
                }
            ]}
            renderItem={(item) => <div>{ item.id }</div>}
        ></Table>
    )
}
```
1. `TableProps`の型定義でジェネリクスを定義している。  
今回の例でいうと、itemsの型推論した型をジェネリクスに格納し、型全体で使用できる様にしている。  
renderItem関数内の引数にもジェネリクスで定義した型を定義している。
```tsx
interface TableProps<TItem> {
    items: TItem[]
    renderItem: (item: TItem) => React.ReactNode
}
```
2. 上記で定義したジェネクスも関数全体で使用できるように定義しておく。
```tsx
export function Table<TItem>(props: TableProps<TItem>) {
  return null
}

// or

const Table = <TItem extends unknown>(props: TableProps<TItem>) => {
    return null;
};

```
3. 上記の型を定義してやることで、propsの値を型推論してくれて、itemsのプロパティを追加しても動的に型推論してくれる。
```tsx
const Component = () => {
  return (
    <Table
      items={[
          {
            id: '1',
            name: 'Matt'
            // 🔼nameを追加しても動的に型推論することができる
          }
        ]}
      renderItem={(item) => <div>{ item.id }</div>}
    ></Table>
    // 型定義内容
    //  function Table<{
    //    id: string;
    //    name: string;
    //   }>(props: TableProps<{
    //    id: string;
    //    name: string;
    //   }>): null
    //  )
}
```

---
## まとめ
かなり汎用性が高くなるため、使用することがあれば使用していきたい

---
## 参考文献
<a href="https://twitter.com/mpocock1/status/1503352924537339904" target="_blank">TypeScript Tip #8</a>
