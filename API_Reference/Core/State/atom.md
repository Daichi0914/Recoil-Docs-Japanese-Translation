# `atom(options)`

atomはRecoilのstateを表します。`atom()`関数は書き込み可能なRecoilStateオブジェクトを返します。

***

```React
function atom<T>({
  key: string,
  default: T | Promise<T> | RecoilValue<T>,

  effects_UNSTABLE?: $ReadOnlyArray<AtomEffect<T>>,

  dangerouslyAllowMutability?: boolean,
}): RecoilState<T>
```

* `key` - atomを内部的に識別するために使用される一意の文字列。この文字列は、アプリケーション全体で他のatomおよびselectorに対して一意である必要があります。
* `default` - atom、Promise、または同じタイプの値を表す別のatomまたはselectorの初期値。
* `effects_UNSTABLE` - atomの[Atom Effects](https://recoiljs.org/docs/guides/atom-effects)のオプションの配列。
* `dangerouslyAllowMutability` - Recoilは、atomのstateの変化に依存して、atomを再描画に使用するコンポーネントに通知するタイミングを決定します。atomの値が変更された場合、これをバイパスし、登録するコンポーネントに適切に通知せずにstateを変更する可能性があります。これを防ぐために、保存されている値はすべて凍結されます。場合によっては、このオプションを使用してこれをオーバーライドすることが望ましいこともあります。

***

ほとんどの場合、次のhookを使用してatomを操作します。

* [`useRecoilState()`](https://qiita.com/Daichi44/items/2ec591ec3e952f1784c3):このhookは、atomに対する読み取りと書き込みの両方を行う場合に使用します。このhookは、コンポーネントをatomに登録します。
* [`useRecoilValue()`](https://qiita.com/Daichi44/items/f78a2ce0cfc01de354c5):このhookは、atomの読み取りのみを行う場合に使用します。このhookは、コンポーネントをatomに登録します。
* [`useSetRecoilState()`](https://qiita.com/Daichi44/items/ab0e98e0a48d0a59bdb4):このhookは、atomへの書き込みのみを行う場合に使用します。
* [`useResetRecoilState()`](https://qiita.com/Daichi44/items/a887cfa33260136b1d68):このhookを使用して、atomをデフォルト値にリセットします。

コンポーネントを登録せずにatomの値を読み取る必要があるまれなケースについては、[`useRecoilCallback()`](https://qiita.com/Daichi44/items/0a276eb144f443a72efd)を参照してください。

atomは静的な値で初期化することも、`Promise`や同じ型の値を表す`RecoilValue`で初期化することもできます。
`Promise`は保留中の可能性があり、デフォルトselectorは非同期かもしれないため、これはatomの値も保留中の可能性があり、読み込み時にエラーをスローするかもしれないということを意味します。
atomの設定時に`Promise`を割り当てることはできません。非同期関数には[selector](https://qiita.com/Daichi44/items/13912706763c22f9cbe9)を使用してください。

atomを使って`Promise`や`RecoilValue`を直接格納することはできませんが、オブジェクトにラップすることはできます。`Promise`は変更可能(ミュータブル)かもしれないことに注意してください。atomは純粋であれば関数に設定することができますが、そのためにはupdater形式のセッターを使う必要があるかもしれません。(例: `set (myAtom, ()=>myFunc);`)

## サンプルコード

```React
import {atom, useRecoilState} from 'recoil';

const counter = atom({
  key: 'myCounter',
  default: 0,
});

function Counter() {
  const [count, setCount] = useRecoilState(counter);
  const incrementByOne = () => setCount(count + 1);

  return (
    <div>
      Count: {count}
      <br />
      <button onClick={incrementByOne}>Increment</button>
    </div>
  );
}
```
