---
title: "Next.jsのプロジェクトを作成するときにやってることメモ（基本編）"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "typescript", "nextjs", "react"]
published: true
---

## 概要

Next.js のプロジェクトを作成するときの手順いつも忘れるので自分用にメモる

## どういうプロジェクトを作成するのか？

- 作成時点で最新の Next.js を利用する
- next export を利用し、SSR を一切しない
  - firebase hosting などの hosting サービスだけを利用して動かす想定
- Github Actions による CI が設定されている

## 環境

```
~ ❯❯❯ sw_vers
ProductName:	macOS
ProductVersion:	12.4
BuildVersion:	21F5071b
~ ❯❯❯ node -v
v16.13.1
~ ❯❯❯ npm -v
8.3.0
~ ❯❯❯ yarn -v
1.22.17
~ ❯❯❯
```

記事を書いてる時点のバージョンは以下の通り

```json:package.json
  "dependencies": {
    "next": "12.1.6",
    "react": "18.1.0",
    "react-dom": "18.1.0"
  },
  "devDependencies": {
    "@testing-library/jest-dom": "^5.16.4",
    "@testing-library/react": "^13.2.0",
    "@types/node": "17.0.31",
    "@types/react": "18.0.8",
    "@types/react-dom": "18.0.3",
    "eslint": "8.14.0",
    "eslint-config-next": "12.1.6",
    "eslint-config-prettier": "^8.5.0",
    "eslint-plugin-prettier": "^4.0.0",
    "husky": "^7.0.0",
    "jest": "^28.1.0",
    "jest-environment-jsdom": "^28.1.0",
    "lint-staged": "^12.4.1",
    "prettier": "^2.6.2",
    "typescript": "4.6.4"
  },
```

## create-next-app

[公式ドキュメント](https://nextjs.org/docs/api-reference/create-next-app) を参考に Next.js のプロジェクトを作成。

```sh
yarn create next-app ./ --typescript
```

※ `./` でカレントディレクトリにプロジェクトを作成できる

## 必要なパッケージをインストール

create-next-app だけでは必要最低限のパッケージしかインストールされないので、色々とインストールしていく。

### インストールするパッケージ

- lint まわり
  - eslint-config-prettier
  - eslint-plugin-prettier
  - prettier
  - husky
  - lint-staged
- test まわり
  - @testing-library/react
  - @testing-library/jest-dom
  - jest
  - jest-environment-jsdom

```sh
yarn add -D eslint-config-prettier eslint-plugin-prettier prettier husky lint-staged @testing-library/react @testing-library/jest-dom jest jest-environment-jsdom
```

## husky / lint-staged の設定

### husky-init の実行

便利コマンドがあるので実行する

```sh
npx husky-init
```

### pre-commit の修正

最終行を lint-staged に修正する

```sh:.husky/pre-commit
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx lint-staged
```

### lint-staged の設定

package.json に以下の設定を追加する

```json
  "lint-staged": {
    "src/**/*.{ts,tsx}": [
      "eslint --fix",
      "git add"
    ]
  }
```

## eslint / prettier の設定

細かい設定は好みによる

### .eslintrc.json を修正する

```json:.eslintrc.json
{
  "extends": [
    "next/core-web-vitals",
    "prettier",
    "plugin:prettier/recommended",
    "plugin:react/recommended"
  ],
  "rules": {
    "prettier/prettier": "error",
    "react/prop-types": "off",
    "react/react-in-jsx-scope": "off",
    "react-hooks/exhaustive-deps": "warn",
    "no-param-reassign": [
      "error",
      {
        "props": true,
        "ignorePropertyModificationsFor": ["state"]
      }
    ],
    "no-image-element": "off",
    "@next/next/no-img-element": "off"
  },
  "ignorePatterns": ["**/__tests__/*.ts", "**/__mocks__/*.ts", "next-env.d.ts"],
  "settings": {
    "react": {
      "version": "detect"
    }
  }
}

```

### .prettierrc を作成

```json:.prettierrc
{
  "tabWidth": 2,
  "printWidth": 100,
  "trailingComma": "none",
  "semi": false,
  "singleQuote": true,
  "arrowParens": "avoid"
}
```

## ディレクトリ構造の変更

create-next-app で作成したプロジェクトだと、`pages/` がプロジェクト直下にあるので`src/`をかませる。
※ 自分の好みでアプリケーションの実行コードは `src/` ディレクトリに集約したい。

## jest の設定

### jest.config.js を作成する

[公式ドキュメント](https://nextjs.org/docs/testing#setting-up-jest-with-the-rust-compiler)を参考に`jest.config.js`を作成する。

```js:jest.config.js
const nextJest = require('next/jest')

const createJestConfig = nextJest({
  dir: './'
})

const customJestConfig = {
  moduleDirectories: ['node_modules', '<rootDir>/'],
  testEnvironment: 'jest-environment-jsdom',
  testMatch: ['**/__tests__/**/*.+(ts|tsx)', '**/(*.)+(spec|test).+(ts|tsx)'],
  // ToDo: 適宜修正する
  collectCoverageFrom: [
    'src/**/*.ts',
    'src/**/*.tsx',
    '!src/**/__tests__/*.ts',
    '!src/**/*.test.ts(x)',
    '!src/**/__stories__/*.ts(x)',
    '!src/types/**.d.ts',
    '!src/pages/**/*.ts(x)',
    '!src/utils/index.ts'
  ]
}

module.exports = createJestConfig(customJestConfig)
```

### package.json の修正

`.scripts`に以下を追加する。

```json:package.json
    "test": "jest",
    "test:w": "jest --watch",
    "test:c": "jest --coverage"
```

### 動作確認

適当なコードを書いて jest が正常に動くか確認する。

```ts:src/utils/numberWithCommas.ts
export const numberWithCommas = (number: number) => new Intl.NumberFormat().format(number)
```

```ts:src/utils/__tests__/numberWithCommas.ts
import { numberWithCommas } from '../numberWithCommas'

test.each`
  input      | output
  ${0}       | ${'0'}
  ${1}       | ${'1'}
  ${12}      | ${'12'}
  ${123}     | ${'123'}
  ${1234}    | ${'1,234'}
  ${12345}   | ${'12,345'}
  ${123456}  | ${'123,456'}
  ${1234567} | ${'1,234,567'}
`('$input が $output に変換される', ({ input, output }) => {
  expect(numberWithCommas(input)).toEqual(output)
})
```

```sh:結果
~❯❯❯ yarn test:c
yarn run v1.22.17
$ jest --coverage
 PASS  src/utils/__tests__/numberWithCommas.test.ts
  ✓ 0 が 0 に変換される (9 ms)
  ✓ 1 が 1 に変換される
  ✓ 12 が 12 に変換される (1 ms)
  ✓ 123 が 123 に変換される
  ✓ 1234 が 1,234 に変換される (1 ms)
  ✓ 12345 が 12,345 に変換される
  ✓ 123456 が 123,456 に変換される
  ✓ 1234567 が 1,234,567 に変換される

---------------------|---------|----------|---------|---------|-------------------
File                 | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
---------------------|---------|----------|---------|---------|-------------------
All files            |     100 |      100 |     100 |     100 |
 numberWithCommas.ts |     100 |      100 |     100 |     100 |
---------------------|---------|----------|---------|---------|-------------------
Test Suites: 1 passed, 1 total
Tests:       8 passed, 8 total
Snapshots:   0 total
Time:        0.352 s, estimated 1 s
Ran all test suites.
✨  Done in 0.90s.
~❯❯❯
```

## next export の設定

### nest.config.js の修正

next export の設定を追加する。

```js:next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  exportPathMap: async (_defaultPathMap, { _dev, _dir, _outDir, _distDir, _buildId }) => {
    return {
      '/': { page: '/' }
    }
  }
}

module.exports = nextConfig
```

### package.json の修正

`.scripts.build` を以下の通りに修正

```json:package.json
    "build": "next build && next export",
```

### 動作確認

`yarn dev` や `yarn build` などを実行し正常に動作するか確認する。

## CI の設定

PR に連動した CI を設定する。

### test

```yml: .github/workflows/test.yml
name: jest
on:
  pull_request:
    types: [opened, synchronize]
# slack通知をする場合
# env:
#  SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK_URL }}
jobs:
  jest:
    name: jest
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [16.x]
    steps:
      - uses: actions/checkout@v1
      - name: Setup NodeJs
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}
      - name: yarn install
        run: yarn
      - name: Run test
        run: yarn run test:c
      - name: Upload test coverage artifact
        uses: actions/upload-artifact@v1
        with:
          name: coverage
          path: coverage
        env:
          CI: true
      - name: display coverage
        uses: vebr/jest-lcov-reporter@v0.2.0
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          lcov-file: ./coverage/lcov.info
# slack通知をする場合
#      # テスト失敗時はこちらのステップが実行される
#      - name: Slack Notification on Failure
#        uses: rtCamp/action-slack-notify@v2.0.2
#        if: failure()
#        env:
#          SLACK_CHANNEL: front-end-git-notice
#          SLACK_TITLE: TEST Failure
#          SLACK_MESSAGE: 'TEST が失敗しましたので確認してください:eyes:'
#          SLACK_COLOR: danger
```

### lint ※ husky/lint-staged 設定しているけど一応

```yml: .github/workflows/lint.yml
name: lint
on:
  pull_request:
    types: [opened, synchronize]
# slack通知する場合
# env:
#  SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK_URL }}
jobs:
  eslint:
    name: lint
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [16.x]
    steps:
      - uses: actions/checkout@v1
      - name: Setup NodeJs
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}
      - name: yarn install
        run: yarn
      - name: eslint review
        uses: reviewdog/action-eslint@v1
        with:
          github_token: ${{ secrets.github_token }}
          reporter: github-pr-review
          eslint_flags: './**/*.{ts,tsx}'
      - name: eslint
        run: yarn eslint
      - name: Typecheck
        uses: andoshin11/typescript-error-reporter-action@v1.0.2
# slack通知する場合
#      # テスト失敗時はこちらのステップが実行される
#      - name: Slack Notification on Failure
#        uses: rtCamp/action-slack-notify@v2.0.2
#        if: failure()
#        env:
#          SLACK_CHANNEL: front-end-git-notice
#          SLACK_TITLE: Lint Failure
#          SLACK_MESSAGE: 'Lint エラーが出ているので確認してください:eyes:'
#          SLACK_COLOR: danger

```

### build

jest は通るけど、build が通らないみたいなことがまれにあるので build が通るかどうかも PR で確認する。

```yml: .github/workflows/lint.yml
name: build
on:
  pull_request:
    types: [opened, synchronize]
# slack通知する場合
# env:
#  SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK_URL }}
jobs:
  build:
    name: build
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [16.x]
    steps:
      - uses: actions/checkout@v1
      - name: Setup NodeJs
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}
      - name: yarn install
        run: yarn
      - name: Run build
        run: yarn run build
# slack通知する場合
#      # テスト失敗時はこちらのステップが実行される
#      - name: Slack Notification on Failure
#        uses: rtCamp/action-slack-notify@v2.0.2
#        if: failure()
#        env:
#          SLACK_CHANNEL: front-end-git-notice
#          SLACK_TITLE: BUILD Failure
#          SLACK_MESSAGE: 'BUILD が失敗しましたので確認してください:eyes:'
#          SLACK_COLOR: danger

```

## 最後に

今までの設定を行えば最低限の Next.js のアプリケーションを構築できる。
実際には上記以外にも、以下のような設定が必要になるので機会があれば手順を作成しておく。

## 私が実際のアプリケーションを構築する上でやっている設定

- State 管理 FW（Redux など）の設定
- storybook の設定
- デプロイ周りの設定

## Tips

以下のコマンドで独自のテンプレを利用した Next.js のプロジェクトが作成できる。
どんなプロジェクトでも使うよねってやつをボイラープレートとして作成しておくと、プロジェクトの立ち上げがスムーズになるかも？

```sh
yarn create next-app my-app -e https://github.com/xxx/yyy
```

※ github の場合、public のリポジトリにする必要があるので注意
※ 何故か `.scripts.prepare` が実行されてなかったので、プロジェクト直下で `yarn` をする必要がある
