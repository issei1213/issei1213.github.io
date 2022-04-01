---
title: "Typescript Tip #15"
date: 2022-04-01T23:23:48+09:00
tags: 
  - "TypeScript"
---
## 背景
Twitterを眺めていると汎用性が高そうな関数を作成できるTypeScriptのツイートがあったので、整理するためにまとめる。<br>
https://twitter.com/mpocock1/status/1509850662795989005


## 概要
関数の引数のタイプをみて、タイプに応じてpayloadが必要か必要がないかをTypeScriptで判断する。<br>
```typescript
// 第一引数がSIGN_OUTの場合は、payloadはなし
sendEvent('SIGN_OUT')

// 第一引数がLOG_INの場合は、payloadは必須
sendEvent("LOG_IN", {
    userId: '123'
})
```

成功パターン
```typescript
sendEvent('SIGN_OUT')

sendEvent("LOG_IN", {
    userId: '123'
})
```

失敗パターン(コンパイルエラー)
```typescript
sendEvent('SIGN_OUT', {})

sendEvent('LOG_IN', {
    userId: 122
})

sendEvent('LOG_IN', {})

sendEvent('LOG_IN')
```

## 本題
以下のコードで実現可能

```typescript
type AuthEvent =
    | {
    type: 'LOG_IN'
    payload: {
        userId: string
    }
}
    | {
    type: 'SIGN_OUT'
}

const sendEvent = <Type extends AuthEvent['type']>(
    ...args: Extract<AuthEvent, { type: Type }> extends { payload: infer TPayload } 
        ? [Type, TPayload]
        : [Type]
) => {
    const [type, payload] = args
    // ...
}
```
### 処理の流れ
1. ジェネリクスに制約をつける形で`Type`の型を作成する。<br>
この時点で`Type`は`LOG_IN`or`SIGN_OUT`のどちらかになる。<br>
ちなみに`AuthEvent['type']`のようにキーを指定して型定義を取得する方法を**ルックアップ型**という。
```typescript
<Type extends AuthEvent['type']>
```

2. TypeScriptは関係ないが、`...args`は引数を配列で格納する
```typescript
...args
// Ex) ['SIGN_OUT'], ['LOG_IN', { userId: '123' }]
```
3. `Extract`型で`AuthEvent`が`{ type: Type }`に代入可能なプロパティを残した型が生成できる。<br>
```typescript
Extract<AuthEvent, { type: Type }>
```
1の時点で`Type`は`LOG_IN`or`SIGN_OUT`にどちらかに決まっていたので、`{ type: Type }`は`{ type: 'LOG_IN' }` or `{ type: 'SIGN_OUT' }`のどちらかになる。<br>
なので、上記のコードで以下のどちらかになる
```typescript
{ payload: { userId: string } }

or

never
```
`Extract`型については、以下の記事を参考にするとわかりやすいかも。<br>
[【TypeScript】Utility Typesをまとめて理解する](https://qiita.com/k-penguin-sato/items/e2791d7a57e96f6144e5#extracttu)

4. 3で`Extract`の型はわかったので`{ payload: infer TPayload }`をが存在するかの条件分岐を行う。<br>
`Extract`型に`payload`が存在するかを確認して、`[Type, TPayload]`or`[Type]`の分岐を行う。<br>
また、`infer`は型は`payload`が存在していれば`TPayload`として推論してくれる。
この時点で`payload`がある引数なのか、ない引数なのかの判定ができる。
```typescript
Extract<AuthEvent, { type: Type }> extends { payload: infer TPayload }
    ? [Type, TPayload]
    : [Type]
```

`infer`については以下の記事がしっくりきた。  
[TypeScriptのinferを今度こそちゃんと理解する](https://zenn.dev/brachio_takumi/articles/464106a6a80eca8ab919#infer)

ちなみに失敗パターンは以下の型エラーになる
```typescript
sendEvent('SIGN_OUT', {})
// Expected 1 arguments, but got 2.

sendEvent('LOG_IN', {
    userId: 122
    // Type 'number' is not assignable to type 'string'
})

sendEvent('LOG_IN', {})
// Argument of type '{}' is not assignable to parameter of type '{ userId: string; }'.
// Property 'userId' is missing in type '{}' but required in type '{ userId: string; }'.

sendEvent('LOG_IN')
// Expected 2 arguments, but got 1.

```


## まとめ
うまく説明できていない気がするが、TypeScriptの復習になった。今後も続けていきたい。<br>
Thank you Matt Pocock for your informative tweet. I learned a lot.

## 参考文献
https://twitter.com/mpocock1/status/1509850662795989005 <br> 
[【TypeScript】Utility Typesをまとめて理解する](https://qiita.com/k-penguin-sato/items/e2791d7a57e96f6144e5#extracttu) <br>
[TypeScriptのinferを今度こそちゃんと理解する](https://zenn.dev/brachio_takumi/articles/464106a6a80eca8ab919#infer)

