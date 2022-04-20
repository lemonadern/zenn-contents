---
title: "new Date() で日付文字列のパースをするのを避けよう"
emoji: "😢"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [JavaScript, Date]
published: false
---

# TL;DR
- `new Date()` および `Date.parse()` でむやみに文字列の解釈をするのは避けるべき
  - 文字列の解析の挙動が環境によって異なるため
  - なかでもV8の実装ではパース時のバリデーションが非常に緩い
- 日付系のライブラリを適切に利用しよう

# 日付文字列をパースしてDateを得る

`"2022-01-01"` などの日付を表す文字列をパースして Date オブジェクトを得る方法はいくつかありますが、Google検索すると `new Date()` を利用する方法が多くヒットします。

```js
new Date("2022-01-01")
// => Sat Jan 01 2022 09:00:00 GMT+0900 (Japan Standard Time)
```
お手軽にパースすることができますし、正しい日付で生成できていますね。

しかしこの方法で日付文字列のパースを行う際には注意が必要です。
使い方によっては環境間（主にブラウザ間）における異なった動作を誘発するため、`new Date()` およびこの後に説明する `Date.parse()` を用いて文字列をパースするのは避けるべきです。

# `new Date`で文字列の解析をする上で知っておくべき仕様
## new Date に文字列を渡したときの動作
`new Date(dateString)`のように呼び出した場合、内部的には `Date.parse` を用いて文字列の解析が行われます。[^1]

## Date.parse が受け取る文字列のフォーマット
このとき、日付を表す文字列に要求されるもっとも基本的なフォーマットは、 ISO8601 をもとにECMAScriptで定義された[^2]ものです。
```
YYYY-MM-DDTHH:mm:ss.sssZ
YYYY
YYYY-MM
YYYY-MM-DD
THH:mm
THH:mm:ss
THH:mm:ss.sss
```
今回は `YYYY-MM-DD` のようなケースを想定しているので、日付を表す文字列はハイフン区切りになっている必要があることがわかりました。
ところが、 `Date.parse` が受け入れるフォーマットはそれだけではありません。

>  If the String does not conform to that format the function may fall back to any implementation-specific heuristics or implementation-specific date formats. [^3]

> 与えられた文字列がそのフォーマットに適合していない場合、実装固有の慣習的なフォーマットやその他の実装固有のフォーマットにフォールバックすることがあります。

なんと `Date.parse` は実装依存でその他のフォーマットも許容するのです。
`Date.parse` が `NaN` を返すのは、文字列の解釈ができなかった場合や、不正な要素値を含んでいると判断された場合です。
これは、ある環境で解析できる文字列が、他の環境では解析できないかもしれないということを意味します。
たとえば Chrome, Firefox, Safari はそれぞれJSエンジンが異なるので、挙動の違いを簡単に見ることができます。

```js
// ISO8601 に準拠していないフォーマットの文字列を与える

// Chrome (Google Chrome 100.0.4896.127) 
new Date("2022.04.20")
// => Wed Apr 20 2022 00:00:00 GMT+0900 (Japan Standard Time)

// Firefox (Mozilla Firefox 99.0.1)
new Date("2022.04.20")
// => Invalid Date
```
ユースケースによりますが、 このような仕様を知らずに `new Date()` や `Date.parse()` を利用した場合、環境差異による問題を発生させてしまう可能性があります。

# 特にバリデーションの緩いV8
ここまで読み、
**「環境差異が問題とはいえ、ISO8601 に準拠した書式の文字列を利用すれば問題ないのでは」**
と思った方もいるのではないでしょうか。

しかし罠は他にもあります。
試しに Node.js で、存在しない日付である `"2022-02-29"` を `new Date()` に与えてみましょう。

```js
// Node.js v16.14.2

new Date("2022-02-29")
// => Tue Mar 01 2022 09:00:00 GMT+0900 (Japan Standard Time)
```
なんと3月1日になってしまいました。
V8エンジンを利用している Node.js や Chrome では、存在しない日付に対しても非常に緩いバリデーションが行われます。
具体的には、存在しない日付でも31日までは正しい日付として扱われ、実際の月に差分が加算された日付が返されます。
同じ処理を実行しても Firefox や Safari では `Invalid Date` となるので、V8固有の実装であることがわかります。

便利な機能に感じるかもしれませんが、文字列のバリデーションを目的としている場合などには依然として使いづらい挙動ですし、これが環境固有のものであることを知っておかなければなりません。

# 回避策
もしも文字列をパースしてDateオブジェクトを得たいのならば、落とし穴が無数に存在する Date の標準機能を利用するよりも、ライブラリに頼ることをおすすめします。

ここまで見てきたような
- 書式に気をつけなければならない
- 存在する日付かどうかを気にしなければならない

などの問題であれば、大抵の日付計算系のライブラリを利用することで解決することができます。
ここでは、 `date-fns` の例を示します。
```js
// ISO8601に準拠していない文字列でも、書式を指定して対応できる
dateFns.parse('2022.04.20', 'yyyy.MM.dd', new Date())
// => Wed Apr 20 2022 09:00:00 GMT+0900 (Japan Standard Time)

// 存在しない日付は、`Invalid Date` となる
dateFns.parse('2022-02-29', 'yyyy-MM-dd', new Date())
// => Invalid Date
```

# まとめ
というわけで、 `new Date()` および `Date.parse()` は環境間で動作が異なるのでむやみな利用を避けようというお話でした。

誤った点がありましたら、やさしくマサカリを投げてくれると助かります。ぜひよろしくおねがいします。




[^1]: 渡された引数の数が1つで型変換後の引数の型がStringのとき（ざっくりすぎる説明）、parseメソッドと同じルールでパースが行われる。 https://tc39.es/ecma262/multipage/numbers-and-dates.html#sec-date
[^2]: ECMAScriptでは、`ISO 8601`の`Calendar Dates`拡張書式を単純化したものをを標準的な日付文字列変換のフォーマットとして定義している。 https://tc39.es/ecma262/multipage/numbers-and-dates.html#sec-date-time-string-format
[^3]: https://tc39.es/ecma262/multipage/numbers-and-dates.html#sec-date.parse