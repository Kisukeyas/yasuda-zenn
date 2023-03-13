---
title: "useReducerについて（React)"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react", "js"]
published: true
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
ちなみに、dispatch 関数にわたすオブジェクトは、`action object`と呼ばれる

### 2. 特定の状態とアクションの次の状態を返すレデューサー関数を作成

状態設定ロジックをイベント ハンドラーからレデューサー関数に移動するには、次のことを行います。

1. 現在の状態 (tasks) を最初の引数として宣言します。
2. 2 番目の引数として`actionオブジェクト`を宣言します。
3. レデューサーから次の状態を返します(React が状態を設定します)。

```jsx
function tasksReducer(tasks, action) {
  switch (action.type) {
    case "added": {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case "changed": {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case "deleted": {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error("Unknown action: " + action.type);
    }
  }
}
```

### 3. コンポーネントのレデューサーを使用

```jsx
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

useReducer フックは useState に似ています。初期状態を渡す必要があり、ステートフルな値と状態を設定する方法 (この場合はディスパッチ関数) を返します。しかし、それは少し違います。

useReducer でコンポーネントを描き直すと 状態変数変更処理部・イベントハンドラー部が分離されて、コードの可視性が上がる

```jsx
import { useReducer } from "react";
import AddTask from "./AddTask.js";
import TaskList from "./TaskList.js";

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

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

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case "added": {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case "changed": {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case "deleted": {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error("Unknown action: " + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: "Visit Kafka Museum", done: true },
  { id: 1, text: "Watch a puppet show", done: false },
  { id: 2, text: "Lennon Wall pic", done: false },
];
```

このように懸念事項を分離すると、コンポーネント ロジックが読みやすくなります。現在、イベント ハンドラーはアクションをディスパッチすることによって何が起こったかのみを指定し、リデューサー関数はそれらに応答して状態がどのように更新されるかを決定します。

## 最後に

まずは、useState で書いてみて、状態を変更するロジックが複数出てきた時に、useReducer の使用を検討してみてはいかがでしょうか？
