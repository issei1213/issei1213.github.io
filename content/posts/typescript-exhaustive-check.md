---
title: "Typescript Exhaustive Check"
date: 2024-05-31T00:27:26+09:00
tags: 
  - "TypeScript"
---
## 背景
ソフトウェアデザインを読んでいて網羅性チェックについて知ったので、TypeScriptでの網羅性チェックについて自分なりにまとめてみた。

---
## 概要
網羅性チェックとは、全てのケースを網羅しているどうかをチェックしていることを指す。
TypeScriptは様々なケースで利用できるが、今回は文字列リテラル型を利用して網羅性チェックを行う方法をまとめていく。

---
## 本題
以下のようなコードがあるとする。
`Fruit`型には`apple`、`banana`、`orange`の3つの文字列リテラル型があり、`checkAllFruitHandler`関数は`Fruit`型を引数に取り、その値によって処理を分岐している。
ここではシンプルに`console.log`で出力している。

```ts
type Fruit = 'apple' | 'banana' | 'orange';

function checkAllFruitHandler(fruit: Fruit): void {
    if(fruit === 'apple') {
        console.log('This is an apple.');
        return
    } else if (fruit === 'banana') {
        console.log('This is a banana.');
        return
    } else  {
        console.log('This is an orange.');
        return
    }
}

checkAllFruitHandler('apple') // This is an apple.
checkAllFruitHandler('banana') // This is an banana.
checkAllFruitHandler('orange') // This is an orange.

```

しかし、`Fruit`型に `grape` が追加された場合、`checkAllFruitHandler`関数は`grape`に対する条件分岐がないため、
`checkAllFruitHandler('grape')`を実行すると、`This is an orange.`が出力されてしまう。
この時にコンパイルエラーも発生しないため、不具合の原因が分かりにくい。
```ts
type Fruit = 'apple' | 'banana' | 'orange' | 'grape';

function checkAllFruitHandler(fruit: Fruit): void {
    if(fruit === 'apple') {
        console.log('This is an apple.');
        return 
    } else if (fruit === 'banana') {
        console.log('This is a banana.');
        return
    } else  {
        console.log('This is an orange.');
        return
    }
}

checkAllFruitHandler('grape') // This is an orange.

```

このような問題を解決するために、全てのケースを網羅しているかをチェックする必要がある。
大まかに分けて二つパターンがあるため、紹介していく。

1. `switch`文を利用する
上記では `if`文を利用していたが、`switch`文を利用して全てのケースをチェックしていく。
```ts
type Fruit = 'apple' | 'banana' | 'orange';

function checkAllFruits(fruit: Fruit): void {
    switch (fruit) {
        case 'apple':
            console.log('This is an apple.');
            break;
        case 'banana':
            console.log('This is a banana.');
            break;
        case 'orange':
            console.log('This is an orange.');
            break;
        default:
            const _: never = fruit;
    }
}
```
ポイントは、`default`節の部分に`never`型を指定してい点。
`default`節には`Fruit`型の全てのケースを網羅しているため、`never`型を指定することで、`Fruit`型の全てのケースを網羅していることを示している。

もし、`Fruit`型に新しい文字列リテラル型が追加された場合、コンパイルエラーが発生するため、網羅性チェックができる。
```ts
type Fruit = 'apple' | 'banana' | 'orange' | "grape";

function checkAllFruits(fruit: Fruit): void {
    switch (fruit) {
        case 'apple':
            console.log('This is an apple.');
            break;
        case 'banana':
            console.log('This is a banana.');
            break;
        case 'orange':
            console.log('This is an orange.');
            break;
        default:
            const _: never = fruit;
        // Error    
        // '_' is declared but its value is never read.(6133)
        // Type 'string' is not assignable to type 'never'.(2322)
    }
}
```

また、TypeScript4.9で追加になった`satisfies` 演算子を使って`default`節を記載することもできる。
型推論を利用して、`default`節には`never`型を指定することで、網羅性チェックをしている。
```ts
type Fruit = 'apple' | 'banana' | 'orange' | 'grape';

function checkAllFruits(fruit: Fruit): void {
    switch (fruit) {
        case 'apple':
            console.log('This is an apple.');
            break;
        case 'banana':
            console.log('This is a banana.');
            break;
        case 'orange':
            console.log('This is an orange.');
            break;
        default:
            fruit satisfies never
            // Error
            // Type 'string' does not satisfy the expected type 'never'.(1360)
    }
}
```

このようのswitch式を利用することで、文字列リテラル型に追加してもTSのコンパイルエラーで検知できるようになり、
保守性の高いコードになる。



2. オブジェクトで管理する
文字列リテラル型の文字列をオブジェクトのフィールドに指定して、バリューに対応する処理をかいていく。
こうすることで、文字列リテラル型に追加してもコンパイルエラーが発生するため、網羅性チェックができる。
```ts
type Fruit = 'apple' | 'banana' | 'orange';

const checkAllFruitHandler: Record<Fruit, () => void> = {
    apple: () => console.log('This is an apple.'),
    banana: () => console.log('This is a banana.'),
    orange: () => console.log('This is an orange.'),
};


checkAllFruitHandler['apple']()     // This is an apple.
checkAllFruitHandler['banana']()    // This is a banana.
checkAllFruitHandler['orange']()    // This is an orange.
```

例えば、`grape`を追加した場合、以下のようにエラーが発生する。
```ts
type Fruit = 'apple' | 'banana' | 'orange' | 'grape';


// Error
// Property 'grape' is missing in type '{ apple: () => void; banana: () => void; orange: () => void; }' but required in type 'Record<Fruit, () => void>'.(2741)
const checkAllFruitHandler: Record<Fruit, () => void> = {
    apple: () => console.log('This is an apple.'),
    banana: () => console.log('This is a banana.'),
    orange: () => console.log('This is an orange.'),
};

```

---
## まとめ
今回はTypeScriptで網羅性チェックを行う方法についてまとめてみた。他の言語では、標準で網羅性チェックができるものもあるらしい。
他の言語を学ぶことで、TypeScriptの理解度を上がることもあると思うので、ぜひ挑戦していきたい。

