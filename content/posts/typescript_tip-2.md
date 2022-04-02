---
title: "TypeScript Tip #2"
date: 2022-04-02T13:39:20+09:00
tags: 
  - "TypeScript"
---
## 背景
TypeScriptの教材で以下のサンプルコードがあったので、コードリーディングしていく。  


## 概要
以下のコードリーディングをしていく。  
非同期データ取得関数をラップして、そこで起きる例外をフォローしつつ正常系と異常系に振り分ける関数を定義している。
```typescript
type Result<T, E extends Error> = Ok<T, E> | Err<T, E>

export class Ok<T, E extends Error> {
    constructor(readonly val: T) {}
    isOk = (): this is Ok<T, E> => true
    isErr = (): this is Err<T, E> => false
}
export class Err<T, E extends Error> {
    constructor(readonly err: E) {}
    isOk = (): this is Ok<T, E> => false
    isErr = (): this is Err<T, E> => true
}
export const withResult =
    <T, A extends any[], E extends Error>(fn: (...args: A) => Promise<T>)    =>
        async (...args: A): Promise<Result<T, E>> => {
            try {
                return new Ok(await fn(...args))
            } catch (error) {
                if (error instanceof Error) {
                    return new Err(error as E)
                }
                //   return new Err(error as E)
            }
        }

type User = {
    userId: number,
    id: number,
    title: string,
    completed: boolean
}

const getUser = async(userId: string): Promise<User>  => {
    const response = await fetch(`https://jsonplaceholder.typicode.com/todos/${userId}`)
    const result = await response.json()
    return result
}


const wrapperMethod = async(): Promise<void> => {
    const data = await withResult(getUser)('1');
    console.log(data)
    // Ok: {
    // "val": {
    //     "userId": 1,
    //        "id": 1,
    //        "title": "delectus aut autem",
    //        "completed": false
    //  }
    //}
    
    if (data.isErr()) {
        console.error(data.err);
    } else {
        const user = data.val;
        console.log(`Hello, userId:${user.userId}!`);
        // "Hello, userId:1!" 
    }
}

wrapperMethod()
```

## 本題
1. classの中身を確認していく
```typescript
export class Ok<T, E extends Error> {
    constructor(readonly val: T) {}
    isOk = (): this is Ok<T, E> => true
    isErr = (): this is Err<T, E> => false
}

export class Err<T, E extends Error> {
    constructor(readonly err: E) {}
    isOk = (): this is Ok<T, E> => false
    isErr = (): this is Err<T, E> => true
}
```
1-1. `E extends Error`はジェネリクス型で型推論される型`E`に`Error`型の制約を設けている  
1-2. `isOk = (): this is Ok<T, E> => true`の`is`は`isOK`メソッドの戻り値をクラスで型指定させて、`this`の絞り込みをしている。  
<a href="https://qiita.com/ryo2132/items/ce9e13899e45dcfaff9b#is" target="_blank">TypeScript の"is"と"in"を理解する</a>  
1-3. 各クラスのメソッドの返り値はBooleanは逆になっている

2. `Result`をみていく
```typescript
type Result<T, E extends Error> = Ok<T, E> | Err<T, E>
```
2-1. 1-1と同じように`Result`のジェネリクス内でも制約を設けている    
2-2. 先ほど定義してクラスの合併型を定義している

3. `withResult`関数を見ていく
```typescript
export const withResult =
  <T, A extends any[], E extends Error>(fn: (...args: A) => Promise<T>) =>
  async (...args: A): Promise<Result<T, E>> => {
    try {
      return new Ok(await fn(...args))
    } catch (error) {
      if (error instanceof Error) {
        return new Err(error as E)
      }
    //   return new Err(error as E)
    }
  }
```
3-1. `<T, A extends any[], E extends Error>(fn: (...args: A) => Promise<T>)`で型推論させて`T` `A` `E`の型をしていく。  
`withResult(getUser)('1')`の場合、`getUser`の部分が`(fn: (...args: A) => Promise<T>)`になる。これより、`A: string[]` `T: User`で型推論される。
なので、以下のような変換される
```typescript
<User, string[] extends any[], E extends Error>(fn: (...args: string[]) => Promise<User>)
```
3-2. `async (...args: A): Promise<Result<T, E>>`この部分も上記の型推論の結果、以下になる
```typescript
async (...args: string[]): Promise<Result<User, E>>
```
3-3. `return new Ok(await fn(...args))`でfn関数(今回の場合、`getUser(1)`)実行され、その結果が`Ok`クラスのインスタンスが作成される。  
もし例外処理が発生した場合は、`Err`クラスのインスタンスが作成される。

## まとめ
正直この解説には自信がないので、ツッコミがありましたらお待ちしております🙇‍


## 参考文献
<a href="https://oukayuka.booth.pm/items/2368045" target="_blank">りあクト！ TypeScriptで始めるつらくないReact開発 第3.1版【Ⅰ. 言語・環境編】</a>  
<a href="https://qiita.com/ryo2132/items/ce9e13899e45dcfaff9b#is" target="_blank">TypeScript の"is"と"in"を理解する</a>  