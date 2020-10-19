# コアコンセプト
## 概要
Recoilを使用すると、atom(共有state)からselector(純粋関数)を通ってReactコンポーネントに流れるデータフローグラフを作成できます。
atomは、コンポーネントが登録できるstateの単位です。selectorは、このstateを同期的または非同期的に変換します。

## Atoms
atomはstateの単位です。
atomが更新されると、登録された各コンポーネントが新しい値で再描画されます。実行時にも作成できます。[React local component state] の代わりにatomを使用できます。同じatomが複数のコンポーネントから使用されている場合、それらのコンポーネントはすべてstateを共有します。

atomは、atom関数を使用して作成されます。

```React
const fontSizeState = atom({
  key: 'fontSizeState',
  default: 14,
});
```

atomには一意のkeyが必要で、これはデバッグ、永続化、およびすべてのatomのマップを表示できる特定の高度なAPIに使用されます。
2つのatomが同じkeyを持つとエラーになるで、それらがグローバルに一意であることを確認してください。コンポーネントのstateと同様に、既定値もあります。

コンポーネントからatomを読み書きするには、`useRecoilState`というHookを使用します。Reactの`useState`に似ているが、コンポーネント間でstateを共有できるようになります。

```React
function FontButton() {
  const [fontSize, setFontSize] = useRecoilState(fontSizeState);
  return (
    <button onClick={() => setFontSize((size) => size + 1)} style={{fontSize}}>
      Click to Enlarge
    </button>
  );
}
```

ボタンをクリックすると、ボタンのフォントサイズが1つ大きくなります。
他のコンポーネントでも同じフォントサイズを使用できるようになりました。

```React
function Text() {
  const [fontSize, setFontSize] = useRecoilState(fontSizeState);
  return <p style={{fontSize}}>This text will increase in size too.</p>;
}
```

## Selectors
selectorは、atomやその他のselectorを入力として受け付ける純粋な関数です。
これらの上流のatomまたはselectorが更新されると、selector関数が再評価されます。コンポーネントはatomと同じようにselectorに登録でき、selectorが変更されると再レンダリングされます。

selectorは、stateに基づく派生データを計算するために使用されます。これにより冗長なstateを避けることができ、通常はreducerがstateを同期させて有効に保つ必要がなくなります。その代わりに、stateの最小セットはatomに格納され、それ以外はすべて、その最小stateの関数として効率的に計算されます。selectorは、どのコンポーネントが必要とし、どのstateに依存しているかを追跡するので、この機能的なアプローチがより効率的になります。

構成要素の観点から見ると、selectorとatomは同じ境界面を有するため、互いに置換することができます。

selectorは、selector関数を使用して定義します。

```React
const fontSizeLabelState = selector({
  key: 'fontSizeLabelState',
  get: ({get}) => {
    const fontSize = get(fontSizeState);
    const unit = 'px';

    return `${fontSize}${unit}`;
  },
});
```

getプロパティは、計算される関数です。渡されたget引数を使用して、atomやその他のselectorの値にアクセスできます。
別のatomまたはselectorにアクセスすると、依存関係が作成され、別のatomまたはselectorを更新すると、このatomまたはselectorが再計算されます。

この`fontSizeLabelState`の例では、selectorには`fontSizeState` atomという1つの依存関係があります。
概念的には、`fontSizeLabelState` selectorは`fontSizeState`を入力として受け取り、フォーマットされたフォントサイズラベルを出力として返す純粋な関数のように動作します。

selectorは、`useRecoilValue()`を使用して読み取ることができます。
これは、atomまたはselectorを引数として取り、対応する値を返します。`fontSizeLabelState` selectorが書き込み可能ではない(書き込み可能selectorの詳細については、selectorAPIリファレンスを参照してください。)ため、`useRecoilState()`は使用しません。

```React
function FontButton() {
  const [fontSize, setFontSize] = useRecoilState(fontSizeState);
  const fontSizeLabel = useRecoilValue(fontSizeLabelState);

  return (
    <>
      <div>Current font size: ${fontSizeLabel}</div>

      <button onClick={() => setFontSize(fontSize + 1)} style={{fontSize}}>
        Click to Enlarge
      </button>
    </>
  );
}
```

ボタンをクリックすると、ボタンのフォントサイズが大きくなり、現在のフォントサイズを反映するようにフォントサイズのラベルが更新されます。