# waitForNone(dependencies)

要求された依存関係の現在のstateに対する[`Loadable`](https://qiita.com/Daichi44/items/812aa5ebf149c849e108)のセットを返す並行性ヘルパー。​

依存関係は、タプル配列またはオブジェクト内の名前付き依存関係として指定できます。

***

```React
function waitForNone(dependencies: Array<RecoilValue<>>):
  RecoilValueReadOnly<UnwrappedArrayOfLoadables>
```

```React
function waitForNone(dependencies: {[string]: RecoilValue<>}):
  RecoilValueReadOnly<UnwrappedObjectOfLoadables>
```

***

​`waitForNone()`は[`waitForAll()`](https://qiita.com/Daichi44/items/b929ff5376d0672dafa0)と似ていますが、値を直接返すのではなく、すぐに戻り、各依存関係に対して[`Loadable`](https://qiita.com/Daichi44/items/812aa5ebf149c849e108)を返す点が異なります。​[`noWait()`](https://qiita.com/Daichi44/items/ca11fcef04d764ef13a8)と似ていますが、一度に複数の依存関係を要求できる点が異なります。

​このヘルパーは、部分的なデータを操作する場合や、別のデータが使用可能になったときにUIを段階的に更新する場合に便利です。

## Incremental Loading の例
​次の使用例は、複数のレイヤを持つグラフを表示します。​各画層には、コストがかかる可能性のあるデータクエリーがあります。​ペンディング(保留)中の各レイヤーの編集ボックスを使用してグラフを即座にレンダリングし、そのレイヤーのデータが入力されるたびにグラフを更新して新しい各レイヤーを追加します。​クエリーでエラーが発生した画層がある場合、その画層だけにエラーメッセージが表示され、残りの画層は引き続きレンダリングされます。

```React
function MyChart({layerQueries}: {layerQueries: Array<RecoilValue<Layer>>}) {
  const layerLoadables = useRecoilValue(waitForNone(layerQueries));

  return (
    <Chart>
      {layerLoadables.map((layerLoadable, i) => {
        switch (layerLoadable.state) {
          case 'hasValue':
            return <Layer key={i} data={layerLoadable.contents} />;
          case 'hasError':
            return <LayerErrorBadge key={i} error={layerLoadable.contents} />;
          case 'loading':
            return <LayerWithSpinner key={i} />;
        }
      })}
    </Chart>
  );
}
```