---
title: "ざっくりと理解するPrisma"
emoji: "😜"
type: "tech"
topics: ["Prisma"]
published: true
---

:::message
当記事ではPrismaの使い方については解説しておりません
なぜPrismaの特徴となぜPrismaが生まれたのかという背景を雑にまとめています
:::

# Prismaとは

次世代ORM。それではこれまでのORMと何が違うのか

## Prismaが達成したい8つの目標

https://www.prisma.io/docs/concepts/overview/why-prisma#tldr

### リレーショナルデータをマッピングする代わりにオブジェクトで考える

```prisma
model Post {
  id        Int     @id @default(autoincrement())
  title     String
  content   String?
  published Boolean @default(false)
  author    User?   @relation(fields: [authorId], references: [id])
  authorId  Int?
}

model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}
```

スキーマ定義するだけでモデルオブジェクトが使えるようになる
型も効いてる。何これやばい！

```js
const allUsers = await prisma.user.findMany()
```

### 複雑なモデルオブジェクトを回避するためのクラスでなくクエリ

例えばRails。次から次へとメソッド増やすな。mixin入れるな

バリデーションも次から次へと仕様変更で複雑化

モデルがどんどんファットになり、ロジックは肥大化

モデルの分割も簡単にできなくて困る

コレやめようよというわけだ

```ruby
class Product < ApplicationRecord
  include ...
  
  validate ...
  validate ...

  def method1
    ...
  end
  
  def method2
    ...
  end
  
  def method3
    ...
  end
end
```

### データベースとアプリケーションモデルの信頼できる唯一の情報源

さっきのスキーマとして定義したモデルこそが真である

railsの例みたいにごちゃごちゃとモデルの定義を付け足すことは許さないよ

### 一般的な落とし穴やアンチパターンを防ぐ健全な制約

これも先程と同じ

モデルがファットになったり、下手な抽象化に複雑化する心配がなくなるよね

### An abstraction that makes the right thing easy（「継承の落とし穴」）

(これだけは自分の英語力だと日本語訳できない・・・。正しく抽象化をできるようにする？？？)

とりあえずオブジェクト指向による継承は悪だと最近認知されてきた

下手な継承はやめようよという意図だと思う

### コンパイル時に保証される型安全なデータベースクエリ

スキーマと型は同一。Prismaオブジェクトの型が保証されている
サイコー

### ボイラープレートが少ないため、開発者はアプリの重要な部分に集中できる

管理しなければならないファイル数やコードが少ない。

コードの質を維持できるので、本質的なアプリケーションの開発に集中できる

### ドキュメントを検索する代わりに、コードエディタでオートコンプリート

型が定義されているので、一度基本的な使い方を覚えてしまえば開発者はわざわざPrismaのドキュメントを見にいく必要もないよね

## これまでのSQL、ORMとその他データベースツールの問題

### 生のSQL

**メリット**

- 開発者がすべてSQLをコントロールできる。

**デメリット**

- 開発の生産性が低くなる
- クエリ結果に型安全を得られない

### SQLクエリビルダー

**メリット**

- ほとんど開発者がSQLをコントロールできる

**デメリット**

- 生産性は生のSQLほどではないが、基本的には低い
- SQLの観点からデータについて考える必要が出てくる

### SQLクエリビルダー

**メリット**

- ほとんど開発者がSQLをコントロールできる

**デメリット**

- 生産性は生のSQLほどではないが、基本的には低い
- SQLの観点からデータについて考える必要が出てくる

### ORMクエリ

**メリット**

- 開発者がSQLをコントロールする必要はない
- 生産性も高い

**デメリット**

(本題はココ)

ORMは最初いいかもしれないけど、時間が経つにつれてより複雑になる

するとORMが何をやっているのか目的が全く見えなくなる

- 開発者のメンタルモデル→オブジェクトのモデル
- SQLのメンタルモデル→テーブル

このズレが開発者に対してストレスをあてている

具体的なことでいうと「N+1問題」

そこで**Prisma**の出番

「開発者がSQLをオブジェクトとして捉えられればいいんじゃね？」

ということでPrismaのようなORMが生まれました