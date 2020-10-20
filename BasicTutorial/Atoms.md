# Atom

atomには、アプリケーションstateの信頼できる情報源(source)が含まれています。
このtodoリストでは、sourceはオブジェクトの配列であり、各オブジェクトはtodo項目を表します。

リストatomを`todoListState`と呼び出し、`atom()`関数を使用して作成します。

```React
const todoListState = atom({
  key: 'todoListState',
  default: [],
});
```

atomに一意のkeyを与え、デフォルト値を空の配列に設定します。
このatomの内容を読み取るには、TodoListコンポーネントの`useRecoilValue()`hookを使用します。

```React
function TodoList() {
  const todoList = useRecoilValue(todoListState);

  return (
    <>
      {/* <TodoListStats /> */}
      {/* <TodoListFilters /> */}
      <TodoItemCreator />

      {todoList.map((todoItem) => (
        <TodoItem key={todoItem.id} item={todoItem} />
      ))}
    </>
  );
}
```

コメントアウトされたコンポーネントは、次のセクションで実装されます。

新しいtodo項目を作成するには、`todoListState`の内容を更新するsetter関数にアクセスする必要があります。
`useSetRecoilState()`hookを使用して、`TodoItemCreator`コンポーネントに設定関数を取得できます。

```React
function TodoItemCreator() {
  const [inputValue, setInputValue] = useState('');
  const setTodoList = useSetRecoilState(todoListState);

  const addItem = () => {
    setTodoList((oldTodoList) => [
      ...oldTodoList,
      {
        id: getId(),
        text: inputValue,
        isComplete: false,
      },
    ]);
    setInputValue('');
  };

  const onChange = ({target: {value}}) => {
    setInputValue(value);
  };

  return (
    <div>
      <input type="text" value={inputValue} onChange={onChange} />
      <button onClick={addItem}>Add</button>
    </div>
  );
}

// utility for creating unique Id
let id = 0;
function getId() {
  return id++;
}
```

古いtodoリストに基づいて新しいtodoリストを作成できるように、setter関数の**updater**形式を使用していることに注意してください。

`TodoItem`コンポーネントは、todo項目の値を表示しますが、そのテキストを変更したり項目を削除したりすることもできます。`todoListState`を読み取って、項目テキストを更新し、完了としてマークし、削除するために使用する設定関数を取得するには、`useRecoilState()`を使用します。

```React
function TodoItem({item}) {
  const [todoList, setTodoList] = useRecoilState(todoListState);
  const index = todoList.findIndex((listItem) => listItem === item);

  const editItemText = ({target: {value}}) => {
    const newList = replaceItemAtIndex(todoList, index, {
      ...item,
      text: value,
    });

    setTodoList(newList);
  };

  const toggleItemCompletion = () => {
    const newList = replaceItemAtIndex(todoList, index, {
      ...item,
      isComplete: !item.isComplete,
    });

    setTodoList(newList);
  };

  const deleteItem = () => {
    const newList = removeItemAtIndex(todoList, index);

    setTodoList(newList);
  };

  return (
    <div>
      <input type="text" value={item.text} onChange={editItemText} />
      <input
        type="checkbox"
        checked={item.isComplete}
        onChange={toggleItemCompletion}
      />
      <button onClick={deleteItem}>X</button>
    </div>
  );
}

function replaceItemAtIndex(arr, index, newValue) {
  return [...arr.slice(0, index), newValue, ...arr.slice(index + 1)];
}

function removeItemAtIndex(arr, index) {
  return [...arr.slice(0, index), ...arr.slice(index + 1)];
}
```

これで、完全に機能するToDoリストができました！
次のセクションでは、selectorを使用してリストを次のレベルアップする方法について説明します。