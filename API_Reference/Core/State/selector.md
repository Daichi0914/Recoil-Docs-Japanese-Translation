# `selector(options)`

selectorは、Recoil内の関数または**派生state**を表します。​特定の依存関係値のセットに対して常に同じ値を返す副作用のない「純粋関数」と考えることができます。​get関数のみが指定されている場合、selectorは読み込み専用で、`RecoilValueReadOnly`オブジェクトを返します。​セットも指定されている場合は、書き込み可能な`RecoilState`オブジェクトを返します。

***

```React
function selector<T>({
  key: string,

  get: ({
    get: GetRecoilValue
  }) => T | Promise<T> | RecoilValue<T>,

  set?: (
    {
      get: GetRecoilValue,
      set: SetRecoilState,
      reset: ResetRecoilState,
    },
    newValue: T | DefaultValue,
  ) => void,

  dangerouslyAllowMutability?: boolean,
})
```

```React
type ValueOrUpdater<T> = T | DefaultValue | ((prevValue: T) => T | DefaultValue);
type GetRecoilValue = <T>(RecoilValue<T>) => T;
type SetRecoilState = <T>(RecoilState<T>, ValueOrUpdater<T>) => void;
type ResetRecoilState = <T>(RecoilState<T>) => void;
```

​
* `key` - atomを内部的に識別するために使用される一意の文字列。​この文字列は、アプリケーション全体で他のatomおよびselectorに対して一意である必要があります。​永続性のために使用する場合は、実行間で安定している必要があります。
* `get` - 派生stateの値を評価する関数。​直接値を返すこともあれば、非同期`Promise`を返すこともありますし、同じ型を表す別のatomやselectorを返すこともあります。​次のプロパティを含む最初のパラメータとしてオブジェクトが渡されます。
  * `get` - 他のatom/selectorから値を取得するために使用する関数。​この関数に渡されるすべてのatom/selectorは、selectorの**依存関係**のリストに暗黙的に追加されます。​selectorの依存関係のいずれかが変更されると、selectorは再評価されます。
* `set?​` - このプロパティが設定されている場合、selectorは**書き込み可能**なstateを返します。​コールバックのオブジェクトを最初のパラメータおよび新しい入力値として渡される関数。​入力値は、`T`型の値か、ユーザーがselectorをリセットした場合は`DefaultValue`型のオブジェクトです。​コールバックには次のものがあります。
  * `get` - 他のatom/selectorから値を取得するために使用する関数。​この関数は、指定されたatom/selectorにselectorを登録しません。
  * `set` - 上流の Recoil state の値を設定する関数。​最初のパラメータは Recoil state で、2番目のパラメータは新しい値です。​新しい値は、リセットアクションを伝達するupdater関数または`DefaultValue`オブジェクトです。
* `dangerouslyAllowMutability` - selectorは派生stateの「純粋関数」を表し、同じ依存性入力値のセットに対して常に同じ値を返すべきです。​これを保護するために、selectorに格納されているすべての値はデフォルトでフリーズされます。​場合によっては、このオプションを使用してオーバーライドする必要があります。

***

単純な静的依存関係を持つselector:

```React
const mySelector = selector({
  key: 'MySelector',
  get: ({get}) => get(myAtom) * 100,
});
```

## Dynamic Dependencies (動的依存関係)

​読み取り専用selectorには、依存関係に基づいてselectorの値を評価する`get`メソッドがあります。​これらの依存関係のいずれかが更新されると、selectorが再評価されます。​依存関係は、selectorを評価するときに実際に使用するatomまたはselectorに基づいて動的に決定されます。​以前の依存関係の値に応じて、異なる追加の依存関係を動的に使用できます。​Recoilは、現在のデータフローグラフを自動的に更新し、selectorが現在の依存関係のセットからの更新のみに登録されるようにします。

​この例では、`mySelector`は`toggleState`の状態に応じて`selectorA`または`selectorB`と`toggleState`のatomに依存します。

```React
const toggleState = atom({key: 'Toggle', default: false});

const mySelector = selector({
  key: 'MySelector',
  get: ({get}) => {
    const toggle = get(toggleState);
    if (toggle) {
      return get(selectorA);
    } else {
      return get(selectorB);
    }
  },
});
```

## Writeable Selectors (書き込み可能なselector)
​双方向selectorは、入力値をパラメータとして受け取り、それを使用して、データフローグラフに沿って上流に変更を伝播することができます。​ユーザーはselectorに新しい値を設定することも、selectorをリセットすることもできるため、入力値は、selectorが表すのと同じ型、またはリセットアクションを表す`DefaultValue`オブジェクトのいずれかになります。

​この単純なselectorは、基本的にatomをラップしてフィールドを追加します。​set操作とreset操作を単に上流のatomに渡すだけです。

```React
const proxySelector = selector({
  key: 'ProxySelector',
  get: ({get}) => ({...get(myAtom), extraField: 'hi'}),
  set: ({set}, newValue) => set(myAtom, newValue),
});
```

​このselectorはデータを変換するため、入力値が`DefaultValue`かどうかを確認する必要があります。

```React
const transformSelector = selector({
  key: 'TransformSelector',
  get: ({get}) => get(myAtom) * 100,
  set: ({set}, newValue) =>
    set(myAtom, newValue instanceof DefaultValue ? newValue : newValue / 100),
});
```

## Asynchronous Selectors (非同期selector)
​selectorは非同期評価関数を持つこともでき、出力値に`Promise`を返します。​詳細については、[このガイド](https://qiita.com/Daichi44/items/47e9ac34c61c35531abb)を参照してください。

```React
const myQuery = selector({
  key: 'MyQuery',
  get: async ({get}) => {
    return await myAsyncQuery(get(queryParamState));
  }
});
```

## Example (Synchronous) (同期処理の例)

```React
import {atom, selector, useRecoilState, DefaultValue} from 'recoil';

const tempFahrenheit = atom({
  key: 'tempFahrenheit',
  default: 32,
});

const tempCelcius = selector({
  key: 'tempCelcius',
  get: ({get}) => ((get(tempFahrenheit) - 32) * 5) / 9,
  set: ({set}, newValue) =>
    set(
      tempFahrenheit,
      newValue instanceof DefaultValue ? newValue : (newValue * 9) / 5 + 32
    ),
});

function TempCelcius() {
  const [tempF, setTempF] = useRecoilState(tempFahrenheit);
  const [tempC, setTempC] = useRecoilState(tempCelcius);
  const resetTemp = useResetRecoilState(tempCelcius);

  const addTenCelcius = () => setTempC(tempC + 10);
  const addTenFahrenheit = () => setTempF(tempF + 10);
  const reset = () => resetTemp();

  return (
    <div>
      Temp (Celcius): {tempC}
      <br />
      Temp (Fahrenheit): {tempF}
      <br />
      <button onClick={addTenCelcius}>Add 10 Celcius</button>
      <br />
      <button onClick={addTenFahrenheit}>Add 10 Fahrenheit</button>
      <br />
      <button onClick={reset}>>Reset</button>
    </div>
  );
}
```

## Example (Asynchronous) (非同期処理の例)

```React
import {selector, useRecoilValue} from 'recoil';

const myQuery = selector({
  key: 'MyDBQuery',
  get: async () => {
    const response = await fetch(getMyRequestUrl());
    return response.json();
  },
});

function QueryResults() {
  const queryResults = useRecoilValue(myQuery);

  return (
    <div>
      {queryResults.foo}
    </div>
  );
}

function ResultsSection() {
  return (
    <React.Suspense fallback={<div>Loading...</div>}>
      <QueryResults />
    </React.Suspense>
  );
}
```

より複雑な例については、[このガイド](https://qiita.com/Daichi44/items/47e9ac34c61c35531abb)を参照してください。