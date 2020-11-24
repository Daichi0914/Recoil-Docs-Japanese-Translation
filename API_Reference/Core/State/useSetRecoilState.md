# useSetRecoilState(state)

​書き込み可能なRecoil stateの値を更新するセッター関数を返します。

***

```React
function useSetRecoilState<T>(state: RecoilState<T>): SetterOrUpdater<T>;

type SetterOrUpdater<T> = (T | (T => T)) => void;
```

* state: 書き込み可能なRecoil state([`atom`](https://qiita.com/Daichi44/items/0a9b9af69dddddbcd7e2)または書き込み可能な[`selector`](https://qiita.com/Daichi44/items/13912706763c22f9cbe9))

​非同期に使用してstateを変更できるセッター関数を返します。​セッターには新しい値が渡されるか、前の値を引数として受け取るアップデーター関数が渡されます。

***

​これは、コンポーネントが読み取りを行わずにstateに書き込む場合に推奨されるhookです。​コンポーネントが[`useRecoilState()`](https://qiita.com/Daichi44/items/2ec591ec3e952f1784c3) hookを使用してセッターを取得した場合、コンポーネントは更新を登録し、atomまたはselectorが更新されたときに再レンダリングを行います。​`useSetRecoilState()`を使用すると、コンポーネントを登録せずに値を設定し、値が変更されたときに再レンダリングできます。

## サンプルコード

```React
import {atom, useSetRecoilState} from 'recoil';

const namesState = atom({
  key: 'namesState',
  default: ['Ella', 'Chris', 'Paul'],
});

function FormContent({setNamesState}) {
  const [name, setName] = useState('');

  return (
    <>
      <input type="text" value={name} onChange={(e) => setName(e.target.value)} />
      <button onClick={() => setNamesState(names => [...names, name])}>Add Name</button>
    </>
)}

// このコンポーネントはマウント時に1回レンダリングされます
function Form() {
  const setNamesState = useSetRecoilState(namesState);

  return <FormContent setNamesState={setNamesState} />;
}
```