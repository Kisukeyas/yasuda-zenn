---
title: "イベントハンドラの伝播を防ぐ方法について(React)"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react", "js"]
published: true
---

## イベントハンドラの伝播

下のコードでは<div>の中に<button>があります。
どちらのタグにも`onClick`ハンドラーを記述しています。
ボタンをクリックすると、

1. ボタンの onClick が呼び出されます。
2. div の onClick が呼び出されます。
   子コンポーネントのイベントが親コンポーネントにも到達しています。これをイベントハンドラの伝播と言います。 \*`onScroll`を除いて全てのイベントは伝播します。

@[codesandbox](https://codesandbox.io/embed/exciting-stallman-xl0zrm?fontsize=12)

## どうやってイベントハンドラーの伝播を止めるか

イベントハンドラは唯一の引数としてイベントオブジェクトを受け取ります。
通常`e`と書きます。このオブジェクトを使用してイベントに関する情報を読み取ることができます。

このイベントオブジェクトを使用すると伝播を停止することもできます。
イベントが親コンポーネントに到達するのを防ぎたい場合は
`e.stopPropagation()`を呼び出す必要があります。

@[codesandbox](https://codesandbox.io/embed/falling-sun-9dw9vk?fontsize=12)

ボタンを押すと

1. Button コンポーネントのハンドラーを呼び出す
2. まず最初に`e.stopPropagation()`イベントが親コンポーネントに伝播しないように止めています。
3. 次に onClick()Props が呼び出されて実行されて、画面にアラートが出ます。

## 意図的に親コンポーネントのイベントを先に呼び出したい時

`Capture`をつけることで先に div のイベントハンドラーが呼び出されるようになります。
順番としては、

1. 下に移動し、すべての onClickCapture ハンドラーを呼び出します。
2. クリックされた要素の onClick ハンドラを実行します。
3. 上向きに移動し、すべての onClick ハンドラを呼び出します。

```jsx
<div
  onClickCapture={() => {
    /* this runs first */
  }}
>
  <button onClick={(e) => e.stopPropagation()} />
  <button onClick={(e) => e.stopPropagation()} />
</div>
```
