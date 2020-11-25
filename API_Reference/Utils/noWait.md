# noWait(state)

指定された[`atom`](https://qiita.com/Daichi44/items/0a9b9af69dddddbcd7e2)または[`selector`](https://qiita.com/Daichi44/items/13912706763c22f9cbe9)の現在のstateの[`Loadable`](https://qiita.com/Daichi44/items/812aa5ebf149c849e108)を返すselectorヘルパー。

```React
function noWait<T>(state: RecoilValue<T>): RecoilValueReadOnly<Loadable<T>>
```

***

​このヘルパーを使用すると、エラーが発生した場合や依存関係が保留中の場合に、スローすることなく非同期の可能性のある依存関係の現在のstateを取得できます。​[`useRecoilValueLoadable()`](https://qiita.com/Daichi44/items/679030df40e81a608be5)と似ていますが、hookではなくselectorである点が異なります。​`noWait()`はselectorを返すので、今度はhookだけでなく他のRecoil selectorでも使用できます。

## サンプルコード

```React
const myQuery = selector({
  key: 'MyQuery',
  get: ({get}) => {
    const loadable = get(noWait(dbQuerySelector));

    return {
      hasValue: {data: loadable.contents},
      hasError: {error: loadable.contents},
      loading: {data: 'placeholder while loading'},
    }[loadable.state];
  }
})
```