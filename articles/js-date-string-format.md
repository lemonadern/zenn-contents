---
title: "「new Date() がSafariでNaNになった」人に知ってほしい、日付文字列フォーマットの話"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript","ecmascript","date","safari"]
published: false
---

# 対象読者
```js
const date = new Date("2022-06-07 20:04:03");
// Invalid Date
```
上記のように、`new Date()`を使って文字列からDateオブジェクトを生成しようとしたら、Safariで`NaN`や`Invalid Date`になってしまった人のための記事です。

:::message
ここではSafariを例に挙げていますが、「特定のブラウザで思い通りに動かない」という状況のあなたもぜひ読んでください。
:::

:::details ブラウザごとの動作
冒頭のコードの
```js
new Date(`"2022-06-07 20:04:03"`)
```
の部分を、各ブラウザで評価してみます。

## Chrome
(Version 102.0.5005.61)

`Tue Jun 07 2022 20:04:03 GMT+0900 (Japan Standard Time)`

## FireFox
(Version 100.0.2)

`Date Tue Jun 07 2022 20:04:03 GMT+0900 (Japan Standard Time)`

## Safari

WIPWIPWIPWIPWIPWIPWIPWIPWIP
:::

# 原因
Safariだけ日付が得られないという自体が起きていますが、実はこれは仕様どおりの挙動です。

正確に言えば、これは **Date に渡している `"2022-06-07 20:04:03"` という文字列が、言語仕様としてサポートしているフォーマットではない**ために起きる挙動です。