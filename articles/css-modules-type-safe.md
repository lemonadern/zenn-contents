---
title: "Next.js で型安全に CSS Modules を使う"
emoji: "💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["css", "cssmodules", "typescript"]
published: false
---

# 動機

## Next.js で CSS Modules を使う
Next.js では、ビルトインで CSS Modules がサポートされています。
具体的には、`*.module.css`という名前のファイルを生成すれば、
```tsx
import styles from 'src/styles/Home.module.scss'
```
のように読みこんで
```tsx
<div className={styles.container}>
  {/* ... */}
</div>
```
このように利用することができます。

## これだけでは不安
当たり前のように `styles.container`にアクセスしていますが、`container`という名前を持つクラスが存在している保証はありません。

:::details 型定義を覗いてみる
Next.jsが用意している`styles`の型定義を覗いてみましょう。
```ts:global.d.ts
declare module '*.module.scss' {
  const classes: { readonly [key: string]: string }
  export default classes
}
```
key が string型であることしか保証されていないので、TypeScript側からは存在しない型を指定できることがわかります。
:::

