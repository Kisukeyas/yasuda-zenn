---
title: "useEffect本当に使う？（React）"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react", "js"]
published: true
---

## useEffect とは

React のコンポーネント内でロジックを記述する際は下記があります。

1. レンダリング：state や props を用いて、画面に表示する JSX を返却します。レンダリング結果を計算するだけです。
2. イベントハンドラー：コンポーネント内でネストされた関数です。特定のユーザーアクションによって引き起こされるコンポーネントの副作用が含まれます。

しかし、上記では不十分な場合があります。

画面がレンダリングされて、表示されるたびにサーバーに接続する必要があるコンポーネントを思い浮かべます。
サーバーへの接続は画面の表示をするための計算ではないので、レンダリング中には発生しません (これは副作用です)
ただし、クリックのような単一の特定のイベントでもありません。

useEffect を使用すると、イベントハンドラーではなく、**レンダリング自体によって引き起こされる副作用**を指定できます。

フォームの送信などはユーザーがボタンを押すなど操作を加えた時に発生するので、イベントハンドラーですが、サーバーへの接続はコンポーネントが表示されると画面の操作に関係なく毎回起こります。
Effect は React がレンダリング結果を DOM にコミットする最後に行います。

## props が変更されたときに状態をリセットする

```jsx
export default function ProfilePage({ userId }) {
  const [message, setMessage] = useState("");

  // 🔴 Avoid: Resetting state on prop change in an Effect
  useEffect(() => {
    setMessage("");
  }, [userId]);
  // ...
}
```

ユーザー ID を変更するたびに message の値をリセットする処理を実装しています。このように書くこともあると思います。
しかし、これは、`ProfilePage` コンポーネントをレンダリングして、DOM にコミットするときに、再レンダリングが走り、結果的に２回レンダリングが走ることになり、非効率的です。

```jsx
export default function ProfilePage({ userId }) {
  return <Profile userId={userId} key={userId} />;
}

function Profile({ userId }) {
  // ✅ This and any other state below will reset on key change automatically
  const [message, setMessage] = useState("");
  // ...
}
```

上のように Key を使うと、userID ごとに別のコンポーネントと認識されて、state の値がリセットされます。
こうするとレンダリングの回数を減らすことができます。

## 最後に

他にも必要のない Effect 処理のパターンがあれば教えてください。
間違っていましたら、ご指摘ください。

https://react.dev/learn/you-might-not-need-an-effect
