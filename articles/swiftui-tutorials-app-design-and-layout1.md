---
title: "App Design and Layout Composing complex interfacesをやってみた"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["SwiftUI", "Swift"]
published: true
---

## 概要

[これ](https://developer.apple.com/tutorials/swiftui/composing-complex-interfaces) をやってみた感想。
自分用学習メモ。

## Session1

[以前の Chapter](https://developer.apple.com/tutorials/swiftui/creating-and-combining-views) で出てきた内容。
新しいものは特になし。

## Session2

- `enum`
  - `CaseIterable` は Rust の match っぽいことができるようになるのかな
  - いまだ `Codable` がわからない
- `Dictionary`

## Session3

- Step2: コンストラクタみたいなの無いのに引数が作られるのまだなれない
- Step4-5: `.leading` って段落の字下げか
  - padding 重ねるのちょっと気持ち悪い（css みたいに一気に指定できなくなさそうだが、調べてない）

## Session4

- Step1: `modelData.categories[key]!` の `!` ってなんだ？
  - array index out of bounds exception の抑制？
- Step5: `EdgeInsets()` 余白なくす的な？

## Session5

- UI のプロパティを覚えるの大変そう
- TabView べんり...

## 所感

- 相変わらず爆速で UI が作れるのはすごい
- ScrollView とか TabView とか予め UI が用意されてるので、配置とデータバインディングだけすれば良い感がある
  - その代わり Apple の UI の思想から外れたものを作るのはまあまあ大変そうではある
- Swift の学習を全くしてない状態でチュートリアルをやっているので、`enum` とか `Dictinary` とか新鮮
