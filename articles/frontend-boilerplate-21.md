---
title: "自分用の Web フロントエンドボイラープレートを作成する（2021 SPRING）" # 記事のタイトル
emoji: "💪" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["javascript", "typescript", "firebase", "nextjs", "react"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

## 概要

自分用の Web フロントエンドボイラープレートを作成する。
※ 作成中

### 利用する(予定の)技術

- [Firebase](https://firebase.google.com/)
  - platform
  - [Firebase Hosting](https://firebase.google.com/products/hosting)
    - web hosting
  - [Cloud Firestore](https://firebase.google.com/products/firestore)
    - NoSQL data store
  - [Firebase Authentication](https://firebase.google.com/products/auth)
    - IDaaS
- [TypeScript](https://www.typescriptlang.org/)
  - type system
- [Next.js](https://nextjs.org/)
  - framework
  - [Static HTML Export](https://nextjs.org/docs/advanced-features/static-html-export)
- [React](https://reactjs.org/)
  - ui library
- [useSWR](https://swr.vercel.app/)
  - data fetching

### リポジトリ

https://github.com/MasahikoJinno/fe-boilerplate-21

## Create Next App

[参考]
https://nextjs.org/docs/api-reference/create-next-app

### 以下のコマンドを実行

```sh
npx create-next-app ./
```

### 動作確認

```sh
yarn dev
```

## 不要なファイルを削除

https://github.com/MasahikoJinno/fe-boilerplate-21/pull/1

## TypeScript の導入

[参考]
https://nextjs.org/docs/basic-features/typescript

### 以下のコマンドを実行

```sh
yarn add -D typescript @types/node @types/next @types/react @types/react-dom
```

### 設定ファイルを追加

- next-env.d.ts
- tsconfig.json

※ 詳しくは差分を確認

### TypeScript で書き直す

- src/pages/index.js → src/pages/index.tsx

※ 詳しくは差分を確認

### 差分

https://github.com/MasahikoJinno/fe-boilerplate-21/pull/2

## eslint/prettier の設定

### 以下のコマンドを実行

```sh
yarn add -D @typescript-eslint/eslint-plugin @typescript-eslint/parser eslint eslint-config-prettier eslint-plugin-prettier eslint-plugin-react eslint-plugin-react-hooks prettier husky lint-staged
```

### eslint, prettier の設定ファイルを追加

- .eslintrc
- .prettierrc

### package.json に以下を追加

commit 前に eslint（prettier）の検証/修正を入れる。

```json
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "src/**/*.{ts,tsx}": "eslint --fix"
  }
```

### 差分

https://github.com/MasahikoJinno/fe-boilerplate-21/pull/3

## Firebase, Static HTML Export の設定

### 以下のコマンドを実行

```sh
yarn add -D firebase-tools
```

### firebase の設定

- firebase の project を作成
- firebase init

[参考]
https://qiita.com/Miracle-T-Shirt09/items/8ac43ab7a18b3001dda0

### next.config.js の作成

```js
module.exports = {
  /**
   * Static HTML Export setting
   * https://nextjs.org/docs/api-reference/next.config.js/exportPathMap
   */
  exportPathMap: async (
    _defaultPathMap,
    { _dev, _dir, _outDir, _distDir, _buildId }
  ) => {
    return {
      "/": { page: "/" },
    };
  },
};
```

### package.json の修正

```json
    "build": "next build && next export",
    "deploy": "yarn build && firebase deploy",
```

### deploy の確認

```sh
yarn deploy
```

## ボイラープレート部分の実装

以降鋭意作成中

## routing

### 以下のファイルを作成

- src/pages/about.tsx ※ 静的パス
- src/pages/dynamic/[id].tsx ※ 動的パス（id が動的になる）
- src/routing/route.ts ※ アプリケーションのパス定義
- src/routing/isCSR.ts ※ nextjs のサーバが動かない状態でも動的なパスに遷移できるようにする hooks

### src/pages/index.tsx の修正

useCSR を利用する。
CSR（Client Side Routing）が終了するまではローディングのコンポーネントを表示。

```tsx
const execRouting = useCSR();

if (!execRouting) {
  return <p>loading...</p>;
}
```

### 動作確認

#### 開発環境での動作確認

```sh
yarn dev
```

#### ホスティングサーバでの動作確認

```sh
yarn deploy
```

### 差分

https://github.com/MasahikoJinno/fe-boilerplate-21/pull/5

## auth

## data fetch

## testing

## CI/CD

## usage
