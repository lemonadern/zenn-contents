---
title: "Vueで子→親の処理を実行したいときにpropsと$emitを使い分けるなら"
emoji: "⛳"
type: "tech"
topics: ["vue"]
published: false
---

Vue.jsにおいて子コンポーネントでイベントを発火させて親コンポーネントの処理を実行したいとき、関数を子コンポーネントの`props`に渡す方法と`$emit`を使う方法があります。
