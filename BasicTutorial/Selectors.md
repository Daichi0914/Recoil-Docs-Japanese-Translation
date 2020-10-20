# Selector

**selector**は、**divider state (派生状態)**を表します。divider stateは、与えられたstateを何らかの方法で変更する純粋な関数にstateを渡す出力と考えることができます。

派生stateは、他のデータに依存する動的データを構築できるため、強力な概念です。todoリストアプリケーションのコンテキストでは、次のものが派生stateと見なされます。

* **フィルタされたtodoリスト**:いくつかの基準(すでに完了している項目を除外するなど)に基づいてフィルタされた特定の項目を持つ新しいリストを作成することによって、完全なtodoリストから派生します。
* **ToDoリストの統計**:リスト内の項目の総数、完了した項目の数、完了した項目の割合など、リストの有用な属性を計算することによって、完全なToDoリストから得られます。

フィルタされたtodoリストを実装するには、atomに値を保存できる一連のフィルタ条件を選択する必要があります。
使用するフィルタオプションは、「すべて表示」、「完了を表示」、および「未完了を表示」です。デフォルト値は「すべて表示」です。

```React
const todoListFilterState = atom({
  key: 'todoListFilterState',
  default: 'Show All',
});
```

`todoListFilterState`と`todoListState`を使用して、フィルタされたリストを導出する`filteredTodoListState`selectorを構築できます。

```React
const filteredTodoListState = selector({
  key: 'filteredTodoListState',
  get: ({get}) => {
    const filter = get(todoListFilterState);
    const list = get(todoListState);

    switch (filter) {
      case 'Show Completed':
        return list.filter((item) => item.isComplete);
      case 'Show Uncompleted':
        return list.filter((item) => !item.isComplete);
      default:
        return list;
    }
  },
});
```

`filteredTodoListState`は内部的に2つの依存関係 (`todoListFilterState`と`todoListState`) を追跡し、どちらかが変更された場合に再実行されるようにします。

>コンポーネントの観点から見ると、selectorは、atomを読み取るために使用されるのと同じhookを使用して読み取ることができます。ただし、一部のhookは書き込み**可能なstate**(i.e `useRecoilState()`)でのみ機能することに注意してください。
>全てのatomは書き込み可能stateであるが、書き込み可能stateとみなされるのは一部のselectorのみである(`get`プロパティと`set`プロパティの両方を持つselector)。このトピックの詳細は、コアコンセプトを参照してください。

フィルタリングされたtodoListの表示は、TodoListコンポーネントで1行変更するだけです。

```React
function TodoList() {
  // changed from todoListState to filteredTodoListState
  const todoList = useRecoilValue(filteredTodoListState);

  return (
    <>
      <TodoListStats />
      <TodoListFilters />
      <TodoItemCreator />

      {todoList.map((todoItem) => (
        <TodoItem item={todoItem} key={todoItem.id} />
      ))}
    </>
  );
}
```

`todoListFilterState`にデフォルト値「すべて表示(Show All)」が指定されているため、UIはすべてのtodoを表示していることに注意してください。
フィルタを変更するには、`TodoListFilters`コンポーネントを実装する必要があります。

```React
function TodoListFilters() {
  const [filter, setFilter] = useRecoilState(todoListFilterState);

  const updateFilter = ({target: {value}}) => {
    setFilter(value);
  };

  return (
    <>
      Filter:
      <select value={filter} onChange={updateFilter}>
        <option value="Show All">All</option>
        <option value="Show Completed">Completed</option>
        <option value="Show Uncompleted">Uncompleted</option>
      </select>
    </>
  );
}
```

数行のコードでフィルタリングを実装することができました！
同じ概念を使用して、`TodoListStats`コンポーネントを実装します。

次の統計を表示します。

* todo項目の合計数
* 完了項目の合計数
* 未完了項目の総数
* 達成率

統計情報ごとにselectorを作成することもできますが、より簡単な方法は、必要なデータを含むオブジェクトを返すselectorを1つ作成することです。このselectorを`doListStatsState`と呼びます。

```React
const todoListStatsState = selector({
  key: 'todoListStatsState',
  get: ({get}) => {
    const todoList = get(todoListState);
    const totalNum = todoList.length;
    const totalCompletedNum = todoList.filter((item) => item.isComplete).length;
    const totalUncompletedNum = totalNum - totalCompletedNum;
    const percentCompleted = totalNum === 0 ? 0 : totalCompletedNum / totalNum;

    return {
      totalNum,
      totalCompletedNum,
      totalUncompletedNum,
      percentCompleted,
    };
  },
});
```

`todoListStatsState`の値を読み取るには、もう一度`useRecoilValue()`を使用します。

```React
function TodoListStats() {
  const {
    totalNum,
    totalCompletedNum,
    totalUncompletedNum,
    percentCompleted,
  } = useRecoilValue(todoListStatsState);

  const formattedPercentCompleted = Math.round(percentCompleted * 100);

  return (
    <ul>
      <li>Total items: {totalNum}</li>
      <li>Items completed: {totalCompletedNum}</li>
      <li>Items not completed: {totalUncompletedNum}</li>
      <li>Percent completed: {formattedPercentCompleted}</li>
    </ul>
  );
}
```

まとめると、私たちはすべての要件を満たすtodoリストアプリを作成しました。

* todo項目を追加する
* todo項目を編集する
* todo項目を削除する
* todo項目をフィルタする
* 有用な統計を表示