# useRecoilStateLoadable(state)

このhookは、非同期selectorの値を読み取るために使用することを目的としています。​このhookは、コンポーネントを指定されたstateに暗黙的に登録します。

​[`useRecoilState()`](https://qiita.com/Daichi44/items/2ec591ec3e952f1784c3)とは異なり、このhookは非同期selector([React Suspense](https://reactjs.org/docs/concurrent-mode-suspense.html)と連携する目的で)から読み込む際に`Error`や`Promise`をスローしません。​代わりに、このhookは値の[`Loadable`](https://qiita.com/Daichi44/items/812aa5ebf149c849e108)をsetterコールバックとともに返します。

***

```React
function useRecoilStateLoadable<T>(state: RecoilState<T>): [Loadable<T>, (T | (T => T)) => void]
```

* ​`state`: いくつかの非同期操作を持つ書き込み可能なatomまたはselector。​返されるロード可能ファイルのステータスは、指定されたstate selectorのステータスによって異なります。

インタフェースの現在のstateの[`Loadable`](https://qiita.com/Daichi44/items/812aa5ebf149c849e108)を返します。

* `state`: selectorのステータスを示します。​取り得る値は`'hasValue'`(値あり), `'hasError'`(エラーあり), または`'loading'`(ロード中)である。
* `contents`: この`Loadable`によって表される値。​stateが`hasValue`の場合は実際の値、stateが`hasError`の場合はスローされた`Error`オブジェクト、stateが`loading`の場合は値の`Promise`です。

***

## サンプルコード

```React
function UserInfo({userID}) {
  const [userNameLoadable, setUserName] = useRecoilStateLoadable(userNameQuery(userID));
  switch (userNameLoadable.state) {
    case 'hasValue':
      return <div>{userNameLoadable.contents}</div>;
    case 'loading':
      return <div>Loading...</div>;
    case 'hasError':
      throw userNameLoadable.contents;
  }
}
```