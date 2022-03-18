---
title: "watchとwatchEffect"
date: 2022-03-18T10:36:36+09:00
tags:
  - "Vue.js"
---
## 背景
Vue.jsを使用する際にwatchを使う頻度はそこまで多くなく、使う度にドキュメントを確認するので、記事にまとめておく。

## 概要
Vue3の`watch`と`watchEffect`の仕様をまとめていく。また、それぞれの違いもまとめる。

## 本題
### watch
#### 説明
指定したデータを監視して、変化があればコールバック関数が実行する。

#### 使い方
単一のデータ監視と複数のデータ監視の2種類の使い方が存在する。
##### 1. 単一のデータを監視する場合

```typescript
const count = ref(0)
// 第一引数に監視したいデータを記載する
watch(count, (newCount, prevCount) => {
  /* ... */
})
```
```typescript
const state = reactive({ 
    count: 0 
})
// 第一引数に監視したいデータをコールバック関数で記載する
watch(() => state.count, (newCount, prevCount) => {
    /* ... */
  }
)
```
---
##### 2. 複数のデータを監視する場合
- 配列を使用することで、複数のデータを監視できる
- 複数の監視データが同時に変更する時には、コールバック関数は一度して動かないため注意
```typescript
const firstName = ref('')
const lastName = ref('')

// 第一引数を配列にしている
watch([firstName, lastName], (newValues, prevValues) => {
  console.log(newValues, prevValues)
})

firstName.value = 'John' // logs: ["John", ""] ["", ""]
lastName.value = 'Smith' // logs: ["John", "Smith"] ["John", ""]
```

複数のデータを監視し、両方のデータの検知をしたいときには、`nextTick`などを使用する
```typescript
const changeValues = async () => {
  firstName.value = 'John' // logs: ["John", ""] ["", ""]
  await nextTick()
  lastName.value = 'Smith' // logs: ["John", "Smith"] ["John", ""]
}
```
アクティブな配列やオブジェクトの値を検知するには、値のコピーを作成する必要がある。また、コールバック関数で記載する。
```typescript
const numbers = reactive([1, 2, 3, 4])

//　スプレッド構文で展開する必要がある
watch(() => [...numbers], (numbers, prevNumbers) => {
    console.log(numbers, prevNumbers)
  }
)
numbers.push(5) // logs: [1,2,3,4,5] [1,2,3,4]
```
深くネストしたときには、`deep`を利用する
```typescript
watch(() => state, (state, prevState) => {
    console.log('deep', state.attributes.name, prevState.attributes.name)
  },
  { deep: true }
)
```

しかし、`deep`を使った時には、現在値と前回値の取得ができなくなることがあるため、完全に監視するときにはライブラリを利用する
```typescript
import _ from 'lodash'

const state = reactive({
  id: 1,
  attributes: {
    name: ''
  }
})

watch(() => _.cloneDeep(state), (state, prevState) => {
    console.log(state.attributes.name, prevState.attributes.name)
  }
)
state.attributes.name = 'Alex'; // Logs: "Alex" ""
```
### watchEffect
#### 説明
- `watchEffect`関数内のリアクティブオブジェクト(`ref`, `reactive`)を監視し、変更されるたびに即座に関数を再実行する。
- 変更前と変更後の値は取得できない
#### 使い方
```typescript
const count = ref(0)

// count(ref)が存在するため、変更した時にはコールバック関数が実行される
watchEffect(() => console.log(count.value))
// -> logs 0

setTimeout(() => {
  count.value++
  // -> logs 1
}, 100)
```

### watchとwatchEffectの違い
|    |  `watch`  |  `watchEffect`  |
| ---- | ---- | ---- |
|  定義の仕方  |  監視対象のデータを第一引数に指定  |  関数内のリアクティブオブジェクト(`ref`, `reactive`)が監視対象  |
|  データの取得  |  変更前と変更後のデータが取得できる  |  取得できない  |
|  初回実行のタイミング  |  定義に実行されない  |  定義時に実行する  |

## まとめ
どちらを使用するかは意見が別れるとこだとは思うが、個人的には監視する対象を明示的にしておきたいので、`watch`の方が好みだと感じた。

## 参考文献
[公式ドキュメント](https://v3.ja.vuejs.org/guide/reactivity-computed-watchers.html#%E7%AE%97%E5%87%BA%E3%83%95%E3%82%9A%E3%83%AD%E3%83%8F%E3%82%9A%E3%83%86%E3%82%A3%E3%81%A8%E3%82%A6%E3%82%A9%E3%83%83%E3%83%81) <br>
[Qita: Vue 3 の Composition API における watch vs watchEffect](https://qiita.com/doz13189/items/d09cfc6e1ff38621c2cc#watch-%E3%81%AE%E5%A4%89%E6%9B%B4%E5%89%8D%E3%81%A8%E5%A4%89%E6%9B%B4%E5%BE%8C%E3%81%AE%E5%80%A4%E3%81%8C%E5%8F%96%E5%BE%97%E3%81%A7%E3%81%8D%E3%82%8B)
