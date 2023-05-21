---
title: "Testing Libraryで暗黙的なロールを使用したDOMの取得方法"
date: 2023-05-21T23:11:10+09:00
tags: 
  - "週1ゆるアウトプット"
---
## 背景
現在以下の書籍でフロントのテストを学習しているので、勉強になったDOMの取得方法をまとめる。
書籍通り、ReactのUIコンポーネントのテスト方法を記載していく。

---
## 概要

ReactのUIコンポーネントのDOMを取得する方法で、暗黙的なロールを活用した取得方法があったので、まとめていく

---
## 本題
以下のコンポーネントのテストをしていく。
```tsx
import {ReactNode} from "react";

export const Heading = ({children}: {children: ReactNode}) => {
    return (
        <h1>{children}</h1>
    );
};
```

今回は簡易的に、`props`の値が`h1`タグで表示されているかをテストしていく。
```tsx
import {render, screen} from "@testing-library/react";
import {Heading} from "./Heading";

test('propsが値がh1タグに存在していること', () => {
    render(<Heading>test</Heading>)

    expect(screen.getByRole('heading', {name: 'test'})).toBeInTheDocument()
})
```

上記の`getByRole`の第一引数に`heading`を指定しているが、h1タグの暗黙的なロールが`heading`であるため、`heading`を指定している。  
このように暗黙的なロールをうまく使えば、`test-id`みたいなカスタム属性をなどを使用せずに、テストをかくことができる。
また、<a href="https://testing-library.com/docs/queries/about/#priority" target="_blank">Testing Library</a>も公式も暗黙的なロールを使用することを推奨している。

暗黙的なルール一覧は<a href="https://www.npmjs.com/package/aria-query" target="_blank">こちら</a>から確認ができる。  
※上記は`aria-query`というライブラリのドキュメントであるが、Testing Libraryのライブラリは内部的に依存するので、上記を参考にしても問題ない


---
## まとめ
UIコンポーネントのテストをする際は、暗黙的なロールを使用することで、テストを書きやすくなるので、ぜひ活用していきたい。

---
## 参考文献
<a href="https://testing-library.com/docs/queries/about/#priority" target="_blank">Testing Library</a>  
<a href="https://www.npmjs.com/package/aria-query" target="_blank">暗黙的なルール一覧(aria-query)</a>  
<a href="https://www.shoeisha.co.jp/book/detail/9784798178639" target="_blank">フロントエンド開発のためのテスト入門 今からでも知っておきたい自動テスト戦略の必須知識</a>	
