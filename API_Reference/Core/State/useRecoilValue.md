# useRecoilValue(state)

​指定されたRecoil stateの値を返します。

​このhookは、コンポーネントを指定されたstateに暗黙的に登録します。

***

```React
function useRecoilValue<T>(state: RecoilValue<T>): T;
```

* `​state`: [`atom`](https://qiita.com/Daichi44/items/0a9b9af69dddddbcd7e2)または[`selector`](https://qiita.com/Daichi44/items/13912706763c22f9cbe9)

***

​このhookは**読み取り専用state**と**書き込み可能state**の両方で動作するため、コンポーネントが書き込みを行わずにstateを読み取る場合に使用することをお勧めします。​atomは書き込み可能なstateですが、セレクタは読み取り専用または書き込み可能です。​詳細は[`selector()`](https://qiita.com/Daichi44/items/13912706763c22f9cbe9)を参照ください。

​Reactコンポーネントでこのhookを使用すると、stateが更新されたときに再レンダリングするようにコンポーネントが登録されます。​このhookは、stateにエラーがあるか、非同期解決が保留中の場合にスローされます。​[このガイド](https://qiita.com/Daichi44/items/47e9ac34c61c35531abb)をご覧ください。

## サンプルコード

```React
import {atom, selector, useRecoilValue} from 'recoil';

const namesState = atom({
  key: 'namesState',
  default: ['', 'Ella', 'Chris', '', 'Paul'],
});

const filteredNamesState = selector({
  key: 'filteredNamesState',
  get: ({get}) => get(namesState).filter((str) => str !== ''),
});

function NameDisplay() {
  const names = useRecoilValue(namesState);
  const filteredNames = useRecoilValue(filteredNamesState);

  return (
    <>
      Original names: {names.join(',')}
      <br />
      Filtered names: {filteredNames.join(',')}
    </>
  );
}
```