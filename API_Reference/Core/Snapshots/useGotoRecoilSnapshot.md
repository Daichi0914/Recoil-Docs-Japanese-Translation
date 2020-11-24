# useGotoRecoilSnapshot(snapshot)

​このhookは、snapshotをパラメータとして取り、現在の[`<Recoil Root>`](https://qiita.com/Daichi44/items/b440f33d3831b86c62b9)stateをこのatomのstateに一致するように更新するコールバックを戻します。

```React
function useGotoRecoilSnapshot(): Snapshot => void
```

## トランザクションの例
​**重要な注意**: この例は、すべてのstate変更に対して再描画するようにコンポーネントを登録するため、効率的ではありません。

```React
function TransactionButton(): React.Node {
  const snapshot = useRecoilSnapshot(); // Subscribe to all state changes
  const modifiedSnapshot = snapshot.map(({set}) => {
    set(atomA, x => x + 1);
    set(atomB, x => x * 2);
  });
  const gotoSnapshot = useGotoRecoilSnapshot();
  return <button onClick={() => gotoSnapshot(modifiedSnapshot)}>Perform Transaction</button>;
}
```

## Time Travel Example
​[Time Travel Example](https://recoiljs.org/docs/guides/dev-tools#time-travel)を参照してください。