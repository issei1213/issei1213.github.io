---
title: "Typescript Tip #1"
date: 2022-04-07T12:43:39+09:00
tags: 
  - "TypeScript"
---
## 背景
前回のmattさんのTypeScript Tip #15が勉強になったので、最初から#1からまとめていく

## 概要
```typescript
export const fruitCounts = {
    apple: 1,
    pear: 4,
    banana: 26
}
```
上記の様なコードがあった時に、特定のキーを取得する型エイリアスで定義すると以下になる。
```typescript
type SingleFruitCount = 
    | {
        apple: number    
    }
    | {
        banana: number
    }
    | {
        pear: number
    }

const singleFruitCount: SingleFruitCount = {
    apple: 10,
}
```
`SingleFruitCount`の型エイリアスを作成すれば、型定義できるが定数`fruitCounts`を型推論させて型エイリアスを作成する。

## 本題
最終的なコードは以下になる。具体的に解説していく。
```typescript
type FruitCounts = typeof fruitCounts

type NewSingleFruitCount = {
    [ K in keyof FruitCounts ]: { 
        [K2 in K]: number
    }
}[keyof FruitCounts]

//  type NewSingleFruitCount = {
//      apple: number;
//    } | {
//      pear: number;
//    } | {
//      banana: number;
//  }
```

1. `typeof fruitCounts`は型推論が効いて以下の型が作成される。
```typescript
type FruitCounts = {
    apple: number;
    pear: number;
    banana: number;
}
```
2. `[ K in keyof FruitCounts ]`の`keyof FruitCounts`は型エイリアスのオブジェクトのキーのみを取得することができる。  
よって、`keyof FruitCounts`は`"apple" | "pear" | "banana"`変換される。  
そうすると、`[ K in keyof FruitCounts ]`は`[ K in "apple" | "pear" | "banana" ]`に変換し、**マップ型**で型を繰り返し実装される。
参考Notion: <a href="https://stitch-squid-866.notion.site/Map-652db5e51b8e465ab9fc79a8596c16af" target="_blank">Map型</a>
3. `[K2 in K]: number`も同じマップ型で繰り返し型定義が作成される。
```typescript
type NewSingleFruitCount = {
    apple: {
        apple: number;
    };
    pear: {
        pear: number;
    };
    banana: {
        banana: number;
    };
}
```
なぜ新たに`[K2 in K]`マップ型を作成しているかというと、もし`K: number`で実装した場合、
```typescript
type NewSingleFruitCount = {
    apple: {
        K: number;
    };
    pear: {
        K: number;
    };
    banana: {
        K: number;
    };
}
```
になり、Kの型推論がうまく効いてくれない。

4. `[keyof FruitCounts]`は`["apple" | "pear" | "banana"]`になり、ルックアップ型でキーを指定して取得することができる  
これまでのコードを型推論したコードを記載すると
```typescript
type NewSingleFruitCount = {
    apple: {
        apple: number;
    }
    pear: {
        pear: number;
    }
    banana: {
        banana: number;
    }
}["apple" | "pear" | "banana"]
```
になり、以下のコードになる
```typescript
type NewSingleFruitCount = {
    apple: number;
} | {
    pear: number;
} | {
    banana: number;
}
```

## まとめ
TypeScriptにはユーティリティ型が充実しているため、マップ型やルックアップ型を使うことが少ないが、いつでも使える状態にしておきたい。

## 参考文献
<a href="https://twitter.com/mpocock1/status/1497262298368409605" target="_blank">TypeScript tip</a>  
<a href="https://stitch-squid-866.notion.site/Map-652db5e51b8e465ab9fc79a8596c16af" target="_blank">Map型</a>	
