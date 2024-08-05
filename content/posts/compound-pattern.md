---
title: "Compound Patternについて"
date: 2024-07-31T01:18:06+09:00
tags: 
  - "Tech"
---
## 背景

宮崎のReact勉強会で小規模なLTをしたので、発表した内容をまとめる。  
LTではCompound Pattern（複合パターン）について発表を行った。
---
## 概要
- Compound Pattern（複合パターン）とは、コンポーネント設計パターンの一つ
- 複数のコンポーネントを組み合わせて、複雑なコンポーネントを構築する方法

---
## ユースケース
1. お知らせ機能を`Modal`コンポーネントを使って実装している

![](/img/posts/compound-pattern/usecase-1.png)  
実際に使用しているコンポーネントは以下  
``modal.tsx``
```tsx
const Modal: React.FC<ModalProps> = ({onClose, children }) => {
    return (
        <div className="..." onClick={onClose}>
            <dialog
                open
                className="..."
                onClick={(e) => e.stopPropagation()}
            >
                {/* NOTE: モーダル閉じるボタン */}
                <button
                    className="..."
                    onClick={onClose}
                >
                    <Icon variant="close" className="..." />
                </button>
                {/* NOTE: ヘッダー・テキスト・フッターなどはchildrenで表示 */}
                {children}
            </dialog>
        </div>
    );
};
```

`modal.tsx` を使った実装コード
```tsx
const ModalPage = () => {
    return (
        <Modal>
            <Heading as="h2">システムメンテナンスのお知らせ</Heading>
            <div className="...">
                <p>
                    平素より弊社サービスをご利用いただき、誠にありがとうございます。
                    <br />
                    下記の日程でシステムメンテナンスを実施いたします。メンテナンス期間中はサービスのご利用ができなくなりますので、ご注意ください。
                    <br />
                    <br />
                    【メンテナンス日時】
                    <br />
                    2024年8月1日（木） 午前2:00 ～ 午前5:00
                    <br />
                    <br />
                    ご不便をおかけいたしますが、ご理解とご協力のほど、よろしくお願いいたします。
                </p>
            </div>

            <div className="...">
                <Button variant="primary" onClick={closeModal}>
                    キャンセル
                </Button>
            </div>
        </Modal>
    )
}

```

2. 次に削除確認モーダルを`Modal`コンポーネントを使って実装していく。  
しかし、削除確認モーダルには閉じるアイコンが不要なモーダルになっている。
![](/img/posts/compound-pattern/usecase-2.png)

3. `Modal` コンポーネント内部に閉じるボタンが実装している。  
そのため、`props`(`isVisibleCloseButton`)を使って、閉じるボタンの表示・非表示を制御する。
```tsx
type ModalProps = {
    children: ReactNode;
    onClose?: () => void;
    isVisibleCloseButton?: boolean; // NOTE: オプショナルなpropsを定義
};

const Modal: React.FC<ModalProps> = ({
    onClose, 
    children, 
    isVisibleCloseButton = true // NOTE: デフォルト値を設定して、指定がない場合は基本表示する
}) => {
    return (
        <div className="..." onClick={onClose}>
            <dialog
                open
                className="..."
                onClick={(e) => e.stopPropagation()}
            >
                {/* NOTE: isVisibleCloseButtonがtrue場合、閉じるボタンを表示する */}
                {isVisibleCloseButton && (
                    <button
                        className="..."
                        onClick={onClose}
                    >
                        <Icon variant="close" className="..." />
                    </button>
                )}

                {children}
            </dialog>
        </div>
    );
};
```

変更後の`modal.tsx` を使った実装コード
```tsx
const ModalPage = () => {
    return (
        <Modal isVisibleCloseButton={false}> // NOTE: isVisibleCloseButtonをfalseに設定
            <Heading as="h2">データ削除の確認</Heading>
            <div className="...">
                <p>
                    このデータを削除してもよろしいですか？
                    <br />
                    この操作は取り消せません。
                </p>
            </div>

            <div className="...">
                <Button variant="primary" onClick={closeModal}>
                    キャンセル
                </Button>
                <Button
                    variant="delete"
                    onClick={() => console.log('called delete API')}
                >
                    削除
                </Button>
            </div>
        </Modal>
    )
}

```

4. propsを追加して閉じるアイコンを管理する方法には、以下の問題がある  
   - 表示関連のpropsが増えていく可能性がある   
        Ex: Modalコンポーネントの中でフッターの内容まで含めていると、isVisibleFooterみたいなpropsが作成される可能性がある
   - オプショナルなpropsが発生するので、デフォルト値を考慮しないといけない 
   - オプショナルなpropsを作成しない場合、修正箇所が多くなる。

5. 上記の問題を解決するために、Compound Patternを使って実装する  
Compound Patternを使った `Modal` コンポーネント  
```tsx
type ModalProps = {
    children: ReactNode;
    onClickBackGround?: () => void;

};

// NOTE: Modal全体をラップするコンポーネント
const Modal = ({ onClickBackGround, children }: ModalProps) => {
    return (
        <div className="..." onClick={onClickBackGround}>
            <dialog open className="..." onClick={(e) => e.stopPropagation()}>
                {children}
            </dialog>
        </div>
    );
};

type ModalCloseButtonProps = {
    onClick: () => void;
};

// NOTE: 閉じるアイコンコンポーネント
const ModalCloseButton = ({ onClick }: ModalCloseButtonProps) => {
    return (
        <button className="..." onClick={onClick}>
            <Icon variant="close" className="..." />
        </button>
    );
};

type ModalHeaderProps = PropsWithChildren;

// NOTE: ヘッダーテキストコンポーネント
const ModalHeader = ({ children }: ModalHeaderProps) => {
    return (
        <div className="...">{children}</div>
    );
};

type ModalBodyProps = PropsWithChildren;

// NOTE: ボディテキストコンポーネント
const ModalBody = ({ children }: ModalBodyProps) => {
    return (
        <div className='...'>{children}</div>
    );
};

type ModalFooterProps = PropsWithChildren<{
    actions?: ReactElement;
    onClickCancel: () => void;
}>;

// NOTE: フッターテキストコンポーネント
const ModalFooter = ({ onClickCancel, actions }: ModalFooterProps) => {
    return (
        <div className="...">
            <button data-ripple-dark="true" data-dialog-close="true" className="..." onClick={onClickCancel}>
                キャンセル
            </button>
            {actions}
        </div>
    );
};

// NOTE: Modalコンポーネントをオブジェクトとして扱い、各コンポーネントをプロパティとして持つ
Modal.CloseButton = ModalCloseButton;
Modal.Header = ModalHeader;
Modal.Body = ModalBody;
Modal.Footer = ModalFooter;

export { Modal };
```

Compound Patternを使った `Modal` コンポーネントを使った実装コード
`Modal.CloseButton` コンポーネントを使って閉じるアイコンを表示していたため、記載しないだけで閉じるアイコンが非表示になる。
```tsx
const ModalPage = () => {
    return (
        <Modal onClickBackGround={closeModal}>
            <Modal.Header>システムメンテナンスのお知らせ</Modal.Header>
            <Modal.Body>
                <p>
                    平素より弊社サービスをご利用いただき、誠にありがとうございます。
                    <br />
                    下記の日程でシステムメンテナンスを実施いたします。メンテナンス期間中はサービスのご利用ができなくなりますので、ご注意ください。
                    <br />
                    <br />
                    【メンテナンス日時】
                    <br />
                    2024年8月1日（木） 午前2:00 ～ 午前5:00
                    <br />
                    <br />
                    ご不便をおかけいたしますが、ご理解とご協力のほど、よろしくお願いいたします。
                </p>
            </Modal.Body>
            <Modal.Footer onClickCancel={closeModal} />
        </Modal>
    )
}


```

## メリット・デメリット
### メリット
- 直感的なコンポーネント
- コンポーネントの柔軟性があがる
- `LocalState` や `GlobalState` がコンポーネント内部で完結することができる、使用する側は意識せずに使用できる

### デメリット
- 親コンポーネント→子コンポーネントや、孫コンポーネントに`props`を渡したい場合、複雑になる。
<details>
<summary>Ex: React.ChildrenやReact.cloneElementを使う場合 </summary>
    
今回はドロップダウンを実現するコンポーネントを例に挙げる　　
ChildrenやcloneElementがレガシーなコードになるため、注意が必要
```tsx
const SelectMenu = ({
  valueRender,
  children,
  onChange,
  testProps,
}: PropsWithChildren<{
  onChange: (value: string) => void;
  valueRender: ReactNode | string;
  testProps?: string;
}>) => {
  const [menu, setMenu] = useState<'open' | 'close'>('close');

  return (
    <div>
      <button
        type="button"
        className="..."
        onClick={() =>
          setMenu((state) => (state === 'open' ? 'close' : 'open'))
        }
      >
        {valueRender}
      </button>
      {menu === 'open' && (
        <ul className="...">
            // NOTE: childrenをmapして、onChangeを追加
          {Children.map(children, (child) =>
            isValidElement(child)
              ? cloneElement(child, {
                  onChange: () => {
                    onChange(child.props.value);
                    setMenu('close');
                  },
                  testProps: testProps,
                  // testProps: 'dummyProps', // シャローマージにより上書きされてしまう
                })
              : child,
          )}
        </ul>
      )}
    </div>
  );
};

const SelectMenuItem = ({
  children,
  value,
  onChange,
}: PropsWithChildren<{
  value: string;
  onChange?: (value: string) => void;
}>) => {
  return (
    <li
      className="..."
      onClick={onChange && (() => onChange(value))}
    >
      <div className="flex gap-3 items-center">{children}</div>
    </li>
  );
};

SelectMenu.Item = SelectMenuItem;

export { SelectMenu };
```


実装コード
```tsx
const SelectMenuPage = () => {
    return (
        <SelectMenu
            valueRender={selected ? selected : '未選択'}
            onChange={onChangeMenu}
        >
            <SelectMenu.Item value="instagram">
                <Icon variant="instagram" />
                <Typography variant="tertiary" size="sm">
                    Instagram
                </Typography>
            </SelectMenu.Item>
            <SelectMenu.Item value="linkedin">
                <Icon variant="linkedin" />
                <Typography variant="tertiary" size="sm">
                    LinkedIn
                </Typography>
            </SelectMenu.Item>
            <SelectMenu.Item value="x">
                <Typography variant="tertiary" size="sm">
                    X（旧Twitter）
                </Typography>
            </SelectMenu.Item>
        </SelectMenu>
    )
}
```
</details>

<details>
<summary>createContextなどを用いたGlobalStoreを使う場合</summary>


上記と同じ様にドロップダウンを実現するコンポーネントを例に挙げる  
`select-menu.tsx`
```tsx
const SelectMenuContext = createContext<{
  menu: 'open' | 'close';
  setMenu: Dispatch<SetStateAction<'open' | 'close'>>;
  selectedValue: ReactElement | undefined;
  setSelectedValue: Dispatch<SetStateAction<ReactElement | undefined>>;
}>({
  menu: 'close',
  setMenu: () => {},
  selectedValue: undefined,
  setSelectedValue: () => {},
});

const SelectMenu = ({ children }: PropsWithChildren) => {
  const [menu, setMenu] = useState<'open' | 'close'>('close');
  const [selectedValue, setSelectedValue] = useState<ReactElement | undefined>(
    undefined,
  );

  return (
    <SelectMenuContext.Provider
      value={{ menu, setMenu, selectedValue, setSelectedValue }}
    >
      {children}
    </SelectMenuContext.Provider>
  );
};

type SelectMenuListProps = {};

const SelectMenuList = ({
  children,
}: PropsWithChildren<SelectMenuListProps>) => {
  const { menu, setMenu, selectedValue } = useContext(SelectMenuContext);

  return (
    <div className="w-1/4">
      <button
        type="button"
        className="relative w-full cursor-default rounded-md bg-white py-1.5 pl-3 pr-10 text-left text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 sm:text-sm sm:leading-6"
        onClick={() =>
          setMenu((state) => (state === 'open' ? 'close' : 'open'))
        }
      >
        {selectedValue ?? '未選択'}
      </button>
      {menu === 'open' && (
        <ul className="absolute z-10 mt-1 max-h-56 w-1/4 overflow-auto rounded-md bg-white py-1 text-base shadow-lg ring-1 ring-black ring-opacity-5 focus:outline-none sm:text-sm">
          {children}
        </ul>
      )}
    </div>
  );
};

const SelectMenuItem = ({
  children,
  value,
}: PropsWithChildren<{
  value: string;
}>) => {
  const { setSelectedValue, setMenu } = useContext(SelectMenuContext);

  return (
    <li
      className="relative cursor-default select-none py-2 pl-3 pr-9 text-gray-900"
      onClick={() => {
        setMenu('close');
        setSelectedValue(() => (
          <div className="flex gap-3 items-center">{children}</div>
        ));
      }}
    >
      <div className="flex gap-3 items-center">{children}</div>
    </li>
  );
};

SelectMenu.List = SelectMenuList;
SelectMenu.Item = SelectMenuItem;

export { SelectMenu };
```

実装コード
```tsx
export const SelectMenuPage = () => {
    return (
        <SelectMenu>
            <SelectMenu.List>
                <SelectMenu.Item value="instagram">
                    <Icon variant="instagram" />
                    <Typography variant="tertiary" size="sm">
                        Instagram
                    </Typography>
                </SelectMenu.Item>

                <SelectMenu.Item value="linkedin">
                    <Icon variant="linkedin" />
                    <Typography variant="tertiary" size="sm">
                        LinkedIn
                    </Typography>
                </SelectMenu.Item>

                <SelectMenu.Item value="x">
                    <Typography variant="tertiary" size="sm">
                        X（旧Twitter）
                    </Typography>
                </SelectMenu.Item>
            </SelectMenu.List>
        </SelectMenu>
    );
};
```

</details>

---
## 感想
今回のユースケースで提示したような、コンポーネント間でpropsを共有しないケースは、積極的に使っていいと思う
コンポーネント間でprops渡しが発生する場合は、実装方法が主に２パターンがあるので、チーム内で議論して決めていく必要があると思う。



---
## 参考文献
<a href="https://tegehoge.connpass.com/event/323105/" target="_blank">React勉強会 Vol.1</a>  
<a href="https://tyotto-good.com/react/compound-component" target="_blank">【React デザインパターン】Compound Components を使って直感的なコンポーネントを実装しよう</a>  
<a href="https://www.patterns.dev/react/compound-pattern" target="_blank">Compound Pattern</a>  