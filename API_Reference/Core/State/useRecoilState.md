# `useRecoilState(state)`

最初の要素がstateの値であり、2番目の要素が呼び出されたときに指定されたstateの値を更新するセッター関数であるタプル(組)を返します。

​このhookは、指定されたstateにコンポーネントを暗黙的に登録します。

***

```React
function useRecoilState<T>(state: RecoilState<T>): [T, SetterOrUpdater<T>];

type SetterOrUpdater<T> = (T | (T => T)) => void;
```

* `state`: [atom](https://qiita.com/Daichi44/items/0a9b9af69dddddbcd7e2)または書き込み可能な[selector](https://qiita.com/Daichi44/items/13912706763c22f9cbe9)。​書き込み可能なselectorは、定義に`get`と`set`の両方が含まれるselectorで、読み取り専用のselectorは`get`のみが含まれます。

​このAPIはReact [useState()](https://reactjs.org/docs/hooks-reference.html#usestate)hookに似ていますが、引数としてデフォルト値の代わりにRecoil stateを取る点が異なります。​この関数は、stateの現在の値とセッター関数のタプルを返します。​セッター関数は、引数として新しい値を取るか、前の値をパラメーターとして受け取るアップデーター関数を取ることができます。

***

これは、コンポーネントがstateを読み書きしようとするときに使用することを推奨するhookです。

​Reactコンポーネントでこのhookを使用すると、stateが更新されたときに再レンダリングするようにコンポーネントが登録されます。​このhookは、stateにエラーがあるか、非同期解決が保留中の場合にスローされます。​[このガイド](https://qiita.com/Daichi44/items/47e9ac34c61c35531abb)をご覧ください。

## サンプルコード

```React
import {atom, selector, useRecoilState} from 'recoil';

const tempFahrenheit = atom({
  key: 'tempFahrenheit',
  default: 32,
});

const tempCelcius = selector({
  key: 'tempCelcius',
  get: ({get}) => ((get(tempFahrenheit) - 32) * 5) / 9,
  set: ({set}, newValue) => set(tempFahrenheit, (newValue * 9) / 5 + 32),
});

function TempCelcius() {
  const [tempF, setTempF] = useRecoilState(tempFahrenheit);
  const [tempC, setTempC] = useRecoilState(tempCelcius);

  const addTenCelcius = () => setTempC(tempC + 10);
  const addTenFahrenheit = () => setTempF(tempF + 10);

  return (
    <div>
      Temp (Celcius): {tempC}
      <br />
      Temp (Fahrenheit): {tempF}
      <br />
      <button onClick={addTenCelcius}>Add 10 Celcius</button>
      <br />
      <button onClick={addTenFahrenheit}>Add 10 Fahrenheit</button>
    </div>
  );
}
```