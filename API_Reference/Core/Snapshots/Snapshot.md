# class Snapshot

​Snapshotオブジェクトは、[atom](https://qiita.com/Daichi44/items/0a9b9af69dddddbcd7e2)のRecoil stateの不変snapshotを表します。​これは、グローバルなRecoil stateを観察、検査、管理するためのAPIを標準化することを意図しています。​主に、開発ツール、グローバルstate同期、履歴ナビゲーションなどに役立ちます。

```React
class Snapshot {
  // Accessors to inspect snapshot state
  getLoadable: <T>(RecoilValue<T>) => Loadable<T>;
  getPromise: <T>(RecoilValue<T>) => Promise<T>;

  // API to transform snapshots for transactions
  map: (MutableSnapshot => void) => Snapshot;
  asyncMap: (MutableSnapshot => Promise<void>) => Promise<Snapshot>;

  // Developer Tools API
  getID: () => SnapshotID;
  getNodes_UNSTABLE: ({
    isModified?: boolean,
  } | void) => Iterable<RecoilValue<mixed>>;
  getInfo_UNSTABLE: <T>(RecoilValue<T>) => {...};
}

function snapshot_UNSTABLE(initializeState?: (MutableSnapshot => void)): Snapshot
```

## Snapshotの取得
### Hooks

​Recoilには、現在のstateに基づいてsnapshotを取得するための次のhookが用意されています。

​* [`useRecoilCallback()`](https://qiita.com/Daichi44/items/0a276eb144f443a72efd) - snapshotへの非同期アクセス
​* [`useRecoilSnapshot()`](https://qiita.com/Daichi44/items/eda197468b0204349d5f) - snapshotへの同期アクセス
​* [`useRecoilTransactionObserver_UNSTABLE()`](https://qiita.com/Daichi44/items/de7e39f80cd1fdce8b7c) - すべてのstate変更に対してsnapshotを登録

### snapshotの構築
​`snapshot_UNSTABLE()`ファクトリを使用して新しいsnapshotを作成することもできます。このファクトリは省略可能な初期化関数を受け付けます。​これは、Reactコンテキスト以外でselectorを[テスト](https://recoiljs.org/docs/guides/testing)または評価するために使用できます。

## snapshotの読み取り
​snapshotは、atomのstateに関しては読み取り専用です。​atomのstateを読み取り、selectorの派生stateを評価するために使用できます。​`getLoadable()`は、このsnapshotのatomまたはselectorのstateを[`Loadable`](https://qiita.com/Daichi44/items/812aa5ebf149c849e108)に提供します。​`getPromise()`メソッドを使用すると、非同期selectorの値が評価されるのを待つことができるので、その値が静的なatom stateに基づいているかどうかを確認できます。

### サンプルコード

```React
function MyComponent() {
  const logState = useRecoilCallback(({snapshot}) => () => {
    console.log("State: ", snapshot.getLoadable(myAtom).contents);

    const newSnapshot = snapshot.map(({set}) => set(myAtom, 42));
  });
}
```

## ​Snapshotの変換

​snapshotを変更したい場合があります。​snapshotは不変ですが、新しい不変(Mutable)なsnapshotに変換セットをmapするメソッドがあります。​mapメソッドは、MutableSnapshotに渡されるコールバックを取ります。MutableSnapshotはコールバック中に変更され、最終的にマッピング操作によって返される新しいsnapshotになります。

```React
class MutableSnapshot {
  set: <T>(RecoilState<T>, T | DefaultValue | (T => T | DefaultValue)) => void;
  reset: <T>(RecoilState<T>) => void;
}
```

​`set()`と`reset()`は、書き込み可能なselectorのsetプロパティに提供されるコールバックと同じ[シグネチャ](http://e-words.jp/w/%E3%82%B7%E3%82%B0%E3%83%8D%E3%83%81%E3%83%A3.html)を持ちますが、新しいsnapshotにのみ影響し、現在のstateには影響しないことに注意してください。

## Snapshotへの移動

​次のhookを使用して、現在のRecoil stateを指定のsnapshotに移動できます。

​[`useGotoRecoilSnapshot()`](https://qiita.com/Daichi44/items/0bbe5d6643462b55df11) - snapshotに合わせて現在のstateを更新する

## 開発者ツール

​snapshotには、Recoilを使用した開発者ツールの構築やデバッグ機能に役立つメソッドがいくつか用意されています。​このAPIはまだ進化しており、初期の開発ツールでは`_UNSTABLE`となっています。

### Snapshot IDs

​コミットされたstateまたは変更されたsnapshotには、`getID()`で取得できる一意の不透明なバージョンIDがあります。​これは、`useGotoRecoilSnapshot()`によって前のsnapshotに戻ったことを検出するために使用できます。

### atomとselectorの列挙

​`getNodes_UNSTABLE()`メソッドを使用すると、このsnapshotで使用されていたすべてのatomとselectorを反復できます。​atom、selector、およびファミリー(?)はいつでも作成できます。​ただし、実際に使用された場合にのみsnapshotに表示されます。​atomおよびselectorは、それらが既に使用されていない場合、後続のstate snapshotから除去されます。

​オプションの`isModified`フラグを指定すると、最後の[トランザクション](https://www.sophia-it.com/content/%E3%83%88%E3%83%A9%E3%83%B3%E3%82%B6%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3)以降に変更されたatomのみを返すことができます。

### デバッグ情報

​`getInfo_UNSTABLE()`メソッドは、atomとselectorに関する追加のデバッグ情報を提供します。​提供されるデバッグ情報は進化していますが、次の情報が含まれる場合があります。

* `loadable` - 現在のstateでロード可能です。​`getLoadable()`などのメソッドとは異なり、このメソッドはsnapshotをまったく変更しません。​現在のstateを提供し、新しいatom/selectorの初期化、新しいselector評価の実行、依存関係やサブスクリプション(購読)の更新は行いません。
* `​isSet` - snapshot stateに格納されている明示的な値を持つatomの場合はTrue。​selectorの場合、またはデフォルトのatom stateを使用する場合はfalse。
* `isModified` - 最後の[トランザクション](https://www.sophia-it.com/content/%E3%83%88%E3%83%A9%E3%83%B3%E3%82%B6%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3)以降に変更されたatomの場合にTrue。
* `type` - `atom`または`selector`のいずれか。
* `​deps` - このノードが依存するatomまたはselectorに対するイテレータ。
* `subscribers` - このsnapshotのこのノードに登録しているものに関する情報。​詳細は開発中です。

## ​stateの初期化

​[`<RecoilRoot>`](https://qiita.com/Daichi44/items/b440f33d3831b86c62b9)コンポーネントおよび`snapshot_UNSTABLE()`ファクトリは、`MutableSnapshot`を介してstateを初期化するためのオプションの`initializeState`propを取ります。​これは、すべてのatomが事前にわかっている場合に永続stateをロードするのに役立ち、初期レンダリングと同期してstateを設定する必要があるサーバサイドレンダリングと互換性があります。​atomごとの初期化/永続化、および動的atomの操作の容易さについては、[atom effects](https://recoiljs.org/docs/guides/atom-effects)を参照してください。
