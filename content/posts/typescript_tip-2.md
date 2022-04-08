---
title: "TypeScript Tip #2"
date: 2022-04-08T20:26:20+09:00
tags: 
  - "TypeScript"
---

## 背景
今回もmattoさんのTypeScriptの動画をまとめていく

## 概要
下記の様な`Entity`の型を定義する。
```typescript
type Entity = 
    | {
        type: 'user'
    }
    | {
        type: 'post'
    }
    | {
        type: 'comment'
    }
```
上記の型の各オブジェクトにidを追加したい。その時のコードは以下になる。  
typeに応じたidのプロパティを追加している。
```typescript
type EntityWithId = 
    | {
        type: 'user'
        userId: string
    }
    | {
        type: 'post'
        postId: string
    }
    | {
        type: 'comment'
        commentId: string
    }

const result1: EntityWithId = {
    type: 'post',
    postId: '123'
}

const result2: EntityWithId = {
    type: 'user',
    userId: '123'
}
```
今回は`EntityWithId`の部分をインデックスシグネチャを利用して作成していく。

## 本題
実際のコードは以下になる。細かくみていく。
```typescript
type EntityWithId = {
    [ EntityType in Entity['type'] ]: {
        type: EntityType
    } & Record<`${EntityType}Id`, string>
}[Entity['type']]
```

1. `[ EntityType in Entity['type'] ]`はルックアップ型とマップ型を使用しており、ルックアップ型を展開すると、以下になる。  
以下のコードをマップ型でインデックスシグネチャを作成している。
```typescript
[ EntityType in "user" | "post" | "comment" ]
```
2. `[Entity['type']]`の部分はユニオンのルックアップ型になっている。よって、`["user" | "post" | "comment"]`になる。
```typescript
type EntityWithId = {
    [ EntityType in "user" | "post" | "comment" ]: {
        type: EntityType
    }
}[Entity['type']]
```
3. `Record`を除く以下の型の定義になる。
```typescript
type EntityWithId = {
    [EntityType in Entity['type']]: {
        type: EntityType
    }
}[Entity['type']]

🔽

type EntityWithId = {
    [ EntityType in "user" | "post" | "comment" ]: {
        type: EntityType
    }
}["user" | "post" | "comment"]

🔽

type EntityWithId = {
    type: "user";
} | {
    type: "post";
} | {
    type: "comment";
}


```
4. 今回のポイントであるRecord型では、テンプレートリテラル型とルックアップ型で定義している。
```typescript
Record<`${EntityType}Id`, string>
```
`EntityType`はマップ型で文字列リテラル型になるため、`"user" | "post" | "comment"`のどれかになる。  
例えば、`user`の場合は以下になる。Record型によって、オブジェクト型が定義できる。
```typescript
Record<`${user}Id`, string>
    
🔽

{
    userId: string
}
```
5. 最後は`&`でインターセクション型(交差型)で定義している

## まとめ
Record型のテンプレートリテラル型が使用できることが勉強になった。

## 参考文献
<a href="https://twitter.com/mpocock1/status/1498284926621396992" target="_blank">TypeScript Tip #2</a>  
