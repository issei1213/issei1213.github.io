---
title: "Component TypesとEvent Handlers"
date: 2022-08-12T21:08:23+09:00
tags: 
  - "SolidJS"
  - "TypeScript"
---
## 背景
SolidJSのコンポーネントとイベントハンドラの型定義について悩んでいたが、公式で解説が乗っていたので、  
個人的にまとめることにする。

---
## 概要
コンポーネントとイベントハンドラの型定義の方法をまとめていく。  
公式ではコンポーネントの型を**Component Types**、エベントハンドラの型を**Event Handlers**として記載しているので、
同一の単語を使用していく。

---
## 本題
### Component Types
#### `Component`
```typescript jsx
import type { JSX, Component } from 'solid-js';
type Component<P = {}> = (props: P) => JSX.Element;
```
基本的にはReactのFCと同じような使い型ができ、propsの型をジェネリクスで定義することができる。  
戻り値は`JSX.Element`になる。

サンプルコード　※公式のコードを参照
```typescript jsx
import { render } from "solid-js/web";
import { createSignal, Component } from "solid-js";

const Counter: Component = () => {
    const [count, setCount] = createSignal(0);
    return (
        <button onClick={() => setCount((c) => c+1)}>
            {count()}
        </button>
    );
};

const InitCounter: Component<{initial: number}> = (props) => {
    const [count, setCount] = createSignal(props.initial);
    return (
        <button onClick={() => setCount((c) => c+1)}>
            {count()}
        </button>
    );
};

function App() {
    return (
        <>
            <InitCounter initial={5}/>  // good
            <Counter/>;                 // good
            <Counter initial={5}/>;     // type error: no initial prop
            <Counter>hi</Counter>       // type error: no children prop
        </>

    );
}

render(() => <App />, document.getElementById("app")!);
```

#### `ParentComponent`
コンポーネントに`children`(子要素)を追加したい場合は、Propsに明示的に追加するか、  
自動的に`children`を追加される`ParentComponent`を使用する。
`ParentProps`に`children`にオプショナルがついているため、省略することもできる。
また、ジェネリクスを使用してpropsの型定義も使用できる。
```typescript jsx
import { JSX, ParentComponent, ParentProps } from 'solid-js';
type ParentProps<P = {}> = P & { children?: JSX.Element };
type ParentComponent<P = {}> = Component<ParentProps<P>>;
```
サンプルコード
```typescript jsx
import { render } from "solid-js/web";
import { createSignal, ParentComponent } from "solid-js";

const CustomInitCounter: ParentComponent<{initial: number}> = (props) => {
    const [count, setCount] = createSignal(props.initial);
    return (
        <button onClick={() => setCount((c) => c+1)}>
            {count()}
            {props.children}
        </button>
    );
};

function App() {
    return (
        <>
            <CustomInitCounter initial={5}>
                <p>childrenだよ</p>
            </CustomInitCounter>
        </>

    );
}

render(() => <App />, document.getElementById("app")!);

```

#### `VoidComponent`
`chidren`を使用しない時に使用する型。  
`Component`の`children`を使用しない型と同等の意味。
```typescript jsx
import {JSX, VoidComponent, VoidProps} from 'solid-js';
type VoidProps<P = {}> = P & { children?: never };
type VoidComponent<P = {}> = Component<VoidProps<P>>;
```

サンプルコード
```typescript jsx
import { render } from "solid-js/web";
import { createSignal, VoidComponent } from "solid-js";

const CustomInitCounter: VoidComponent<{initial: number}> = (props) => {
  const [count, setCount] = createSignal(props.initial);
  return (
    <button onClick={() => setCount((c) => c+1)}>
      {count()}
    </button>
  );
};

function App() {
  return (
    <>
        <CustomInitCounter initial={5} />
    </>

  );
}

render(() => <App />, document.getElementById("app")!);

```
#### `FlowComponent`
```typescript jsx
import {JSX, FlowComponent, FlowProps} from 'solid-js';
type FlowProps<P = {}, C = JSX.Element> = P & { children: C };
type FlowComponent<P = {}, C = JSX.Element> = Component<FlowProps<P, C>>;
```
`<Show>` `<For>`の様な制御フローを使用するコンポーネントを使用している。  
`children`は必須となり、`children`が関数である場合、関数の型など定義することができる。

サンプルコード
```typescript jsx
import { render } from "solid-js/web";
import { FlowComponent, createEffect } from "solid-js";

const CallMeMaybe: FlowComponent<{when: boolean}, () => void> = (props) => {
  createEffect(() => {
    if (props.when)
      props.children();
  });
  return <>{props.when ? 'Calling' : 'Not Calling'}</>;
};


function App() {
  return (
    <>
      // <CallMeMaybe when={true}/>;  // type error: missing children
      // <CallMeMaybe when={true}>hi</CallMeMaybe>;  // type error: children
      <CallMeMaybe when={true}>                   // good
        {() => console.log("Here's my number")}
      </CallMeMaybe>;  
    </>
  );
}

render(() => <App />, document.getElementById("app")!);

```

---

### Event Handlers
イベントハンドラの型を定義できる。`JSX.EventHandler<T, E>`で定義し、`T`はDOMの要素型で`E`イベント型として使用する。

サンプルコード
```typescript jsx
import type { JSX } from 'solid-js';
const onInput: JSX.EventHandler<HTMLInputElement, InputEvent> = (event) => {
  console.log('input changed to', event.currentTarget.value);
};

<input onInput={onInput}/>
```

また、関数に切り出さないで直接要素に記載した場合は、適切に型を判断してくれる。
```typescript jsx
<input onInput={(event) => {
    console.log('input changed to', event.currentTarget.value);
    }}
/>;
```
エディターによる型推論

![](/img/posts/solidjs_types_event-handler.png)

## まとめ
コンポーネントの型の種類が、Reactよりは多くなっているが、リアクティブなコンポーネントを作るために、必要なものなのかなと感じた。  
適切に使用しつつ、コンポーネントの見通しもよくしていきたい。

---
## 参考文献
<a href="https://www.solidjs.com/guides/typescript#component-types" target="_blank">Component Types</a>  
<a href="https://www.solidjs.com/guides/typescript#event-handlers" target="_blank">Event Handlers</a>	
