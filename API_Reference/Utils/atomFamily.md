# atomFamily(options)

​書き込み可能な`RecoilState` [atom](https://qiita.com/Daichi44/items/0a9b9af69dddddbcd7e2)を返す関数を返します。

***

```React
function atomFamily<T, Parameter>({
  key: string,

  default:
    | RecoilValue<T>
    | Promise<T>
    | T
    | (Parameter => T | RecoilValue<T> | Promise<T>),

  effects_UNSTABLE?:
    | $ReadOnlyArray<AtomEffect<T>>
    | (P => $ReadOnlyArray<AtomEffect<T>>),

  dangerouslyAllowMutability?: boolean,
}): Parameter => RecoilState<T>
```

* ​`key` - atomを内部的に識別するために使用される一意の文字列。​この文字列は、アプリケーション全体で他のatomおよびselectorに対して一意である必要があります。
* `default` - atomの初期値。​直接値を指定するか、デフォルト値を表す`RecoilValue`または`Promise`を指定するか、あるいはデフォルト値を取得する関数を指定します。​コールバック関数には、`atomFamily`関数の呼び出し時に使用されるパラメータのコピーが渡されます。
* `effects_UNSTABLE` - Atom Effectsのオプションの配列、あるいはfamilyパラメータに基づいて配列を取得するためのコールバック。
* `dangerouslyAllowMutability` - Recoilは、atomのstateの変化に依存して、atomを再描画に使用するコンポーネントに通知するタイミングを決定します。​atomの値が変更された場合、これをバイパスし、登録するコンポーネントに適切に通知せずにstateを変更する可能性があります。​これを防ぐために、保存されている値はすべて凍結されます。​場合によっては、このオプションを使用してこれをオーバーライドすることが望ましいこともあります。

***

​atomはRecoil stateの断片を表します。​atomは<RecoilRoot>ごとにアプリケーションによって作成され、登録されます。しかし、stateがグローバルでない場合はどうなりますか？ ​stateがコントロールの特定のインスタンスに関連付けられている場合、または特定の要素に関連付けられている場合はどうなりますか？ ​例えば、あなたのアプリはユーザが動的に要素を追加することができ、各要素はその位置のようなstateを持っているUIプロトタイピングツールかもしれません。​理想的には、各元素はそれぞれのstate atomをもちます。​これはメモパターンを使って自分で実装できます。​しかし、Recoilは`atomFamily`ユーティリティでこのパターンを提供します。`atomFamily`は、atomの集合を表します。​`atomFamily`を呼び出すと、渡されたパラメータに基づいて`RecoilState`atomを提供する関数が返されます。

​`atomFamily`は基本的に、パラメータからatomへのmapを提供します。​`atomFamily`に提供する必要があるkeyは1つだけで、基礎となる各atomに固有のkeyが生成されます。​これらのatom keyは持続性のために使用できるため、アプリケーションの実行中に安定している必要があります。​パラメータは異なるコールサイトで生成されることもあり、同じ基底atomを使用する同等のパラメータが必要です。​したがって、`atomFamily`パラメータのreference-equalityの代わりにvalue-equalityが使用されます。​これにより、パラメータに使用できるタイプが制限されます。​`atomFamily`は、プリミティブ型、または配列、オブジェクト、プリミティブ型を含む配列またはオブジェクトを受け入れます。

## サンプルコード

```React
const elementPositionStateFamily = atomFamily({
  key: 'ElementPosition',
  default: [0, 0],
});

function ElementListItem({elementID}) {
  const position = useRecoilValue(elementPositionStateFamily(elementID));
  return (
    <div>
      Element: {elementID}
      Position: {position}
    </div>
  );
}
```

​`atomFamily()`は、単純な[`atom()`](https://qiita.com/Daichi44/items/0a9b9af69dddddbcd7e2)とほとんど同じオプションを取ります。​ただし、既定値をパラメータ化することもできます。​つまり、パラメータ値を受け取り、実際のデフォルト値を返す関数を提供できます。​たとえば、次のようになります。

```React
const myAtomFamily = atomFamily({
  key: ‘MyAtom’,
  default: param => defaultBasedOnParam(param),
});
```

または`selector`の代わりに[`selectorFamily`](https://qiita.com/Daichi44/items/f552dfda4765596d83d7)を使用すると、デフォルトselectorでパラメータ値にアクセスすることもできます。

```React
const myAtomFamily = atomFamily({
  key: ‘MyAtom’,
  default: selectorFamily({
    key: 'MyAtom/Default',
    get: param => ({get}) => {
      return computeDefaultUsingParam(param);
    },
  }),
});
```

## Subscriptions (購読)
​すべての要素のstate mapを持つ単一のatomを保存しようとするよりも、各要素の個別のatomにこのパターンを使用する方が優れている点の1つは、すべての要素がそれぞれ独自のサブスクリプションを保持することです。​そのため、1つの要素の値を更新すると、そのatomのみに登録しているReactコンポーネントのみが更新されます。

## Persistence (持続性)
​永続性オブザーバは、使用されるパラメータ値のシリアライズに基づいて一意のkeyを持つ個別のatomとして、各パラメータ値のstateを永続化します。​したがって、プリミティブまたはプリミティブを含む単純な合成オブジェクトであるパラメータのみを使用することが重要です。​カスタムクラスまたは関数は使用できません。

​同じkeyに基づいた新しいバージョンのアプリでは、単純なatomを`atomFamily`に"アップグレード"することができます。​これを行うと、古い単純なkeyを持つ永続化された値も引き続き読み取ることができ、新しい`atomFamily`のすべてのパラメーター値は、デフォルトで単純なatomの永続化されたstateになります。​ただし、`atomFamily`のパラメータのフォーマットを変更した場合、変更前に保持されていた以前の値は自動的に読み込まれません。​ただし、以前のパラメータフォーマットに基づいて値をルックアップする場合は、デフォルトのセレクタまたはバリデータにロジックを追加できます。​将来的には、このパターンの自動化を支援したいと考えています。