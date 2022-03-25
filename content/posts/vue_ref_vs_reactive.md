---
title: "refとreactive"
date: 2022-03-25T12:46:58+09:00
draft: true
tags: 
- "Vue.js"
---
## 背景
`ref`と`reactive`の使い分けがパッとでてこなかったことがあったのでまとめていく。

## 概要
1. どの状況で使い分ければいいのか
2. `ref`と`reactive`の違い

## 本題
### 1.どの状況で使い分ければいいのか
`ref`はプリミティブ型に対して使用する。<br/>
`reactive`はオブジェクト型に対して使用する。

### 2.`ref`と`reactive`の違い
#### `ref`
```typescript
const boolean = ref(false)
// export function ref<boolean>(value: boolean): Ref<boolean>
```
- TypeScriptの場合は、引数(初期値)を型推論してくれる。
- 基本的になんでも(オブジェクト、配列、プリミティブ)`ref`で使用ができる。
- 引数の値がオブジェクトの場合は、Vue内部で`reactive`に更新している。<br/>**更新している負荷を与えてしまうため、オブジェクトの場合は、`reactive`を使用したほうがいいと言われている。**
- ref.valueでデータを格納する Ex)`boolean.value = true`


#### `reactive`
```typescript
const user = reactive({
    firstName: '太郎',
    lastName: '山田',
    height: 160,
    weight: 50
})
// const user: {firstName: string, lastName: string, height: number, weight: number}
```
- TypeScriptの場合は、引数(初期値)を型推論してくれる。
- オブジェクトのみ使用ができる。
- データの格納は通常のオブジェクトの様に操作する。 ex) `user.firstName = '二郎''`
- 分割代入やスプレッド構文を使用するとリアクティブ性が失われる。

## まとめ
基本的に以下の参考文献を参考したが、公式のベストプラクティスを提示していないため、今後は方針がかわる可能性もある。
その時には、再度更新していく

## 参考文献
[Vue Composition APIのrefとreactiveを解説！違いと使い分け](https://kobatech-blog.com/vue-composition-api-ref-reactive/)

[ref vs reactive in Vue 3?](https://stackoverflow.com/questions/61452458/ref-vs-reactive-in-vue-3)