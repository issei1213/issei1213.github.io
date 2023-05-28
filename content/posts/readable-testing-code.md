---
title: "Jestのリーダブルテストコード"
date: 2023-05-29T2:00:03+09:00
tags:
  - "週1ゆるアウトプット"
---
## 背景
今週、社内のメンバーから以下のリーダブルテストコードの記事を見せてもらった。
かなり自分の中でささったので、Jestの場合どうするかを考えてみた


<a href="https://speakerdeck.com/jnchito/number-vstat" target="_blank">リーダブルテストコード</a>  
<a href="https://logmi.jp/tech/articles/327449" target="_blank">過度なDRYを行わず、APIドキュメントだと思って書く
脳内メモリを消費させない“リーダブルなテストコード”の書き方</a>

---
## 概要
上記の参考記事を参考に、Jest(React)の場合、どのようにリーダブルテストコードを書くかを考えてみた。
今回は、UIコンポーネントとカスタムHookそれぞれのテストコードを書いていく。

---
## 本題
参考の記事にはリーダブルテストコードを書くには、以下の様に書く必要がある記載している

> - テストコードにおいて、過度なDRYは読みやすさの敵  
> - 賢くてロジカルなテストコードより、誰でも(非エンジニアでも)読める愚直なテストコードを  
> - 脳内メモリを使わないテストコードはリーダブル  
>   - 変数をなくし、上から順番に見るだけでテストの意図がわかるようにする
> - 実行可能なAPIドキュメントだと思ってテストコードを書こう(テストコードはプログラムじゃなくてドキュメント)
 
もっとリーダブルにするコツ
> - 実際のユースケースに近いテストデータを使用する
>   - 「あああ」「テストテスト」「ユーザー１」みたいなテストデータはNG
> - describe / context / itの説明を丁寧に書く
>   - 「it "適切な値を返す"」みたいな具体性のない説明はNG
> - すべての情報が1画面に収まるテストコードが理想
>   - 上下スクロールが頻繁に発生したり、他のファイルが見に行かなきゃいけないのはNG
> - DRY禁止はあくまで原則。適宜メリット・デメリットを天秤にかける。
>   - 「明確なメリットがあるDRY」や「可読性の損なわない抽象化」まで放棄するのはNG

上記を考慮して、React(Jest)のテストケースを書いていく
### UIコンポーネントテスト
#### テスト対象コンポーネント
`input`タグで入力した値`firstName`と`lastName`が、`p`タグで表示されるようになっている。
```tsx

// @ref: https://react.dev/learn/reusing-logic-with-custom-hooks#custom-hooks-let-you-share-stateful-logic-not-state-itself
import { useFormInput } from './useFormInput';

export const Form = () => {
    const firstNameProps = useFormInput('');
    const lastNameProps = useFormInput('');

    return (
        <>
            <label>
                First name:
                <input type='text' name='firstName' {...firstNameProps} />
            </label>
            <label>
                Last name:
                <input type='text' name='lastName' {...lastNameProps} />
            </label>
            <p>
                <b>Good morning, {firstNameProps.value} {lastNameProps.value}.</b>
            </p>
        </>
    );
}
```
上記のテストコードの観点としては、firstNameとlastNameの入力値が、pタグで表示されるかどうかを確認するテストコードを書く。
#### テストコード
```tsx
import {render,screen} from "@testing-library/react";
import {Form} from "./Form";
import userEvent from "@testing-library/user-event";

const user = userEvent.setup()
test('FirstNameとlastNameの入力した値が、表示されること', async () => {
    render(<Form />);
    
    await user.type(screen.getByRole('textbox', {name: 'First name:'}), 'taro')
    await user.type(screen.getByRole('textbox', {name: 'Last name:'}), 'yamada')

    expect(screen.getByText('Good morning, taro yamada.')).toBeInTheDocument();
})
```
#### テストコードのポイント
 - `input`タグに入力する内容は、変数にしていない
 - インタラクションの部分は、関数でラップしていない
 - テストコードの説明は、実際のユースケースに近いテストデータを使用している
 - すべての情報が1画面に収まるテストコードになっている

上記は比較的まだ小さいコンポーネントだが、変数を使わず、上から順番に見るだけでテストの意図がわかるようになっている。


### カスタムHookテスト
上記のコンポーネントで使用しているカスタムHook`useFormInput`のテストコードを書いていく。
`input`タグで使用する属性を作成するオブジェクトを生成するカスタムHookである。
#### テスト対象コンポーネント
```tsx
import {ChangeEventHandler, useState} from 'react';
export const useFormInput = (initialValue: string) => {
    const [value, setValue] = useState(initialValue);

    const handleChange: ChangeEventHandler<HTMLInputElement> = (e)=>  {
        setValue(e.target.value);
    }

    return {
        value: value,
        onChange: handleChange
    };
}
```


#### テストコード
```ts
import {renderHook} from "@testing-library/react";
import {useFormInput} from "./useFormInput";
import {act} from "react-dom/test-utils";
import {ChangeEvent} from "react";

test('引数の初期値がuseStateのセットされているか', () => {
    const {result} = renderHook(() => useFormInput('taro'))
    
    expect(result.current.value).toBe('taro')
})

describe('handleChange', () => {
    test('発火したときに、入力された値がstateにセットされるか', () => {
        const {result} = renderHook(() => useFormInput('taro'))

        act(() => {
            result.current.onChange({target: {value: 'jiro'}} as ChangeEvent<HTMLInputElement>)
        })

        expect(result.current.value).toBe('jiro')
    })
})

````

#### テストコードのポイント
 - 初期値の引数は実際に使用している値を使用している
 - `onChange`で取得するイベントオブジェクトは、実際に使用しているイベントオブジェクトを使用している
 - カスタムHookで定義されている関数毎で、`describe`を使用している

---
## まとめ
**テストケースをドキュメント代わりで使用する**視点を持つことで、テストのリーダブル性が上がることがわかった。  
意識次第ですぐに実践できるので、ぜひ取り入れていく

---
## 参考文献
<a href="https://speakerdeck.com/jnchito/number-vstat" target="_blank">リーダブルテストコード</a>  
<a href="https://logmi.jp/tech/articles/327449" target="_blank">過度なDRYを行わず、APIドキュメントだと思って書く
脳内メモリを消費させない“リーダブルなテストコード”の書き方</a>
