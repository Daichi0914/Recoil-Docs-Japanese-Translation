# useRecoilValueLoadable(state)

このhookは、非同期selectorの値を読み取るために使用することを目的としています。​このhookは、コンポーネントを指定されたstateに暗黙的に登録します。

​[`useRecoilValue()`](https://qiita.com/Daichi44/items/f78a2ce0cfc01de354c5)とは異なり、このhookは非同期selector([React Suspense](https://reactjs.org/docs/concurrent-mode-suspense.html)と連携する目的で)から読み込む際に`Error`や`Promise`をスローしません。​代わりに、このhookは[`Loadable`](https://qiita.com/Daichi44/items/812aa5ebf149c849e108)オブジェクトを返します。

***

```React
function useRecoilValueLoadable<T>(state: RecoilValue<T>): Loadable<T>
```

* `state`: いくつかの非同期操作を持つ[`atom`](https://qiita.com/Daichi44/items/0a9b9af69dddddbcd7e2)または[`selector`](https://qiita.com/Daichi44/items/13912706763c22f9cbe9)。​返されるロード可能ファイルのステータスは、指定されたstate selectorのステータスによって異なります。

​インタフェースの現在のstateの[`Loadable`](https://qiita.com/Daichi44/items/812aa5ebf149c849e108)を返します。

* `state`: selectorのステータスを示します。​取り得る値は`'hasValue'`(値あり), `'hasError'`(エラーあり), または`'loading'`(ロード中)である。
* `contents`: この`Loadable`によって表される値。​stateが`hasValue`の場合は実際の値、stateが`hasError`の場合はスローされた`Error`オブジェクト、stateが`loading`の場合は値の`Promise`です。

***

## サンプルコード

```React
function UserInfo({userID}) {
  const userNameLoadable = useRecoilValueLoadable(userNameQuery(userID));
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