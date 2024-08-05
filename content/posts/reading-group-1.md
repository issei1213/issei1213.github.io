---
title: "社内輪読会"
date: 2024-06-30T01:36:29+09:00
tags: 
  - "Tech"
---
## 背景
最近の社内で輪読会でリーダブルコードを読む機会があった。  
その際に、自分が改めて気づきがあったので、それをまとめる。

---
## 概要
輪読会では第2章と第3章を読んだため、それぞれの内容をまとめる。

---
## 本題
### 第2章: 名前に情報を詰め込む

---

#### スコープが小さければ短い名前でもいい  
スコープの小さい関数で使われる変数などで、見える範囲が狭い場合は、短い名前でも問題ない。
全ての情報がみえるのので、短くていい

```ts
if(debug) {
    const m = Map()
    lookUpNAmesNumbers(m)
    
    console.log(m)
}
```
上記の例では、`m`という変数名は短いが、スコープが小さいため問題ない。  
例えば、`m`がグローバル変数だとすると、`m`という変数名だけでは何をしているのかわからない。その場合は、別の命名を検討する。

---

#### 頭文字と省略文字  
エンジニアは文字を省略して書く癖がある。例えば、`BackEndManager`を`BEManager`と省略することがある。  
しかし、これは避けるべきである。新しい人がコードを読む際に、何を指しているのかわからないため。理解できるなら問題ない。

問題ない例として、`document`を`doc`、`string`を`str`と省略することは問題ない。
ここの線引きは難しいが、理解できる範囲で省略することが大事。

### 第3章: 誤解されない名前

---

#### 範囲を指定するときはfirstとlastを使う  
範囲を指定するときは、`first`と`last`を使うことで、範囲を指定することができる。   
例えば、以下の様なコードがあったする。  
```ts
function intgerRange(start = 2, stop = 4){
   ...
}
```
上記の関数は、startの2から始まることはわかるが、stopの4を含むのか含まないのかわからない。
そのため、`first`と`last`を使うことで、範囲を指定することができる。

```ts
function intgerRange(first = 2, last = 4){
   ...
}
```
上記の関数は、startの2から始まり、stopの4を含むことがわかる。  

---

#### ブール値の名前  
ブール値の命名を否定系にすることは避けるべきである。

例えば以下のコードは、否定系の命名になっている。
```ts
const disableSsl = false
```
上記のコードは、`disableSsl`が`false`の場合、SSLが有効になることがわかるが、`true`の場合はSSLが無効になることがわかる。
これでは、コードの可読性が下がるため、肯定系の命名をすることが望ましい。

```ts
const enableSsl = true
```
上記のコードは、`enableSsl`が`true`の場合、SSLが有効になることがわかり、可読性が上がる。  

---
## まとめ
数年前に読んだリーダブルコードを改めて読んで、命名の大切さを振り返ることができた。  
輪読会ではメンバー全員で同じ章を読んだ後、それぞれの意見を共有する方針を取っている。これが意外と盛り上がって楽しいので、今後も継続していきたい

---
## 参考文献
<a href="https://www.oreilly.co.jp/books/9784873115658/" target="_blank">リーダブルコード</a>
