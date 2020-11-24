# useRecoilCallback(callback, deps)

​このhookは[`useCallback()`](https://reactjs.org/docs/hooks-reference.html#usecallback)に似ていますが、コールバックがRecoil stateで動作するためのAPIも提供します。​このhookを使用すると、Recoil stateの読み取り専用の[`Snapshot`](https://qiita.com/Daichi44/items/84a7218e76f62d41d0b7)へのアクセス権と、現在のRecoil stateを非同期に更新する機能を持つコールバックを構築できます。

​このhookを使用する動機には、次のようなものがあります。

* atomまたはselectorが更新された場合、Reactコンポーネントを登録せずにRecoil stateを非同期に読み取り、再描画します。
* ​レンダリング時に実行したくない非同期アクションに対して高価な検索を延期します。
* Recoil stateへの読み取りまたは書き込みを行う場合に、副作用を実行します。
* レンダリング時に更新するatom、またはselectorがわからない可能性があるatom、またはselectorを動的に更新するため、[`useSetRecoilState()`](https://qiita.com/Daichi44/items/ab0e98e0a48d0a59bdb4)を使用できません。
* レンダリング前にデータを[プリフェッチ](https://qiita.com/Daichi44/items/47e9ac34c61c35531abb#%E3%83%97%E3%83%AA%E3%83%95%E3%82%A7%E3%83%83%E3%83%81)しています。

***

```React
type CallbackInterface = {
  snapshot: Snapshot,
  gotoSnapshot: Snapshot => void,
  set: <T>(RecoilState<T>, (T => T) | T) => void,
  reset: <T>(RecoilState<T>) => void,
};

function useRecoilCallback<Args, ReturnValue>(
  callback: CallbackInterface => (...Args) => ReturnValue,
  deps?: $ReadOnlyArray<mixed>,
): (...Args) => ReturnValue
```

* `​callback` - コールバックインターフェースを提供するラッパー関数を持つユーザーコールバック関数。​stateを変更するコールバックはキューに入れられ、現在のRecoil stateを非同期に更新します。​ラップされた関数の型[シグネチャ](http://e-words.jp/w/%E3%82%B7%E3%82%B0%E3%83%8D%E3%83%81%E3%83%A3.html)は、返されたコールバックの型[シグネチャ](http://e-words.jp/w/%E3%82%B7%E3%82%B0%E3%83%8D%E3%83%81%E3%83%A3.html)と一致します。
* `deps` - コールバックを記憶するための、オプションの依存関係のセット。​`useCallback()`と同様に、生成されたコールバックはデフォルトでは記憶されず、レンダーのたびに新しい関数が生成されます。​空の配列を渡すと、常に同じ関数[インスタンス](http://e-words.jp/w/%E3%82%A4%E3%83%B3%E3%82%B9%E3%82%BF%E3%83%B3%E3%82%B9.html)を返すことができます。​deps配列に値を渡すと、depの参照の等価性が変更された場合に新しい関数が使用されます。​これらの値は、コールバックの本体内から、古くならずに使用できます。​([`useCallback`](https://reactjs.org/docs/hooks-reference.html#usecallback)を参照) [eslintを更新](https://recoiljs.org/docs/introduction/installation#eslint)して、これが正しく使用されるようにすることができます。

コールバックインターフェース:

* `snapshot` - [`Snapshot`](https://qiita.com/Daichi44/items/84a7218e76f62d41d0b7)は、コールバックが呼び出された現在の[トランザクション](http://e-words.jp/w/%E3%83%88%E3%83%A9%E3%83%B3%E3%82%B6%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3.html)が開始されたときにReactバッチでコミットされたRecoil atomのstateを読み取り専用で表示します。​atomの値は静的ですが、非同期selectorはまだ保留中または解決中の可能性があります。
* `gotoSnapshot` - 指定された[`Snapshot`](https://qiita.com/Daichi44/items/84a7218e76f62d41d0b7)に一致するようにグローバルstateを更新する[エンキュー](http://e-words.jp/w/%E3%82%AD%E3%83%A5%E3%83%BC.html#Section_%E3%82%A8%E3%83%B3%E3%82%AD%E3%83%A5%E3%83%BC/%E3%83%87%E3%82%AD%E3%83%A5%E3%83%BC)を行います。
* `set` - atomまたはselectorの値を設定する[エンキュー](http://e-words.jp/w/%E3%82%AD%E3%83%A5%E3%83%BC.html#Section_%E3%82%A8%E3%83%B3%E3%82%AD%E3%83%A5%E3%83%BC/%E3%83%87%E3%82%AD%E3%83%A5%E3%83%BC)。​他の場所と同様に、新しい値を直接指定することも、新しい値を返し、現在の値をパラメータとして受け取る更新関数を指定することもできます。​現在の値は、現在の[トランザクション](http://e-words.jp/w/%E3%83%88%E3%83%A9%E3%83%B3%E3%82%B6%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3.html)内の他のすべての[エンキュー](http://e-words.jp/w/%E3%82%AD%E3%83%A5%E3%83%BC.html#Section_%E3%82%A8%E3%83%B3%E3%82%AD%E3%83%A5%E3%83%BC/%E3%83%87%E3%82%AD%E3%83%A5%E3%83%BC)されたstateの変更を表します。
* `reset` - atomまたはselectorの値を既定値にリセットします。

## ​遅延読み取りの例

​この例では、**`useRecoilCallback()`**を使用して、stateが変化したときに再レンダリングするコンポーネントを登録せずに、stateを遅延読み込みします。

```React
import {atom, useRecoilCallback} from 'recoil';

const itemsInCart = atom({
  key: 'itemsInCart',
  default: 0,
});

function CartInfoDebug() {
  const logCartItems = useRecoilCallback(({snapshot}) => async () => {
    const numItemsInCart = await snapshot.getPromise(itemsInCart);
    console.log('Items in cart: ', numItemsInCart);
  });

  return (
    <div>
      <button onClick={logCartItems}>Log Cart Items</button>
    </div>
  );
}
```