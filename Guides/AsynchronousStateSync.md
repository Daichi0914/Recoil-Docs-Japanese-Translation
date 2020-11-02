# 非同期state・同期state

***

### *重要なお知らせ*
*このAPIは現在開発中であり、今後変更される予定です。お楽しみに...*

***

Recoil [atoms](https://recoiljs.org/docs/api-reference/core/atom) は局所的な応用stateを表します。
アプリケーションは、RESTful APIなどを介して、リモートまたはサーバー側のstateを持っている場合もあります。
リモートstateをRecoil atomと同期することを検討してください。これにより、`useRecoilState()`hookを使用してReactコンポーネントからstateに簡単にアクセスしたり、そのstateを他の派生state selectorのデータフローグラフへの入力として使用したりできます。[データベースまたはサーバーに読み取り専用データを問い合わせる](https://qiita.com/Daichi44/items/47e9ac34c61c35531abb)場合は、非同期selectorの使用を検討してください。

## ローカルStateの例
この例では、フレンドステータスをローカルstateとしてのみ提供します。

```React
const currentUserIDState = atom({
  key: 'CurrentUserID',
  default: null,
});

function CurrentUserInfo() {
  const [currentUserID] = useRecoilState(currentUserIDState);
  return <div>Current User: {currentUserID}</div>;
}
```

## サーバからstateを同期
リモートstateの非同期変更を登録し、それに合わせてatom値を更新することができます。これは、通常 React [useEffect()](https://reactjs.org/docs/hooks-reference.html#useeffect) hookまたはその他の一般的なライブラリを使用して実行できます。

```React
function CurrentUserIDSubscription() {
  const setCurrentUserID = useSetRecoilState(currentUserIDState);

  useEffect(() => {
    RemoteStateAPI.subscribeToCurrentUserID(setCurrentUserID);
    // Specify how to cleanup after this effect
    return function cleanup() {
      RemoteServerAPI.unsubscribeFromCurrentUserID(setCurrentUserID);
    };
  }, []);

  return null;
}

function MyApp() {
  return (
    <RecoilRoot>
      <CurrentUserIDSubscription />
      <CurrentUserInfo />
    </RecoilRoot>
  );
}
```

1つの場所で複数のatomの同期を処理する場合は、[State Persistence](https://recoiljs.org/docs/guides/persistence)パターンも使用できます。

## 双方向同期
stateを同期して、ローカルの変更がサーバー上で更新されるようにすることもできます。
これは単純化した例です。フィードバックのループを避けるように注意してください。

```React
function CurrentUserIDSubscription() {
  const [currentUserID, setCurrentUserID] = useRecoilState(currentUserIDState);
  const knownServerCurrentUserID = useRef(currentUserID);

  // Subscribe server changes to update atom state
  useEffect(() => {
    function handleUserChange(id) {
      knownServerCurrentUserID.current = id;
      setCurrentUserID(id);
    }

    RemoteStateAPI.subscribeToCurrentUserID(handleUserChange);
    // Specify how to cleanup after this effect
    return function cleanup() {
      RemoteServerAPI.unsubscribeFromCurrentUserID(handleUserChange);
    };
  }, [knownServerCurrentUserID]);

  // Subscribe atom changes to update server state
  useEffect(() => {
    if (currentUserID !== knownServerCurrentUserID.current) {
      knownServerCurrentID.current = currentUserID;
      RemoteServerAPI.updateCurrentUser(currentUserID);
    }
  }, [currentUserID, knownServerCurrentUserID.current]);

  return null;
}
```

## パラメータを使用したstateの同期
[atomFamily](https://recoiljs.org/docs/api-reference/utils/atomFamily)ヘルパーを使用して、パラメータに基づいてローカルstateを同期することもできます。
このhookの例の呼び出しごとにサブスクリプションが作成されるため、冗長な使用を避けるように注意してください。

```React
const friendStatusState = atomFamily({
  key: 'Friend Status',
  default: 'offline',
});

function useFriendStatusSubscription(id) {
  const setStatus = useSetRecoilState(friendStatusState(id));

  useEffect(() => {
    RemoteStateAPI.subscribeToFriendStatus(id, setStatus);
    // Specify how to cleanup after this effect
    return function cleanup() {
      RemoteServerAPI.unsubscribeFromFriendStatus(id, setStatus);
    };
  }, []);
}
```

# データフローグラフ
リモートstateを表すためにatomを使用する利点は、他の派生stateの入力として使用できることです。
次の例では、現在のサーバのstateに基づいて、現在のユーザと友人のリストを表示します。サーバが現在のユーザを変更すると、リスト全体が再描画されます。友人のステータスだけを変更すると、そのリストエントリだけが再描画されます。リスト項目をクリックすると、現在のユーザがローカルに変更され、サーバのstateが更新されます。

```React
const userInfoQuery = selectorFamily({
  key: 'UserInfoQuery',
  get: userID => async ({get}) => {
    const response = await myDBQuery({userID});
    if (response.error) {
      throw response.error;
    }
    return response;
  },
});

const currentUserInfoQuery = selector({
  key: 'CurrentUserInfoQuery',
  get: ({get}) => get(userInfoQuery(get(currentUserIDState)),
});

const friendColorState = selectorFamily({
  key: 'FriendColor',
  get: friendID => ({get}) => {
    const [status] = get(friendStatusState(friendID));
    return status === 'offline' ? 'red' : 'green';
  },
});

function FriendStatus({friendID}) {
  useFriendStatusSubscription(friendID);
  const [status] = useRecoilState(friendStatusState(friendID));
  const [color] = useRecoilState(friendColorState(friendID));
  const [friend] = useRecoilState(userInfoQuery(friendID));
  return (
    <div style={{color}}>
      Name: {friend.name}
      Status: {status}
    </div>
  );
}

function CurrentUserInfo() {
  const {name, friendList} = useRecoilValue(currentUserInfoQuery);
  const setCurrentUserID = useSetRecoilState(currentUserIDState);
  return (
    <div>
      <h1>{name}</h1>
      <ul>
        {friendList.map(friendID =>
          <li key={friend.id} onClick={() => setCurrentUserID(friend.id)}>
            <React.Suspense fallback={<div>Loading...</div>}>
              <FriendStatus friendID={friendID} />
            </React.Suspense>
          </li>
        )}
      </ul>
    </div>
  );
}

function MyApp() {
  return (
    <RecoilRoot>
      <ErrorBoundary>
        <React.Suspense fallback={<div>Loading...</div>}>
          <CurrentUserIDSubscription />
          <CurrentUserInfo />
        </React.Suspense>
      </ErrorBoundary>
    </RecoilRoot>
  );
}
```