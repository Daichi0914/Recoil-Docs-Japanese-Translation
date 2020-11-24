# useRecoilBridgeAcrossReactRoots()

このhookは、ネストされたReact rootと[レンダラー](http://e-words.jp/w/%E3%83%AC%E3%83%B3%E3%83%80%E3%83%AA%E3%83%B3%E3%82%B0%E3%82%A8%E3%83%B3%E3%82%B8%E3%83%B3.html)でRecoil stateを[ブリッジ](http://e-words.jp/w/%E3%83%96%E3%83%AA%E3%83%83%E3%82%B8.html)するのに役立ちます。

```React
function useRecoilBridgeAcrossReactRoots_UNSTABLE():
  React.AbstractComponent<{children: React.Node}>;
```

ネストされたReact rootが`ReactDOM.render()`で作成された場合、またはネストされたカスタムレンダラーが使用された場合、Reactはコンテキストのstateを子ルートに伝播しません。​このhookは、"bridge"とReact rootがネストされたRecoil stateを共有する場合に便利です。​hookはReactコンポーネントを返します。このコンポーネントは、ネストされたReact rootの代わりに使用して、同じ一貫性のあるRecoil store stateを共有できます。​Reactのルート間でstateを共有する場合と同様に、すべてのケースで変更が完全に同期されるとは限りません。

## サンプルコード

```React
function Bridge() {
  const RecoilBridge = useRecoilBridgeAcrossReactRoots_UNSTABLE();

  return (
    <CustomRenderer>
      <RecoilBridge>
        ...
      </RecoilBridge>
    </CustomRenderer>
  );
}

function MyApp() {
  return (
    <RecoilRoot>
      ...
      <Bridge />
    </RecoilRoot>
  );
}
```