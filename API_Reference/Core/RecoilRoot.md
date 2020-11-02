# \<RecoilRoot ...props /\>

atomが値を持つコンテキストを提供します。
Recoil hooksを使用するコンポーネントの親である必要があります。
atomはそれぞれのroot内で異なる値を持ちます(複数のrootが共存することもある)。ネストされている場合は、最も内側のrootが外側のrootを完全に隠します。

props:

* `initializeState?`: `(MutableSnapshot => void)`
  * \<RecoilRoot>atomのstateを初期化するために[`MutableSnapshot`](https://recoiljs.org/docs/api-reference/core/Snapshot#transforming-snapshots)を取るオプションの関数。これにより、初期レンダリングのstateが設定され、以降のstate変更や非同期の初期化には使用されません。非同期stateの変更には、[`useSetRecoilState()`](https://recoiljs.org/docs/api-reference/core/useSetRecoilState)や[`useRecoilCallback()`](https://recoiljs.org/docs/api-reference/core/useRecoilCallback)などのhookを使用します。

## サンプルコード

```React
import {RecoilRoot} from 'recoil';

function AppRoot() {
  return (
    <RecoilRoot>
      <ComponentThatUsesRecoil />
    </RecoilRoot>
  );
}
```