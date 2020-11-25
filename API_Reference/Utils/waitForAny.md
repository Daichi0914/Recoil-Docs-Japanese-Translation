# waitForAny(dependencies)

要求された依存関係の現在のstateに対する[`Loadable`s](https://qiita.com/Daichi44/items/812aa5ebf149c849e108)のセットを返す並行性ヘルパー。​少なくとも1つの依存関係が使用可能になるまで待機します。

​依存関係は、タプル配列またはオブジェクト内の名前付き依存関係として指定できます。

***

```React
function waitForAny(dependencies: Array<RecoilValue<>>):
  RecoilValueReadOnly<UnwrappedArrayOfLoadables>
```

```React
function waitForAny(dependencies: {[string]: RecoilValue<>}):
  RecoilValueReadOnly<UnwrappedObjectOfLoadables>
```

`​waitForAny()`は[`waitForNone()`](https://qiita.com/Daichi44/items/c9bb722022a003d72aa9)に似ていますが、少なくともひとつの依存性がすぐに値を返すのではなく、利用可能になるまで待つという点が異なります。
