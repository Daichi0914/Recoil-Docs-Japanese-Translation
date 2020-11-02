# 非同期データクエリ
Recoilは、stateと派生stateをデータフローグラフを介してReactコンポーネントにマップする方法を提供します。非常に強力なのは、グラフ中の関数が非同期であることです。これにより、同期Reactコンポーネントのレンダリング関数で非同期関数を簡単に使用できるようになります。Recoilを使用すると、selectorのデータフローグラフに同期関数と非同期関数を[シームレス](http://e-words.jp/w/%E3%82%B7%E3%83%BC%E3%83%A0%E3%83%AC%E3%82%B9.html)に混在させることができます。selectorのgetコールバックから値自体ではなくPromiseを値に返すだけで、インタフェースはまったく同じままです。これらは単なるselectorなので、他のselectorもデータをさらに変換するためにこれらに依存することができます。

selectorは、Recoilデータフローグラフに非同期データを組み込む1つの方法として使用できます。selectorは純粋な関数を表していることに注意してください。与えられた入力の集合に対して、selectorは常に同じ結果を生成します(少なくともアプリケーションの存続期間中は)。selector評価は1回以上実行され、再起動され、キャッシュされる可能性があるため、これは重要です。このため、selectorは、クエリを繰り返すことで一貫したデータが得られる読み取り専用のDBクエリをモデル化するのに適しています。ローカルとサーバーのstateを同期させる場合は、[非同期状態・同期状態]()または[Stateの永続性]()を参照してください。


## 同期の例
ユーザー名を取得するための単純な同期[atom](https://recoiljs.org/docs/api-reference/core/atom)と[selector](https://recoiljs.org/docs/api-reference/core/selector)を次に示します。

```React
const currentUserIDState = atom({
  key: 'CurrentUserID',
  default: 1,
});

const currentUserNameState = selector({
  key: 'CurrentUserName',
  get: ({get}) => {
    return tableOfUsers[get(currentUserIDState)].name;
  },
});

function CurrentUserInfo() {
  const userName = useRecoilValue(currentUserNameState);
  return <div>{userName}</div>;
}

function MyApp() {
  return (
    <RecoilRoot>
      <CurrentUserInfo />
    </RecoilRoot>
  );
}
```

## 非同期の例
ユーザ名が何らかのデータベースに格納されている場合は、Promiseを返すか、非同期関数を使用するだけで済みます。依存関係が変更されると、selectorが再評価され、新しいクエリが実行されます。結果はキャッシュされるため、クエリは一意の入力ごとに1回だけ実行されます。

```React
const currentUserNameQuery = selector({
  key: 'CurrentUserName',
  get: async ({get}) => {
    const response = await myDBQuery({
      userID: get(currentUserIDState),
    });
    return response.name;
  },
});

function CurrentUserInfo() {
  const userName = useRecoilValue(currentUserNameQuery);
  return <div>{userName}</div>;
}
```

selectorのインターフェースは同じなので、このselectorを使用するコンポーネントは、それが同期atom state、派生selector state、または非同期クエリでバックされたかどうかを気にする必要はありません！

しかし、Reactのレンダリング機能は同期であるため、約束が解決する前に何をレンダリングするのだろうか？
Recoilは、[React Suspense](https://reactjs.org/docs/concurrent-mode-suspense.html)と連動して保留中のデータを処理するように設計されている。Suspense境界でコンポーネントをラップすると、保留中のすべての子孫がキャッチされ、フォールバックUIがレンダリングされます。

```React
function MyApp() {
  return (
    <RecoilRoot>
      <React.Suspense fallback={<div>Loading...</div>}>
        <CurrentUserInfo />
      </React.Suspense>
    </RecoilRoot>
  );
}
```

## エラー処理
しかし、要求にエラーがある場合はどうなるでしょうか。
Recoilのselectorは、コンポーネントがその値を使おうとした場合に投げられるエラーを投げることができます。これはReact[`<ErrorBoundary>`](https://reactjs.org/docs/error-boundaries.html)でキャッチできます。たとえば、次のようになります。

```React
const currentUserNameQuery = selector({
  key: 'CurrentUserName',
  get: async ({get}) => {
    const response = await myDBQuery({
      userID: get(currentUserIDState),
    });
    if (response.error) {
      throw response.error;
    }
    return response.name;
  },
});

function CurrentUserInfo() {
  const userName = useRecoilValue(currentUserNameQuery);
  return <div>{userName}</div>;
}

function MyApp() {
  return (
    <RecoilRoot>
      <ErrorBoundary>
        <React.Suspense fallback={<div>Loading...</div>}>
          <CurrentUserInfo />
        </React.Suspense>
      </ErrorBoundary>
    </RecoilRoot>
  );
}
```

## パラメータを使用したクエリ
派生stateだけでなく、パラメータに基づいてクエリを実行できるようにしたい場合があります。たとえば、コンポーネントプロパティに基づいてクエリを実行できます。これを行うには、[selectorFamily](https://recoiljs.org/docs/api-reference/utils/selectorFamily)ヘルパーを使用します。

```React
const userNameQuery = selectorFamily({
  key: 'UserName',
  get: userID => async () => {
    const response = await myDBQuery({userID});
    if (response.error) {
      throw response.error;
    }
    return response.name;
  },
});

function UserInfo({userID}) {
  const userName = useRecoilValue(userNameQuery(userID));
  return <div>{userName}</div>;
}

function MyApp() {
  return (
    <RecoilRoot>
      <ErrorBoundary>
        <React.Suspense fallback={<div>Loading...</div>}>
          <UserInfo userID={1}/>
          <UserInfo userID={2}/>
          <UserInfo userID={3}/>
        </React.Suspense>
      </ErrorBoundary>
    </RecoilRoot>
  );
}
```

## データフローグラフ
クエリをselectorとしてモデリングすることによって、データ・フロー・グラフの混合state、派生state、クエリを作成できることを忘れないでください。stateが更新されると、このグラフはReactコンポーネントを自動的に更新して再レンダリングします。

次の例は、現在のユーザの名前と友人のリストを表示します。友達の名前をクリックすると、その友達が現在のユーザーになり、名前とリストが自動的に更新される。

```React
const currentUserIDState = atom({
  key: 'CurrentUserID',
  default: null,
});

const userInfoQuery = selectorFamily({
  key: 'UserInfoQuery',
  get: userID => async () => {
    const response = await myDBQuery({userID});
    if (response.error) {
      throw response.error;
    }
    return response;
  },
});

const currentUserInfoQuery = selector({
  key: 'CurrentUserInfoQuery',
  get: ({get}) => get(userInfoQuery(get(currentUserIDState))),
});

const friendsInfoQuery = selector({
  key: 'FriendsInfoQuery',
  get: ({get}) => {
    const {friendList} = get(currentUserInfoQuery);
    return friendList.map(friendID => get(userInfoQuery(friendID)));
  },
});

function CurrentUserInfo() {
  const currentUser = useRecoilValue(currentUserInfoQuery);
  const friends = useRecoilValue(friendsInfoQuery);
  const setCurrentUserID = useSetRecoilState(currentUserIDState);
  return (
    <div>
      <h1>{currentUser.name}</h1>
      <ul>
        {friends.map(friend =>
          <li key={friend.id} onClick={() => setCurrentUserID(friend.id)}>
            {friend.name}
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
          <CurrentUserInfo />
        </React.Suspense>
      </ErrorBoundary>
    </RecoilRoot>
  );
}
```

## 同時要求
上記の例を見るとわかるように、`friendsInfoQuery`はクエリーを使用して、各友人の情報を取得します。しかし、これをループで行うことで、本質的に[シリアライズ](http://e-words.jp/w/%E3%82%B7%E3%83%AA%E3%82%A2%E3%83%A9%E3%82%A4%E3%82%BA.html)されます。検索が早ければ、それでいいかもしれません。情報量が多い場合は、[waitForAll](https://recoiljs.org/docs/api-reference/utils/waitForAll)などの並行処理ヘルパーを使用して並行処理を実行できます。このヘルパーは、依存関係の配列と名前付きオブジェクトの両方を受け入れます。

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

[waitForNone](https://recoiljs.org/docs/api-reference/utils/waitForNone)を使用して、部分的なデータを含むUIへの増分更新を処理できます。

```React
const friendsInfoQuery = selector({
  key: 'FriendsInfoQuery',
  get: ({get}) => {
    const {friendList} = get(currentUserInfoQuery);
    const friendLoadables = get(waitForNone(
      friendList.map(friendID => userInfoQuery(friendID))
    ));
    return friendLoadables
      .filter(({state}) => state === 'hasValue')
      .map(({contents}) => contents);
  },
});
```

## プリフェッチ*
パフォーマンス上の理由から、レンダリングの前にフェッチを開始することをお勧めします。これにより、レンダリングの開始時にクエリーを実行できます。Reactのドキュメントには、いくつかの例が記載されています。このパターンはRecoilでも有効です。
*[プリフェッチ](http://e-words.jp/w/%E3%83%97%E3%83%AA%E3%83%AD%E3%83%BC%E3%83%89.html)

上の例を変更して、ユーザーがボタンをクリックしてユーザーを変更するとすぐに次のユーザー情報のフェッチを開始するようにします。

```React
function CurrentUserInfo() {
  const currentUser = useRecoilValue(currentUserInfoQuery);
  const friends = useRecoilValue(friendsInfoQuery);

  const changeUser = useRecoilCallback(({snapshot, set}) => userID => {
    snapshot.getLoadable(userInfoQuery(userID)); // pre-fetch user info
    set(currentUserIDState, userID); // change current user to start new render
  });

  return (
    <div>
      <h1>{currentUser.name}</h1>
      <ul>
        {friends.map(friend =>
          <li key={friend.id} onClick={() => changeUser(friend.id)}>
            {friend.name}
          </li>
        )}
      </ul>
    </div>
  );
}
```

## React Suspense を使わない
保留中の非同期セレクタを処理するためにReact Suspenseを使用する必要はありません。[useRecoilValueLoadable()](https://recoiljs.org/docs/api-reference/core/useRecoilValueLoadable)フックを使用して、レンダリング中のステータスを決定することもできます。

```React
function UserInfo({userID}) {
  const userNameLoadable = useRecoilValueLoadable(userNameQuery(userID));
  switch (userNameLoadable.state) {
    case 'hasValue':
      return <div>{userNameLoadable.contents}</div>;
    case 'loading':
      return <div>Loading...</div>;
    case 'hasError':
      throw userNameLoadable.contents;
  }
}
```