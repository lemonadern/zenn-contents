---
title: "Next.js + TypeScript (+ ESLint + Prettier + husky + EditorConfig)の環境構築"
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs","typescript","eslint","prettier","husky"]
published: true
---

先日、自分用に Next.js のテンプレートリポジトリを作成しました。

:::message alert
(2022-10-21) 以下のリポジトリはメンテナンスされた状態にありません。
:::

https://github.com/lemonadern/template-nextjs-typescript
このときの作業メモをスクラップに残していたので、それを記事としてまとめておきます。
https://zenn.dev/lemonadern/scraps/27f37a37cce2c4

# 目指すもの
アプリを書くとき、すぐに快適に利用できる Next.js のテンプレート
- できる限り堅い型チェックができる設定がなされている
  - `tsconfig` の設定
- リンター・フォーマッタやその他の設定でコーディングスタイルを統一できる最低限の設定がなされている
  - ESLint, Prettier, EditorConfig など
- コミット時に自動でリンター・フォーマッタが実行される
  - husky, lint-staged
- VSCode を使っていれば、ほぼ設定無しで便利な機能の恩恵を受けられる
  - 自動フォーマットや拡張機能の設定
# 作業
## プロジェクト作成
プロジェクトを作成する
```sh
yarn create next-app --typescript
```
## ディレクトリの設定
自分で書くファイルを `src`にまとめる
(`pages/` や `styles/`) を `src/` 配下に移動する
```sh
mkdir src
mv pages/ src/
mv styles/ src/
```
:::details 移動後のディレクトリ構成
![移動後のディレクトリ構成](/images/directory-structure.png)
:::

## パスのエイリアスを追加
絶対パス読み込みができるようにする
```diff json:tsconfig.json
{
  "compilerOptions": {
+    "baseUrl": ".",
+    "paths": {
+      "@/*": ["src/*"],
+      "/*": ["*"]
+    },
  // ...
  }
}
```
`@/`で`src/`から、`/`でトップからのパスを書ける

## TypeScript の設定
https://github.com/tsconfig/bases を使う
いろんな tsconfig ファイルが用意されており、インストールして利用できる

Next.js 用の tsconfig も用意されているが、`strict:false` だったりするのであまり嬉しくない

TypeScriptからはできるだけ強い制約を得たいので、型チェックを厳しくするオプションが全て入っている `strictest` を利用し、その他の Next.js 用の設定は別途上書きする形にする


インストール
```sh
yarn add -D @tsconfig/strictest
```
`tsconfig.json` に追加
```diff json:tsconfig.json
{
+ "extends": "@tsconfig/strictest/tsconfig.json",
  "compilerOptions": {
    "baseUrl": ".",
    // ...
  },
  // ...
}
```

`tsconfig/bases` と重複していて必要ない設定項目を削除しておく
```diff json:tsconfig.json
{
  "extends": "@tsconfig/strictest/tsconfig.json",
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "/*": ["*"]
    },
    "target": "es5",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
-   "skipLibCheck": true,
-   "strict": true,
-   "forceConsistentCasingInFileNames": true,
    "noEmit": true,
-   "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}
```

## ESLint の設定
静的解析およびリントには ESLint を利用する
v11 からの Next.js にはデフォルトで ESLint が用意されているので設定ファイルがあるが、それは一旦削除する
```sh
rm .eslintrc.json   
```
対話式で設定を作成できるコマンドがあるので、それを利用して一から設定を行う
```sh
yarn create @eslint/config
```
適宜、質問に答える
```
? How would you like to use ESLint? … 
  To check syntax only
  To check syntax and find problems
▸ To check syntax, find problems, and enforce code style

? What type of modules does your project use? … 
▸ JavaScript modules (import/export)
  CommonJS (require/exports)
  None of these

? Which framework does your project use? … 
▸ React
  Vue.js
  None of these

? Does your project use TypeScript? ‣ No / Yes # Yes を選択

? Where does your code run? …  (Press <space> to select, <a> to toggle all, <i> to invert selection)
✔ Browser
✔ Node

? How would you like to define a style for your project? … 
▸ Use a popular style guide
  Answer questions about your style

? Which style guide do you want to follow? … 
  Airbnb: https://github.com/airbnb/javascript
  Standard: https://github.com/standard/standard
▸ Google: https://github.com/google/eslint-config-google
  XO: https://github.com/xojs/eslint-config-xo

? What format do you want your config file to be in? … 
▸ JavaScript
  YAML
  JSON

Checking peerDependencies of eslint-config-google@latest
Local ESLint installation not found.
The config that you've selected requires the following dependencies:

eslint-plugin-react@latest @typescript-eslint/eslint-plugin@latest eslint-config-google@latest eslint@>=5.16.0 @typescript-eslint/parser@latest
? Would you like to install them now with npm? ‣ No / Yes # No を選択
```
eslint 関連のものを yarn でインストールする
```sh
yarn add -D eslint-plugin-react@latest @typescript-eslint/eslint-plugin@latest eslint-config-google@latest eslint@>=5.16.0 @typescript-eslint/parser@latest
```
Next.js用の設定を追加する
```diff js: eslintrc.js
module.exports = {
  'env': {
    'browser': true,
    'es2021': true,
  },
  'extends': [
    'plugin:react/recommended',
    'google',
+   'next/core-web-vitals',
  ],
  'parser': '@typescript-eslint/parser',
  'parserOptions': {
    'ecmaFeatures': {
      'jsx': true,
    },
    'ecmaVersion': 'latest',
    'sourceType': 'module',
  },
  'plugins': [
    'react',
    '@typescript-eslint',
  ],
  'rules': {
  },
};
```

その他、あると良さそうな設定も追加しておく
```diff js: eslintrc.js
module.exports = {
  'env': {
    'browser': true,
    'es2021': true,
  },
  extends: [
+   'eslint:recommended',
+   'plugin:@typescript-eslint/eslint-recommended',
+   'plugin:@typescript-eslint/recommended',
    'plugin:react/recommended',
    'next/core-web-vitals',
    'google',
    'prettier',
  ],
  'parser': '@typescript-eslint/parser',
  'parserOptions': {
    'ecmaFeatures': {
      'jsx': true,
    },
    'ecmaVersion': 'latest',
    'sourceType': 'module',
  },
  'plugins': [
    'react',
    '@typescript-eslint',
  ],
  'rules': {
  },
}
```

## ESLint の設定 (2)
ESLint のプラグインを使って、 import 関連がいい感じになるように設定する
[eslint-plugin-import](https://github.com/import-js/eslint-plugin-import)をインストールする
```sh
yarn add -D eslint-plugin-import eslint-import-resolver-typescript @typescript-eslint/parser
```
`eslintrc.js` にプラグインとして追加する
```diff js : eslintrc.js
module.exports = {
  // 省略
  'plugins': [
    'react',
    '@typescript-eslint',
+    'import',
  ],
  // ...
};
```

`import order`関連の好きな設定を追加する
```diff js:eslintrc.js
'rules': {
+     'import/order': ['error', {
+     'newlines-between': 'always', // グループ間に空白行が入る
+     'alphabetize': {
+       'order': 'asc', // アルファベットの昇順に並び替えられる
+     },
+   }],
  },
```
ここでは、importするモジュール群の間に空白行を入れる設定と、モジュールがアルファベット順に並び替えられる設定を行っている。
詳しくは [ドキュメント](https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/order.md) に記載がある

## Prettier の設定
コードフォーマットに Prettier を利用する。
ESLint にもコードをfixする機能はあるが、Prettierにはより細かいルールが多く、コードの見やすさと品質を保つために便利な機能が多い
もちろんeslint と競合しうるので、競合しないような設定にする必要がある

prettier と、eslint-config-prettier をインストールする
```sh
yarn add -D prettier eslint-config-prettier 
```
eslintrc.js の extends の **最後** に prettierを追加する
```diff js:eslintrc.js
  'extends': [
    'plugin:react/recommended',
    'google',
+   'prettier', // 最後に追加する
  ],
```
prettier の設定ファイルを作る
```sh
echo "module.exports = {}" > .prettierrc.js
```
好きな設定を追加する
```diff js:prettierrc.js
{
+ trailingComma: 'es5',
+ tabWidth: 2,
+ semi: false,
+ singleQuote: true,
}
```
## ESLint / Prettier 間の足並みを揃える
```js:eslintrc.js
  'rules': {
    'react/react-in-jsx-scope': 'off'
    'semi': ['error', 'never'],
    'require-jsdoc': ['off'],
    // ...
  },
```
ESLint ではgoogleのスタイルガイドを指定しているので、やりたい設定やPrettierで指定した設定と競合してしまうことがある
Prettier で指定したものに ESLint のルールを合わせ、足並みを揃えておく

## VSCode 用の設定
保存時の自動フォーマットや、プロジェクトで利用を推奨する拡張機能を設定しておく

VSCode でファイルを保存したときに自動でフォーマットされるようにする
```json:.vscode/settings.json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
    "source.fixAll": true
  },
  "editor.formatOnSave": true
}

```
プロジェクト用に、推奨の拡張機能を設定する

```json:.vscode/extensions.json
{
  "recommendations": [
    "dbaeumer.vscode-eslint", 
    "esbenp.prettier-vscode"  
  ]
}
``` 

ここでは、`ESLint` と `Prettier` の拡張機能を指定している。
::: details 追加のやりかた
1. VSCode で入れておいてほしい拡張機能を開き、`Cmd Shift P`でコマンドパレットを開く
1. `Extensions: Add Extension to Workspace Folder Recommendations` を選択する

`.vscode/extensions.json` にreccomendationsが追加される
:::

## git commit 時に自動で lint / format される設定
husky および lint-staged を利用する
インストール
```sh
yarn add -D husky lint-staged
```
初期設定
```sh
yarn husky install
```
設定を追加
```sh
yarn husky add .husky/pre-commit "yarn lint-staged"  
```
`.lintstagedrc.json` を作成し、以下のようにする
```json:.lintstagedrc.json
{
  "*.{ts,tsx}": "eslint --fix",
  "*.{ts.tsx,json,md}": "prettier --write"
}
```

これでコミット時にフォーマットされるようになる

## EditorConfig の設定
[EditorConfig](https://editorconfig.org/) を利用すると、改行コードや文字コード、インデントのスタイルなどを指定できる。
VSCode に限らず あらゆるエディタで EditorConfig のプラグインを使えば、統一したスタイルを利用できる。([プラグイン一覧](https://editorconfig.org/#download))

```:.editorconfig
root = true 

[**/*.{js,jsx,ts,tsx,css,html,md,yml,yaml,json}]
end_of_line = lf
insert_final_newline = true
charset = utf-8
indent_style = space
indent_size = 2
```
VSCode ユーザのために、EditorConfig の拡張機能を追加しておく
```diff json:.vscode/extensions.json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
+   "editorconfig.editorconfig"
  ]
}
```
---

以上です！！！ 作成したリポジトリは Public Template なのでそのまま使えます。
よければお試しください！
https://github.com/lemonadern/template-nextjs-typescript