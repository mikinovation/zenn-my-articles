---
title: "Nuxt3で実践するFSDアーキテクチャ"
emoji: "🍀"
type: "tech"
topics: ["frontend", "vue", "Nuxt"]
published: false
---

# はじめに

## 対象読者

- 中大規模のフロントエンドプロジェクトに携わる・携わっている方
- Nuxt3におけるアーキテクチャ構成に迷っている方
- その他フレームワークでフロントエンド設計考えている方

## 前提知識

### クリーンアーキテクチャ

- レイヤードアーキテクチャであること
- 各レイヤーの責務を明確に分けること

### DDD

- ドメイン駆動であること

- クリーンアーキテクチャやDDDをそのままフロントに適用するのは過剰
- フロントに合わせた設計を考える必要がある

## FSDアーキテクチャとは

## 6つのレイヤー

- shared
- entities
- features
- widgets
- pages
- processes
- app

この6つのレイヤーで構成されている

ただしFSDアーキテクチャの中でも言われているように全てこの通りにプロジェクトの起ち上げから実装する必要はない

このレイヤーを私の使いやすいと思う構成に合わせてアレンジしたいと思う。もちろんNext.jsやAngular等、その他のフレームワーク、ライブラリに合わせて構成をアレンジしてほしい

# Nuxt3を活用したディレクトリ構成

ここからはFSDアーキテクチャが提唱しているものではなく、私個人でNuxt3で中大規模のプロジェクトの構成を考えたものだ

```
- src
  ├ shared
  │  ├ 
  ├ entities
  ├ features
  ├ pages
  ├ middleware
  ├ layouts
  ├ plugins
  └ app.vue
```

## shared

- config
- infrastructure
- ui

## entities

- {sliceName}/todo.ts
- {sliceName}/todoId.ts
- {sliceName}/status.ts

## features

- {sliceName}/components
- {sliceName}/composables
- {sliceName}/schema
- {sliceName}/utils
- {sliceName}/types

slice名以下は必要なディレクトリを適宜増やしたり、減らしてしてほしい

## pages

## middleware

## app

## その他のディレクトリ

- layouts
- plugins
- public
- assets

- app.vueに集約すること

# テスト

本筋とは逸れるかもしないが、堅牢な設計を保つためにはテストが必須であると筆者は考えている

理由としてはテストをしやすい設計を考えることでコンポーネントをよりシンプルに保つよう意識することができるからだ



## ユニットテスト

- vitestを活用する
- schemaのテスト

## インタラクションテスト

- Storybookを活用する
- インタラクションのテスト
- スナップショットのテスト

## E2Eテスト

- playwrite/cypress
- スナップショット
- 成功パターンのみテスト
- ページタイトルのテスト

## テストファイルをどこに配置するか

- 単体テストは各tsファイルの隣(hogehogeSchema.spec.ts)
- インタラクションテストは(Fugafuga.stories.ts)
- playwriteはtestディレクトリ中にフラットに置く

# まとめ
