# useRecoilSnapshot()

このhookは、レンダリング中に[Snapshot](https://qiita.com/Daichi44/items/84a7218e76f62d41d0b7)オブジェクトを同期的に返し、すべてのRecoil state変更の呼び出し元コンポーネントを登録します。​このhookは、デバッグツールや、初期レンダリング中にstateを同期させる必要があるサーバサイドレンダリングに使用できます。

```React
function useRecoilSnapshot(): Snapshot
```

このhookを使用すると、すべてのRecoil stateの変更に対してコンポーネントが再レンダリングされるため、注意が必要です。​将来は、パフォーマンスのためにデバウンスする機能を提供したいと考えています。

## ​Linkサンプルコード
​`<a>`アンカーに変換が適用された現在の状態に基づいてhrefをレンダリングする`<LinkToNewView>`コンポーネントを定義します。​この例では、`uriFromSnapshot()`は、ページのロード時に復元できるURIの現在の状態をエンコードするユーザ定義関数です。

```React
function LinkToNewView() {
  const snapshot = useRecoilSnapshot();
  const newSnapshot = snapshot.map(({set}) => set(viewState, 'New View'));
  return <a href={uriFromSnapshot(newSnapshot)}>Click Me!</a>;
}
```

これは簡単な例です。​私たちはこのようなヘルパーを提供して、より拡張性が高く最適化されたブラウザの履歴の永続性し、ライブラリーにリンクを生成します。​たとえば、clickハンドラをハイジャックして、ブラウザの履歴を置き換えるローカルstateを更新します。