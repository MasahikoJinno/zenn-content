---
title: "App Design and Layout Working with UI controlsをやってみた"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["SwiftUI", "Swift"]
published: true
---

## 概要

[これ](https://developer.apple.com/tutorials/swiftui/working-with-ui-controls) をやってみた感想。
自分用学習メモ。

## Session1

- static let \`default\`
  - バッククオートはなんだ？
  - default が予約語とか？
- `Identifiable`
  - id が必須になる？
- `Text("Goal Date: ") + Text(profile.goalDate, style: .date)`
  - `+` 演算子使えるの...？
- Badge はなんでスケールしたのだろうか
- `Divider()` を追加すると中央揃えが左揃えになった
  - Divider を入れると幅が中身の成り行き（Hug）から親?要素に準ずる（Fill）になるっぽい
- `hueRotation`
- toolbar
  - すごいね

## Session2

特に新しい情報はなし

## Session3

- TextField, Picker, DatePicker など入力系の UI も充実してそう

## Session4

- `draftProfile` が入力データ（永続化=確定していない一時データ）
  - `onAppear` で永続データを draftProfile に入れる
    - `onAppear` は View が表示されるタイミングで実行される
  - `onDisappear` で入力データを modelData.profile に代入（入力したデータの永続化）
    - `onDisappear`は View が非表示になるタイミングで実行される
    - Cancel タップ時は`onDisappear` 実行前に `draftProfile` に `modelData.profile` を代入して入力した値をリセットしている
- 微妙にタイミングが違う気がするが、React でいう `useEffect` 的な使い方ができそう
  - Component が Render（mount）されるタイミングでデータを fetch するなど

## 所感

- 入力フォームの作り方がわかった
- 非同期処理など
