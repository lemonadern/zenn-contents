---
title: "プロトタイピングツールとしての RedwoodJS"
emoji: "🪵"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["redwoodjs", "react", "graphql", "prisma", "prototyping"]
published: false
---

本稿は、Webアプリのプロトタイプを作るための道具として RedwoodJS を紹介する記事です。

# 前説：プロトタイピングにおける技術選定

シンプルなWebアプリのプロトタイプを作るとき、みなさんはどのような技術選定を行うでしょうか。

プロトタイプと言えど UI の検証もある程度は含んでいる場合がほとんどでしょうから、筆者としては UI の構築には React を利用したい[^1]ところです。テンプレートエンジンでは著しく開発効率が落ちるので、フルスタックフレームワークとしての Rails や Django はこの時点で選べないことになります。
しかし、React を選んだとしても大半のアプリケーションでは永続層が必要ですし、フロントエンドで計算させたくないロジックも多々あります。バックエンドを別で作る場合に直面するのは、クライアント側とのAPIスキーマの整合性をどう取るかという問題です。できればフェッチングには型を効かせたいので [aspida](https://github.com/aspida/aspida) や [zodios](https://github.com/ecyrbe/zodios) などを利用することになりますが、怠惰な筆者の頭に擡げるのは、最初からそのあたりがすべて揃っているスタックが欲しいという気持ちばかりです。[^2]

プロトタイピングのために手の込んだ設定をしたくない、でも開発速度のためには開発体験も捨てられないというジレンマに陥ったことのある開発者の方は少なくないのではないでしょうか。

幸いなことに、近年ではそんな問題を解決するフレームワークが次々登場しています。今回はそのうちの1つ、**RedwoodJS** をご紹介します。

# RedwoodJS とは

https://redwoodjs.com/

RedwoodJS は、 TypeScript / JavaScript 製の Web フルスタックフレームワークです。

- React + React 周辺の各種便利ライブラリ
- GraphQL
- Prisma による永続層アクセス
- その他 Redwood による各種サポート

などの組み合わせによって、非常に便利な開発体験を実現しています。ここでは個別に挙げませんが、[リポジトリ](https://github.com/redwoodjs/redwood)あるいは[ドキュメント](https://redwoodjs.com/docs/index)を見れば、RedwoodJS がいかに高機能なフレームワークであるかがわかると思います。

## RedwoodJS のありがたいお言葉

[トップページ](https://redwoodjs.com/)に掲げられている言葉を見てみましょう。

> **Focus on building your startup, not fighting your framework.**

なんて頼もしい言葉なんでしょうか。
「プロトタイピングのプロジェクトでちまちま設定してられないけど開発体験が…」
などど喚いていた筆者に向けたメッセージのようです。

# 便利なところ

## Zero Config の時点でめちゃくちゃリッチ

最初にプロジェクトを生成した時点で、ESLint, EditorConfig, VSCode の設定ファイルまでもが追加されます。VSCodeユーザなら何も設定せずとも `Format on Save` が動きますし、さらにはデフォルトの時点で Storybook も Jest も使えます。
設定が何やらとごちゃごちゃ言っていた筆者を一撃で黙らせてくれました。

## 自動生成がすごい

RedwoodJS は Ruby on Rails に影響を受けたフレームワークです。Redwood には、Rails のジェネレータよろしく `generate` や `scaffold` といった機能があります。

Redwood の開発で利用するほとんどの単位に `generate` コマンドが用意されている[^3]ので、ボイラープレート的な内容を書くことなくコードを書き始められます。またこのとき、デフォルトではテスト用のファイルや Storybook のファイルも同時に生成されます。なんて至れり尽くせりなんでしょうか。[scaffdog](https://github.com/scaffdog/scaffdog) や [Hygen](https://github.com/jondot/hygen) を使って頑張って整備していた内容を肩代わりしてくれているようです。
設定が何やら言っていた筆者を...(略)

自動生成のもう一つ嬉しいポイントは、ディレクトリ構成が自明に定まる点です。Redwood に従っているだけで、いい感じのディレクトリ構成を保って開発が進められます。

最も単純なCRUDの動作を作るだけなら、Prisma でデータベースのスキーマを書いて `scaffold` コマンドを叩くだけでフロントエンドまで全部が勝手に生成されます。[チュートリアル](https://redwoodjs.com/docs/tutorial/chapter2/getting-dynamic)でこれを見たときはさすがに驚きました。

## Form が楽

フォーム関連の機能として、[React Hook Form](https://github.com/react-hook-form/react-hook-form) をラップしたコンポーネントやフックがすでに用意されています。Redwood 用に（少しだけ）カスタマイズされていたりしますが、十分にその恩恵を受けて記述できます。

## フェッチングに型が効く

GraphQL が使われているので、Query や Mutation に型が付きます。とても嬉しいですね。

# 全体の概観

Redwoods Way なアプリケーションは、どんな構成で書くことになるのでしょうか。ここでは、フロントエンドおよびバックエンドにおける主要なコンポーネントについて紹介することで、RedwoodJS アプリケーションの構成に対するなんとなくのイメージを持っていただければと思います。

## フロントエンド

フロントエンドにおける主要なコンポーネントは、

- Page
    - URLのパスに対応したページ
    - Routes.tsxで指定
- Layout
    - children を受け取り、ページのレイアウトを決める
    - Routes.tsx でページに対して指定
- Cell
    - 非同期処理(データフェッチ)を書くのに特化した機能
    - ( Redwood 特有のポイント で後述)
- Component
    - 普通のコンポーネント

の4つです。

Cell だけは特殊なので後述しますが、それ以外は問題なく受け入れられると思います。

## バックエンド

バックエンドにおける主要なコンポーネントは

- Prisma スキーマ
    - データベーススキーマの定義
- services
    - SDL に対応した GraphQL Resolver
    - ビジネスロジック・バリデーションなど
    - Prisma による永続層アクセス
- GraphQL SDL
    - GraphQL スキーマの定義

があります。 

services についてあまりに責務が多すぎると感じるかもしれませんが、RailsのやっているMVCだと思えば大したことではありません。GraphQL Resolver はほとんど Controller と見ることができますし、Prisma をはテーブルのモデルを返す ActiveRecord だと思ってください（流石に厳しいですね）
実際のところ、Experimental ではあるものの RedwoodJS には [RedwoodRecord](https://redwoodjs.com/docs/redwoodrecord) という機能が存在します。 Prisma をラップして実現されていて、 Rails における ActiveRecord の書き味を目指して開発されているORMです。

Redwood を Rails に無理やり当てはめて語ってしまいましたが、実は無理にこう考える必要はありません。なぜなら Resolver はただのJSの関数だからです。
Resolver に課せられた制約は、

- SDLに定義された Query または Mutation と同名であること （RedwoodJS での制約）
- Query または Mutation に対応するデータを返すこと （Resolver としての振る舞い）

だけです。(後述します) これさえ守っていれば、処理を他の関数に分割してしまっても何ら問題がありません。 Prisma をただのデータアクセスライブラリとして利用して、一連の処理をいくつかの関数に分解すれば、ある程度複雑な処理も書けてしまいます。[^4]

# Redwood 特有のポイント

デフォルトで豪勢な開発体験を提供してくれる Redwood ですが、Rails Way ならぬ Redwoods Way を実現するために独特な機能やルールを追加したりしているので、React や GraphQL を知っている人でも最低限抑えるべきポイントがいくつかあります。ここでは、Redwood 特有のクセを感じるポイントを紹介しておきます。

## 非同期のフェッチングには `Cell` を使う

いきなり聞き慣れない言葉が出てきました。 `Cell` とは、RedwoodJS でフェッチング（非同期処理）を行うために用意された機能です。

https://redwoodjs.com/docs/cells#usage

`QUERY` , `Loading` , `Empty` , `Success` , `Failure` などの決められた名前の定数を定義してエクスポートすると、Redwoodは フェッチングや非同期処理の状態に応じたコンポーネントの出し分けを勝手に行います。

下に示すのは、[チュートリアル](https://redwoodjs.com/docs/tutorial/chapter2/cells#our-first-cell)から引用したCellのコードにコメントを追加したものです。

```tsx: web/src/components/ArticlesCell/ArticlesCell.tsx
import type { ArticlesQuery } from 'types/graphql'
import type { CellSuccessProps, CellFailureProps } from '@redwoodjs/web'

// `Success` で利用するデータを取得する GraphQL Query
export const QUERY = gql`
  query ArticlesQuery {
    articles {
      id
    }
  }
`
// ローディング時に表示するコンポーネント
export const Loading = () => <div>Loading...</div>

// クエリの結果が空だったときに表示するコンポーネント
export const Empty = () => <div>Empty</div>

// フェッチングに失敗したときに表示するコンポーネント
export const Failure = ({ error }: CellFailureProps) => (
  <div style={{ color: 'red' }}>Error: {error.message}</div>
)

// フェッチングに成功したときに表示するコンポーネント
export const Success = ({ articles }: CellSuccessProps<ArticlesQuery>) => {
  return (
    <ul>
      {articles.map((item) => {
        return <li key={item.id}>{JSON.stringify(item)}</li>
      })}
    </ul>
  )
}
```

コンポーネントを定義してエクスポートするだけという、なかなか見たことのない書き方です。

コンポーネントの中で条件分岐を行うのではなく、別でコンポーネントを定義してより宣言的に書こうとするスタイルだけ見れば、Suspenseを使った非同期処理にも近しいものを感じます。

## ルーティングは react-router 系

RedwoodJS のルーティングは Next.js のような File System Routing ではなくreact-router スタイルで定義します。URLパスとページの対応はもちろん、ページのレイアウトや認証についてもここで指定します。

```tsx: web/src/Routes.tsx
import { Router, Route, Set, Private } from '@redwoodjs/router'

import BlogLayout from 'src/layouts/BlogLayout/BlogLayout'
import ScaffoldLayout from 'src/layouts/ScaffoldLayout'

const Routes = () => {
  return (
    <Router>
      <Route path="/login" page={LoginPage} name="login" />
      <Route path="/signup" page={SignupPage} name="signup" />
      <Route path="/forgot-password" page={ForgotPasswordPage} name="forgotPassword" />
      <Route path="/reset-password" page={ResetPasswordPage} name="resetPassword" />

      {/* 認証が必要なページ */}
      <Private unauthenticated="home">
        <Set wrap={ScaffoldLayout} title="Posts" titleTo="posts" buttonLabel="New Post" buttonTo="newPost">
          <Route path="/admin/posts/new" page={PostNewPostPage} name="newPost" />
          <Route path="/admin/posts/{id:Int}/edit" page={PostEditPostPage} name="editPost" />
          <Route path="/admin/posts/{id:Int}" page={PostPostPage} name="post" />
          <Route path="/admin/posts" page={PostPostsPage} name="posts" />
        </Set>
      </Private>

      {/* BlogLayout を適用するページ */}
      <Set wrap={BlogLayout}>
        <Route path="/article/{id:Int}" page={ArticlePage} name="article" />
        <Route path="/contact" page={ContactPage} name="contact" />
        <Route path="/about" page={AboutPage} name="about" />
        <Route path="/" page={HomePage} name="home" />
      </Set>
      <Route notfound page={NotFoundPage} />
    </Router>
  )
}

export default Routes
```
`<Private>` で認証が必要なページを指定したり、 `<Set>` でページに対してレイアウトを適用したりしています。

## GraphQL Resolver Map は書かない

RedwoodJS では、GraphQL のリゾルバを SDL にマッピングする [Resolver Map](https://www.the-guild.dev/graphql/tools/docs/resolvers) を書く必要はありません。定義したリゾルバは、SDLで定義したものと**同名**の Query あるいは Mutation に自動でマッピングされるからです。

```tsx:api/src/services/contacts/contacts.ts
export const contacts: QueryResolvers['contacts'] = () => {
  return db.contact.findMany()
}

export const contact: QueryResolvers['contact'] = ({ id }) => {
  return db.contact.findUnique({
    where: { id },
  })
}
```

```tsx:api/src/graphql/contacts.sdl.ts
export const schema = gql`
  type Contact {
    id: Int!
    name: String!
    email: String!
    message: String!
    createdAt: DateTime!
  }

  type Query {
    contacts: [Contact!]! @requireAuth
    contact(id: Int!): Contact @requireAuth
  }
`
```

# まとめ

Redwood は、React 周りの便利ツールが全部入りなうえに、永続層までの繋ぎ込みを非常に便利にしてくれるフレームワークです。また “Redwoods Way“ な規約に意図的に縛られることで、効率的な開発が行えます。このような特徴は保守を前提としたプロダクションコードに採用するには検討の余地があるものですが、プロトタイプのように検証が終わったらすぐ捨てる前提の短命なプロジェクトに対しては多くの場合で有効なものだと言えないでしょうか。
React を普段書いていて GraphQL, Prisma を知っている開発者であれば、ほとんど学習コストをかけることなく開発に取り掛かることができます。 React を使ったフルスタックな Web アプリを書く場合があれば、ぜひ検討してみてください。

Redwood はまだまだ日本語の記事も少ないフレームワークなので、この記事をきっかけに興味を持って利用してくれる人が増えることを願っています。

# RedwoodJS に入門するには
この記事を読んで RedwoodJS に入門したくなった方は、ぜひ公式の[チュートリアル](https://redwoodjs.com/docs/tutorial/foreword)をやってみてください。とてもわかりやすく学べます。


# RedwoodJS の日本語記事
https://zenn.dev/mugi/articles/334f9556095a07

https://zenn.dev/rinda_1994/articles/9d4af758ea4980

[^1]: 筆者はフロントエンドを書いている人だから、という理由もあります。
[^2]: すべて揃ったスターターを自分で作れ？全くそのとおりですね…
[^3]: https://redwoodjs.com/docs/cli-commands#generate-alias-g
[^4]: 何なら [こんな](https://speakerdeck.com/naoya/typescript-niyoru-graphql-batukuendokai-fa) スタイルの開発もある程度は真似できます。