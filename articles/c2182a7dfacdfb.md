---
title: "hogehoge"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

## Reducer とは

コンポーネントが複雑になるにつれて、コンポーネントの状態が更新されるさまざまな方法をすべて一目で確認することが難しくなる可能性があります。

たとえば、以下の `TaskApp` コンポーネントは状態のタスクの配列を保持し、3 つの異なるイベント ハンドラーを使用してタスクを追加、削除、および編集します。

```jsx
const [tasks, setTasks] = useState(initialTasks);

function handleAddTask(text) {
  setTasks([
    ...tasks,
    {
      id: nextId++,
      text: text,
      done: false,
    },
  ]);
}

function handleChangeTask(task) {
  setTasks(
    tasks.map((t) => {
      if (t.id === task.id) {
        return task;
      } else {
        return t;
      }
    })
  );
}

function handleDeleteTask(taskId) {
  setTasks(tasks.filter((t) => t.id !== taskId));
}
```

@[codesandbox](https://codesandbox.io/embed/sleepy-austin-yrv1mz?fontsize=14&hidenavigation=1&theme=dark)

各イベント ハンドラーは、状態を更新するために `setTasks` を呼び出します。
このコンポーネントが成長するにつれて、全体に散りばめられた状態ロジックの量も増加します。
この複雑さを軽減し、すべてのロジックを 1 つのアクセスしやすい場所に保持するために、その状態ロジックをコンポーネント外の「Reducer」と呼ばれる単一の関数に移動できます。

レデューサーは、状態を処理する別の方法です。次の 3 つの手順で `useState` から `useReducer` に移行できます。

1. イベント ハンドラーからアクションをディスパッチします。
2. 特定の状態とアクションの次の状態を返すレデューサー関数を作成します。
3. コンポーネントのレデューサーを使用します。

### 1. イベント ハンドラーからアクションをディスパッチします。

イベントハンドラーの中身を dispatch 関数で表します。

```jsx
function handleAddTask(text) {
  dispatch({
    type: "added",
    id: nextId++,
    text: text,
  });
}

function handleChangeTask(task) {
  dispatch({
    type: "changed",
    task: task,
  });
}

function handleDeleteTask(taskId) {
  dispatch({
    type: "deleted",
    id: taskId,
  });
}
```

`type`によって実行する処理を切り分けている。
処理内容を書いていないので、コード量は減らすことができる
ちなみに、dispatch関数にわたすオブジェクトは、`action object`と呼ばれる

### 
この例で状態設定ロジックをイベント ハンドラーからレデューサー関数に移動するには、次のことを行います。

1. 現在の状態 ( tasks) を最初の引数として宣言します。
2. action2 番目の引数としてオブジェクトを宣言します。
レデューサーから次の状態を返します(React が状態を設定します)。
