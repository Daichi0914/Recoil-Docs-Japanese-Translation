# isRecoilValue(value)

`value`がatomまたはselectorの場合は`true`、それ以外の場合は`false`を返します。

```React
function isRecoilValue(value: mixed): boolean
```

***

## サンプルコード

```React
import {atom, isRecoilValue} from 'recoil';

const counter = atom({
  key: 'myCounter',
  default: 0,
});

const strCounter = selector({
  key: 'myCounterStr',
  get: ({get}) => String(get(counter)),
});

isRecoilValue(counter); // true
isRecoilValue(strCounter); // true

isRecoilValue(5); // false
isRecoilValue({}); // false
```