# useRecoilTransactionObserver_UNSTABLE(callback)

## NOTE:このAPIは不安定であると考えてください

​このhookは、atomのstateが変更されるたびに実行されるコールバックを登録します。​1つの[トランザクション](https://www.sophia-it.com/content/%E3%83%88%E3%83%A9%E3%83%B3%E3%82%B6%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3)で複数の更新をバッチ処理できます。​このhookは、state変更、開発ツール、ビルド履歴などを保持するのに最適です。

```React
function useRecoilTransactionObserver_UNSTABLE(({
  snapshot: Snapshot,
  previousSnapshot: Snapshot,
}) => void)
```

​コールバックは、Reactバッチトランザクションの現在および前のstateの[`snapshot`](https://qiita.com/Daichi44/items/84a7218e76f62d41d0b7)を提供します。​個々のatomの変更のみを登録したい場合は、代わりにeffectsを考慮してください。​将来的には、特定の条件に登録する機能や、パフォーマンスに関する[デバウンス](https://slideship.com/users/@iktakahiro/presentations/2017/12/MArWbm3VYEKCqB2ZNd5dts/)を提供できるようになる可能性があります。

## デバッグの例

```React
function DebugObserver() {
  useRecoilTransactionObserver_UNSTABLE(({snapshot}) => {
    window.myDebugState = {
      a: snapshot.getLoadable(atomA).contents,
      b: snapshot.getLoadable(atomB).contents,
    };
  });
  return null;
}

function MyApp() {
  return (
    <RecoilRoot>
      <DebugObserver />
      ...
    </RecoilRoot>
  );
}
```