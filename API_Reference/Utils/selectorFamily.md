# selectorFamily(options)

​読み取り専用の`RecoilValueReadOnly`selectorまたは書き込み可能な`RecoilState`selectorを返す関数を返します。

​`selectorFamily`は[`selector`](https://qiita.com/Daichi44/items/13912706763c22f9cbe9)に似た強力なパターンですが、selectorのgetコールバックとsetコールバックにパラメータを渡すことができます。​`selectorFamily()`ユーティリティは、ユーザ定義パラメータを使用して呼び出すことができる関数を返し、selectorを返します。​一意の各パラメータ値は、同じメモ型selectorインスタンスを返します。

***

```React
function selectorFamily<T, Parameter>({
  key: string,

  get: Parameter => ({get: GetRecoilValue}) => Promise<T> | RecoilValue<T> | T,

  dangerouslyAllowMutability?: boolean,
}): Parameter => RecoilValueReadOnly<T>
```

```React
function selectorFamily<T, Parameter>({
  key: string,

  get: Parameter => ({get: GetRecoilValue}) => Promise<T> | RecoilValue<T> | T,

  set: Parameter => (
    {
      get: GetRecoilValue,
      set: SetRecoilValue,
      reset: ResetRecoilValue,
    },
    newValue: T | DefaultValue,
  ) => void,

  dangerouslyAllowMutability?: boolean,
}): Parameter => RecoilState<T>
```

ここで

```React
type ValueOrUpdater<T> =  T | DefaultValue | ((prevValue: T) => T | DefaultValue);
type GetRecoilValue = <T>(RecoilValue<T>) => T;
type SetRecoilValue = <T>(RecoilState<T>, ValueOrUpdater<T>) => void;
type ResetRecoilValue = <T>(RecoilState<T>) => void;
```

* key - atomを内部的に識別するために使用される一意の文字列。​この文字列は、アプリケーション全体で他のatomおよびselectorに対して一意である必要があります。
* get - selectorの値を返す名前付きコールバックのオブジェクトに渡される関数で、`selector()`インタフェースと同じです。​これは、selectorファミリ関数を呼び出すことによってパラメータが渡される関数によってラップされます。
* set? ​- 書き込み可能なselectorを生成するオプションの関数。​これは、`selector()`インタフェースと同じ名前付きコールバックのオブジェクトを取る関数である必要があります。​これも、selector family関数を呼び出してパラメータを取得する別の関数によってラップされます。

***

`selectorFamily`は基本的に、パラメータからselectorへのmapを提供します。​パラメータはファミリを使用してコールサイトで生成されることが多く、同じ基礎となるselectorを同等のパラメータで再利用する必要があるため、デフォルトでは参照等価ではなく値等価が使用されます。​(この動作を調整するための不安定な`cacheImplementationForParams` APIがあります)​これにより、パラメータに使用できるタイプが制限されます。​プリミティブ型またはシリアライズ可能なオブジェクトを使用してください。​Recoilは、オブジェクトと配列をサポートできるカスタム・シリアライザーを使用していますが、一部のコンテナー(ES6セットやmapなど)はobject・keyの順序に不変で、SymbolsとIterablesをサポートし、カスタム・シリアライゼーションのためにtoJSONプロパティーを使用します(Immutableコンテナのようなライブラリで提供されているもの)。​パラメータで関数やPromiseなどの可変オブジェクトを使用すると問題が発生します。

## サンプルコード

```React
const myNumberState = atom({
  key: 'MyNumber',
  default: 2,
});

const myMultipliedState = selectorFamily({
  key: 'MyMultipliedNumber',
  get: (multiplier) => ({get}) => {
    return get(myNumberState) * multiplier;
  },

  // optional set
  set: (multiplier) => ({set}, newValue) => {
    set(myNumberState, newValue / multiplier);
  },
});

function MyComponent() {
  // defaults to 2
  const number = useRecoilValue(myNumberState);

  // defaults to 200
  const multipliedNumber = useRecoilValue(myMultipliedState(100));

  return <div>...</div>;
}
```

## 非同期クエリー サンプルコード

​selectorファミリは、パラメータをクエリーに渡す場合にも便利です。​このようにselectorを使って問い合わせを抽象化しても、与えられた入力と依存性の値の集合に対して常に同じ結果を返す"純粋な"関数であることに注意してください。​その他の例については、この[ガイド](https://qiita.com/Daichi44/items/47e9ac34c61c35531abb)を参照してください。

```React
const myDataQuery = selectorFamily({
  key: 'MyDataQuery',
  get: (queryParameters) => async ({get}) => {
    const response = await asyncDataRequest(queryParameters);
    if (response.error) {
      throw response.error;
    }
    return response.data;
  },
});

function MyComponent() {
  const data = useRecoilValue(myDataQuery({userID: 132}));
  return <div>...</div>;
}
```

## Destructuring(分割) サンプルコード

```React
const formState = atom({
  key: 'formState',
  default: {
    field1: "1",
    field2: "2",
    field3: "3",
  },
});

const formFieldState = selectorFamily({
  key: 'FormField',
  get: field => ({get}) => get(formState)[field],
  set: field => ({set}, newValue) =>
    set(formState, prevState => {...prevState, [field]: newValue}),
});

const Component1 = () => {
  const [value, onChange] = useRecoilState(formFieldState('field1'));
  return (
    <>
      <input value={value} onChange={onChange} />
      <Component2 />
    </>
  );
}

const Component2 = () => {
  const [value, onChange] = useRecoilState(formFieldState('field2'));
  return (
    <input value={value} onChange={onChange} />
  );
}
```