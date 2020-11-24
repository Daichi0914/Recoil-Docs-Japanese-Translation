# class Loadable
​`Loadable`オブジェクトは、Recoil atomまたはselectorの現在のstateを表します。​このstateは、値が使用可能であるか、エラーstateであるか、または非同期の解決を保留している可能性があります。​`Loadable`には次のインタフェースがあります。

* `​state`: atomまたはselectorの現在のstate。​指定できる値は、`'hasValue'`(値あり), `'hasError'`(エラーあり), または`'loading'`(ロード中)です。
* `contents`: `Loadable`によって表される値。​stateが`hasValue`の場合は実際の値、stateが`hasError`の場合はスローされた`Error`オブジェクト、そしてstateが`loading`の場合は`toPromise()`を使って値の`Promise`を取得することができます。

​ロード可能なオブジェクトには、現在のstateにアクセスするためのヘルパーメソッドも含まれています。
​*次のAPIが不安定であるとします:*

* `getValue()` - React SuspenseおよびRecoil selectorの[セマンティック](https://developer.mozilla.org/ja/docs/Glossary/Semantics)に一致する値にアクセスするメソッド。​stateに値がある場合は値を返し、エラーがある場合はそのエラーをスローし、まだ保留中の場合は実行またはレンダリングを中断して保留中のstateを伝播します。
* `toPromise()` - selectorが解決したときに解決するPromiseを返します。​selectorが同期しているか、既に解決されている場合、即座に解決されるPromiseを返します。
* `valueMaybe()` - 使用可能な場合は値を返し、使用できない場合はundefinedを返します。
* `valueOrThrow()` - 使用可能な場合は値を返し、使用できない場合はエラーをスローします。
* ​`map()` - Loadableの値を変換する関数を取り、変換された値で新しいLoadableを返します。​変換関数は値のパラメータを取得し、新しい値を返します。​スローされたエラーやアラートを伝播することもできます。

## サンプルコード

```React
function UserInfo({userID}) {
  const userNameLoadable = useRecoilValueLoadable(userNameQuery(userID));
  switch (userNameLoadable.state) {
    case 'hasValue':
      return <div>{userNameLoadable.contents}</div>;
    case 'loading':
      return <div>Loading...</div>;
    case 'hasError':
      throw userNameLoadable.contents;
  }
}
```