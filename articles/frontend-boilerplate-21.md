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

## ログイン機能を実装

ログイン機能を実装する。
今回は Firebase Authentication の Twitter ログインを実装する。

### Firebase Authentication

ココらへんを参考にする。
[Firebase Authentication](https://firebase.google.com/docs/auth)
[Twitter Login](https://firebase.google.com/docs/auth/web/twitter-login)

### .env の作成

プロジェクトルートに `.env` ファイルを作成。

```.env:.env
FIREBASE_API_KEY=
FIREBASE_AUTH_DOMAIN=
FIREBASE_PROJECT_ID=
FIREBASE_STORAGE_BUCKET=
FIREBASE_MESSAGING_SENDER_ID=
FIREBASE_APP_ID=
FIREBASE_MEASUREMENT_ID=
```

※ Firebase Console > プロジェクトの設定 > マイアプリ > ウェブアプリ > Firebase SDK snipet > 構成 を参考に作成。

### firebase, dotenv を追加

Firebase Authentication を利用するために Firebase SDK を追加します。
また、firebaseConfig は環境変数で指定したかったので、dotenv も追加します。

以下のコマンドを実行

```
$ yarn add firebase
```

```
$ yarn add -D dotenv
```

### auth/useLogin.ts の実装

認証、ログインをするために以下のファイルを作成。
前述の通り今回は Twitter Login となります。

```ts:auth/useLogin.ts
import { useState, useEffect, useCallback, createContext } from 'react'
import Router from 'next/router'
import firebase from 'firebase/app'

type ReturnVoidFunc = () => void

type Result = [
  /** 認証が完了したかどうか */
  boolean,
  /** ユーザ情報 */
  firebase.User | null,
  /** ログイン用の関数 */
  ReturnVoidFunc
]

type Context = {
  user: firebase.User | null
  login: ReturnVoidFunc | null
}

const defaultValue = {
  user: null,
  login: null
}

export const UserContext = createContext<Context>(defaultValue)

export const useLogin = (): Result => {
  const [checked, setChecked] = useState(false)
  const [user, setUser] = useState<firebase.User | null>(null)

  const execLogin = useCallback(() => {
    ;(async () => {
      const provider = new firebase.auth.TwitterAuthProvider()
      try {
        await firebase.auth().signInWithPopup(provider)
        const user = firebase.auth().currentUser
        console.log('user:', user)
        if (user) {
          await Router.push('/')
        }
      } catch (e) {
        console.log('err:', e)
      }
    })()
  }, [])

  useEffect(() => {
    firebase.auth().onAuthStateChanged(async user => {
      if (user) {
        setUser(user)
        console.log('useEffect user:', user)
      } else {
        await Router.push('/login')
      }
      setChecked(true)
    })
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [])

  return [checked, user, execLogin]
}
```

※ execLogin や useEffect 内の認証に関してはアプリケーションに応じて変更する。

### src/pages/\_app.tsx の実装

認証を行うために、 `_app.tsx` を実装します。

```tsx:src/pages/_app.tsx
import React, { FC } from 'react'
import { AppProps } from 'next/app'
import Head from 'next/head'
import getConfig from 'next/config'
import firebase from 'firebase/app'
import 'firebase/auth'
// import 'firebase/firestore'
// import 'firebase/analytics'

import { useLogin, UserContext } from '../auth/useLogin'

const firebaseConfig = getConfig()?.publicRuntimeConfig?.firebase?.config

if (firebase.apps.length === 0) {
  firebase.initializeApp(firebaseConfig)
}

const App: FC<AppProps> = ({ Component }) => {
  const [authStateChecked, user, login] = useLogin()

  if (!authStateChecked) {
    return <p>...loading</p>
  }

  return (
    <>
      <Head>
        <title>my next app</title>
      </Head>
      {process.browser && (
        <UserContext.Provider
          value={{
            user,
            login
          }}
        >
          <Component />
        </UserContext.Provider>
      )}
    </>
  )
}

export default App
```

[参考: Next.js Custom App](https://nextjs.org/docs/advanced-features/custom-app)

### ユーザ情報を参照する

useContext を利用し、ユーザ情報を任意のコンポーネントで利用することができます。

```tsx:src/pages/index.tsx
const Home: FC = () => {
// ...省略
  const user = useContext(UserContext)
// ...省略
  return (
    <>
      {/* 省略 */}
    </>
  )
}
```

### 差分

https://github.com/MasahikoJinno/fe-boilerplate-21/pull/6

## data fetch

## testing

## CI/CD

## usage
