# useResetRecoilState(state)

​指定されたstateの値を既定値にリセットする関数を返します。

​`useResetRecoilState()`を使用すると、コンポーネントは、stateが変更されるたびに再レンダリングするようにコンポーネントを登録することなく、stateをデフォルト値にリセットできます。

***

```React
function useResetRecoilState<T>(state: RecoilState<T>): () => void;
```

* `state`: 書き込み可能な Recoil state

## サンプルコード

```React
import {todoListState} from "../atoms/todoListState";

const TodoResetButton = () => {
  const resetList = useResetRecoilState(todoListState);
  return <button onClick={resetList}>Reset</button>;
};
```