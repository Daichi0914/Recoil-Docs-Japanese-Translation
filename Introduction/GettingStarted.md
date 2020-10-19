# 入門<br>

## Create React App
<br>

RecoilはReactの状態管理ライブラリであるため、Recoilを使用するには、Reactをインストールして実行する必要があります。Reactアプリケーションをブートストラップするための最も簡単で推奨される方法は、`Create ReactApp`を使用することです。
<br>

```
npx create-react-app my-app
```

`npxはnpm5.2以降に付属するパッケージランナーツールです。古いnpmバージョンの手順を参照してください。`
<br>

Create React Appをインストールするその他の方法については、公式ドキュメントを参照してください。

<br>
<br>

## Installation
<br>

Recoilパッケージはnpmにあります。最新の安定バージョンをインストールするには、次のコマンドを実行します。
<br>
```
npm install recoil
```
または、yarnを使用している場合:
```
yarn add recoil
```
<br>
<br>

## RecoilRoot
<br>
反動状態を使用するコンポーネントRecoilRootは、親ツリーのどこかに表示される必要があります。これを配置するのに適した場所は、ルートコンポーネントです。

```
import React from 'react';
import {
  RecoilRoot,
  atom,
  selector,
  useRecoilState,
  useRecoilValue,
} from 'recoil';

function App() {
  return (
    <RecoilRoot>
      <CharacterCounter />
    </RecoilRoot>
  );
}
```

CharacterCounter次のセクションでコンポーネントを実装します。

<br>
<br>

## Atom
<br>
Atomは片表す状態。Atomは、任意のコンポーネントから読み取りおよび書き込みを行うことができます。Atomの値を読み取るコンポーネントは暗黙的にそのAtomにサブスクライブされているため、Atomを更新すると、そのAtomにサブスクライブされているすべてのコンポーネントが再レンダリングされます。

```
const textState = atom({
  key: 'textState', // unique ID (with respect to other atoms/selectors)
  default: '', // default value (aka initial value)
});
```

Atomからの読み取りとAtomへの書き込みが必要なコンポーネントは、useRecoilState()以下に示すように使用する必要があります。

```
function CharacterCounter() {
  return (
    <div>
      <TextInput />
      <CharacterCount />
    </div>
  );
}

function TextInput() {
  const [text, setText] = useRecoilState(textState);

  const onChange = (event) => {
    setText(event.target.value);
  };

  return (
    <div>
      <input type="text" value={text} onChange={onChange} />
      <br />
      Echo: {text}
    </div>
  );
}
```

<br>
<br>

## Selector
<br>
Selectorは、片表す派生状態。派生状態は、状態の変換です。派生状態は、特定の状態を何らかの方法で変更する純粋関数に状態を渡す出力と考えることができます。

```
const charCountState = selector({
  key: 'charCountState', // unique ID (with respect to other atoms/selectors)
  get: ({get}) => {
    const text = get(textState);

    return text.length;
  },
});
```

useRecoilValue()フックを使用して、charCountStateの値を読み取ることができます。

```
function CharacterCount() {
  const count = useRecoilValue(charCountState);

  return <>Character Count: {count}</>;
}
```
