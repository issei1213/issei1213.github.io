---
title: "TypeScript Tip #3"
date: 2022-04-09T01:13:25+09:00
tags: 
  - "TypeScript"
---
## 背景
今回もmattoさんのコードリーディングをしていく。  
<a href="https://twitter.com/mpocock1/status/1499002040168636420" target="_blank">TypeScript Tip #3</a>


---
## 概要
今回はTypeScriptのユーティリティをまとめたライブラリ`ts-toolbelt`について説明していく。  
注意点としては、**TypeScriptバージョンが4.1以上である必要がある**。
数多くのユーティリティが提供されているため、今回はMattさんの解説に絞ってまとめていく。  
<a href="https://millsp.github.io/ts-toolbelt/index.html" target="_blank">ts-toolbelt</a>

---
## 本題
### 使い方
ライブラリの中から使用したいユーティリティをimportして使用する。
```typescript
import { String, Union } from 'ts-toolbelt';
```
今回はURLの操作をする時に便利なユーティリティをまとめていく。

### サンプルコード
```typescript
import { String, Union } from 'ts-toolbelt';

const query = '/home?a=foo&b=wow'

type Query = typeof query
//  type Query = "/home?a=foo&b=wow"

type SecondQueryPart = String.Split<Query, '?'>[1]
//  type SecondQueryPart = "a=foo&b=wow"

type QueryElements = String.Split<SecondQueryPart, '&'>
//  type QueryElements = ["a=foo", "b=wow"]

type QueryParams = {
    [ QueryElement in QueryElements[number] ]: {
        [ key in String.Split<QueryElement, '='>[0]]: String.Split<QueryElement, '='>[1]
    }
}[QueryElements[number]]
//  type QueryParams = {
//    a: "foo";
//  } | {
//    b: "wow";
//  }

const obj: Union.Merge<QueryParams> = {
    a: 'foo',
    b: 'wow'
}
//  const obj: {
//    a: "foo";
//    b: "wow";
//  }
```

1. `type SecondQueryPart = String.Split<Query, "?">;`でリテラル型を`?`で区切りタプル型を作成する。
よって、`type SecondQueryPart = ["/home", "a=foo&b=wow"]`になる。その後、ルックアップ型で要素を取得している。  
`type SecondQueryPart = String.Split<Query, '?'>[1]`で`type SecondQueryPart = "a=foo&b=wow"`の型を取得することができる。  
<a href="https://numb86-tech.hatenablog.com/entry/2020/06/28/101757" target="_blank">TypeScript のルックアップ型と keyof キーワード</a>

2. `type QueryElements = String.Split<SecondQueryPart, "&">;`は上記と同じように`&`で区切りタプル型を作成している。  
そのため、`type QueryElements = ["a=foo", "b=wow"]`になる。

3. `QueryParams`型のインデックスシグネチャをみていく。  
`QueryElement in QueryElements[number]`の`QueryElements[number]`は、以下の様になる。
```typescript
QueryElements[number]
🔽
["a=foo", "b=wow"][number]
🔽
"a=foo" | "b=wow"
```
よって、`QueryElement in QueryElements[number]`は`QueryElement in "a=foo" | "b=wow"`でマップ型が作成される。
4. `Key in String.Split<QueryElement, "=">[0]`の`String.Split<QueryElement, "=">[0]`は以下になる。  
`a=foo`の場合
```typescript
String.Split<'a=foo', "=">[0]
🔽
"a"
```
上記と同じ様に`String.Split<QueryElement,"=">[1]`も時も以下の様になる。
```typescript
String.Split<QueryElement,"=">[1]

🔽

"foo"
```
5. ここまでを見ると最後のルックアップ型を考慮せずに記載すると以下になる
```typescript
type QueryParams = {
    [QueryElement in QueryElements[number]]: {
        [Key in String.Split<QueryElement, "=">[0]]: String.Split<QueryElement,"=">[1];
    };
}

🔽

type QueryParams = {
    "a=foo": {
        a: "foo";
    };
    "b=wow": {
        b: "wow";
    };
}
```
6. 上記のコードに最後のルックアップ型を追加すると、指定したキーのバリューを取得することができる。
```typescript
type QueryParams = {
  [QueryElement in QueryElements[number]]: {
    [Key in String.Split<QueryElement, "=">[0]]: String.Split<QueryElement,"=">[1];
  };
}[QueryElements[number]];

🔽

type QueryParams = {
    "a=foo": {
        a: "foo";
    };
    "b=wow": {
        b: "wow";
    };
}["a=foo" | "b=wow"];

🔽

type QueryParams = {
    a: "foo";
} | {
    b: "wow";
}
```
7. 最後に`Union.Merge<T>`でユニオン型の型をインターセクション型に定義することができる。
```typescript
Union.Merge<QueryParams>
    
🔽

Union.Merge<
        {
            a: "foo";
        } | {
            b: "wow";
        }>
    
🔽

 {
     a: "foo";
     b: "wow";
 }
```

---
## まとめ
ts-toolbeltの存在を初めて知ったので、非常に勉強になった。比較的バージョンを新しくする必要があるので、
使用できる環境があれば、積極的に公式のドキュメントを参考にしていきたい。

---
## 参考文献
<a href="https://twitter.com/mpocock1/status/1499002040168636420" target="_blank">TypeScript Tip #3</a>  
<a href="https://millsp.github.io/ts-toolbelt/index.html" target="_blank">ts-toolbelt</a>  
<a href="https://numb86-tech.hatenablog.com/entry/2020/06/28/101757" target="_blank">TypeScript のルックアップ型と keyof キーワード</a>


