# waitForAll(dependencies)

複数の非同期依存関係を同時に評価することを可能にする並行性ヘルパー。

​依存関係は、タプル配列またはオブジェクト内の名前付き依存関係として指定できます。

***

```React
function waitForAll(dependencies: Array<RecoilValue<>>):
  RecoilValueReadOnly<UnwrappedArray>
```

```React
function waitForAll(dependencies: {[string]: RecoilValue<>}):
  RecoilValueReadOnly<UnwrappedObject>
```

***

​並行性ヘルパーはselectorとして提供されるため、ReactコンポーネントのRecoil hookによって、Recoil selectorの依存関係として、またはRecoil stateが使用されるすべての場所で使用できます。

## サンプルコード

```React
function FriendsInfo() {
  const [friendA, friendB] = useRecoilValue(
    waitForAll([friendAState, friendBState])
  );
  return (
    <div>
      Friend A Name: {friendA.name}
      Friend B Name: {friendB.name}
    </div>
  );
}
```

```React
const friendsInfoQuery = selector({
  key: 'FriendsInfoQuery',
  get: ({get}) => {
    const {friendList} = get(currentUserInfoQuery);
    const friends = get(waitForAll(
      friendList.map(friendID => userInfoQuery(friendID))
    ));
    return friends;
  },
});
```

```React
const customerInfoQuery = selectorFamily({
  key: 'CustomerInfoQuery',
  get: id => ({get}) => {
    const {info, invoices, payments} = get(waitForAll({
      info: customerInfoQuery(id),
      invoices: invoicesQuery(id),
      payments: paymentsQuery(id),
    }));

    return {
      name: info.name,
      transactions: [
        ...invoices,
        ...payments,
      ],
    };
  },
});
```