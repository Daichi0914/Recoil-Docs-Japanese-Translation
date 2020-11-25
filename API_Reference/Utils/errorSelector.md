# errorSelector(message)

​提供されたエラーを常にスローする[selector](https://qiita.com/Daichi44/items/13912706763c22f9cbe9)

```React
function errorSelector(message: string): RecoilValueReadOnly
```

## サンプルコード

```React
const myAtom = atom({
  key: 'My Atom',
  default: errorSelector('Attempt to use Atom before initialization'),
});
```