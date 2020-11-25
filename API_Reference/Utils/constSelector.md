# constSelector(constant)

​常に一定の値を提供するselector

```React
function constSelector<T: Parameter>(constant: T): RecoilValueReadOnly<T>
```

`constSelector`は、異なるselector実装によって提供される`RecoilValue<T>`や`RecoilValueReadOnly<T>`などのタイプを使用するインタフェースがある場合に便利です。

​これらのselectorは、参照値の等価性に基づいて記憶します。​したがって、`constSelector()`を同じ値で複数回呼び出すことができ、同じselectorが提供されます。​このため、定数として使用される値は、Recoil serializationを使用して[シリアライズ](http://e-words.jp/w/%E3%82%B7%E3%83%AA%E3%82%A2%E3%83%A9%E3%82%A4%E3%82%BA.html)できる型に制限されます。​制限については、[`selectorFamily`](https://recoiljs.org/docs/api-reference/utils/selectorFamily)のドキュメントを参照してください。

## サンプルコード

```React
type MyInterface = {
  queryForStuff: RecoilValue<Thing>,
  ...
};

const myInterfaceInstance1: MyInterface = {
  queryForStuff: selectorThatDoesQuery,
};

const myInterfaceInstance2: MyInterface = {
  queryForStuff: constSelector(thing),
};
```