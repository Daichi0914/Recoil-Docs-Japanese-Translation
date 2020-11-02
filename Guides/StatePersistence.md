# Stateの永続性

Recoilを使用すると、atomを使用してアプリケーションのstateを保持できます。

***

### *重要なお知らせ*
*このAPIは現在開発中であり、今後変更される予定です。お楽しみに...*

***

## Stateの保存
stateを保存するには、atomの変更を登録し、新しいstateを記録します。React effectsを使用して、個々のatomを登録できます(*[非同期state・同期state](https://qiita.com/Daichi44/items/ae631b34ac50fb9bbd1d)*参照)。ただし、Recoilはhookを提供し、[`useRecoilTransactionObserver_UNSTABLE()`](https://recoiljs.org/docs/api-reference/core/useRecoilTransactionObserver)を使用してすべてのatomのstate変更を登録できるようにします。

サブスクリプションコールバックは、すべてのatom stateを提供し、変更されたatomを通知します。ここから、好みのストレージと[シリアライゼーション](http://dic-it.fideli.com/dictionary/m/word/w/11678/index.html)で変更を保存できます。
以下は、JSONを使った基本的な実装の例です。

```React
function PersistenceObserver() {
  useRecoilTransactionObserver_UNSTABLE(({snapshot}) => {
    for (const modifiedAtom of snapshot.getNodes_UNSTABLE({isModified: true})) {
      const atomLoadable = snapshot.getLoadable(modifiedAtom);
      if (atomLoadable.state === 'hasValue') {
        Storage.setItem(
          modifiedAtom.key,
          JSON.stringify({value: atomLoadable.contents}),
        );
      }
    }
  });
}
```

記憶域は、ブラウザのURL履歴、*[LocalStorage](https://developer.mozilla.org/ja/docs/Web/API/Window/localStorage)、[AsyncStorage](https://github.com/react-native-community/react-native-async-storage)*、または任意の記憶域です。

すべてのatomを持続させたくないかもしれませんし、いくつかのatomは異なる持続性を持っているかもしれません。
*atomごとのstate同期APIがまもなく登場するので、検討してみてください。*

## Stateの復元
stateをストレージに保存したら、アプリケーションをロードするときに復元する必要があります。これは、[`<RecoilRoot>`](https://recoiljs.org/docs/api-reference/core/RecoilRoot)コンポーネントの`initializeState`プロパティを使用して実行できます。

`initializeState`は、[MutableSnapshot](https://recoiljs.org/docs/api-reference/core/Snapshot#transforming-snapshots)に初期atom値を設定する`set()`メソッドを提供する関数です。この値は初期レンダリングに使用されます。`useEffect()`hookでatom値を手動で設定すると、最初にデフォルト値を使用した初期レンダリングが行われ、[フリッカー](https://www.sophia-it.com/content/flicker)が発生したり無効になる可能性があります。

次に基本的な例を示します。

```React
const initializeState = ({set}) => {
  for(const [key, value] of Storage.entries()) {
    set(myLookupOfAtomWithKey(key), JSON.parse(value).value);
  }
}

return (
  <RecoilRoot initializeState={initializeState}>
    <PersistenceObserver />
    <SomeComponent />
  </RecoilRoot>
);
```

*注：`myLookupOfAtomWithKey()`は、keyに基づいて登録されたatomを検索するための擬似コードです。一部のatomは動的に、またはatom族を介して定義されるので、これは最良のアプローチではありません。atomごとの同期効果を定義する新しいAPIが近く公開される予定なので、より使いやすくなるはずです。*

## 同期中 State
また、ユーザーがURL永続性を使用してブラウザーの戻るボタンを押すなど、ストレージの非同期更新を現在のアプリケーションのstateと同期させることもできます。React effectを使用すると、これらの変更を適用し、変更されたatomの値を直接更新できます。

サンプルは近日公開予定...

## 下位互換性と値の検証
state記憶装置が信頼できない場合はどうすればよいでしょうか。あるいは、使用しているatomまたはタイプを変更し、以前に永続化されたstateを操作する必要がある場合はどうすればよいでしょうか。
APIが完成し次第、これを処理する方法についてのドキュメントと例を追加予定...
