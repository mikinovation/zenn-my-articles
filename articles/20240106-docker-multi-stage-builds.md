---
title: "Dockerのマルチステージビルドで効率化！スマートなコンテナデプロイの実践ガイド"
emoji: "🐳"
type: "tech"
topics: ["Docker", "Security", "Node.js"]
publication_name: gerunda
published: false
---

# はじめに: マルチステージビルドの重要性

こんにちは、株式会社 Gerunda の齋藤です。

私は普段フロントエンジニアとしてお仕事をしています。最近のフロントエンドの開発環境は非常に多様化しており、Vercel、Netlify、Cloudflare、AWS、GCP、Azure、さらにはオンプレミス環境といった様々なデプロイ先の選択肢があります。

このような環境の中、特にクラウドサービスを利用した際に Docker を用いたデプロイが非常に重要になってきています。今回は、私が使用した Nuxt3 を用いたアプリケーションを例に、Docker のマルチステージビルドを活用したコンテナデプロイの手法を解説します。

Docker コンテナによるデプロイは開発環境と本番環境のイメージを一つの Dockerfile で効率的に管理し、デプロイプロセスを最適化するための強力な方法です。それに加えて Docker のマルチステージビルドにはいくつかのメリットがあります

この章では、Docker の基本的な概念から始め、マルチステージビルドの具体的な利点とその実装方法について掘り下げていきます。フロントエンドのみでなく、バックエンドも含めて、これらの知識がどのように役立つか、実践的な視点からご紹介していきます。

## Docker の基礎

Docker はアプリケーションとその依存関係をコンテナと呼ばれる隔離された環境にパッケージ化するためのプラットフォームです。このプラットフォームを使用することで、アプリケーションが異なる環境間で一貫した動作を保証します。

ここではコンテナ、イメージ、Dockerfile といった簡単に用語だけ抑えておきましょう

- コンテナ: 軽量で、移植可能、かつ自己完結型のパッケージです。OS レベルの仮想化を提供し、アプリケーションを実行するために必要なすべて（コード、ランタイム、システムツール、ライブラリ、設定）を含みます。
- イメージ: コンテナを実行するためのテンプレートです。イメージはアプリケーションの実行に必要なファイルと設定のスナップショットになります。
- Dockerfile: コンテナイメージを構築するためのレシピのようなものです。Dockerfile では、基本となるイメージの指定、必要なファイルの追加、コマンドの実行などを定義します。

## マルチステージビルドの概要

マルチステージビルドは効率的かつセキュアなアプリケーションのデプロイにおいて不可欠です。

マルチステージビルドは、単一の Dockerfile 内で複数のビルドステージを持つことができる Docker の機能になります。そしてマルチステージビルドには以下 3 つの特徴があります

### ビルドステージの分離

開発用の依存関係やビルドツールは最終的なイメージに含める必要はありません。マルチステージビルドでは、これらのツールをビルドステージでのみ使用し、最終的なイメージからは排除できます

### イメージサイズの削減

必要なコンポーネントのみを最終的イメージに含めることで、イメージサイズを削減し、デプロイの効率を高めます

### セキュリティの向上

最終イメージに不要なライブラリやモジュールを排除することで、攻撃可能なリソースを最小限に抑えることができます

# 実践例: 効率的なイメージ構築

## フロントエンドアプリの事前準備

Nuxt3 のアプリを用意しました。
いろいろと省略してはいますが、ディレクトリ構成は以下のような構成のアプリになっています

```
demo-app/
├ docker/
|   └── Dockerfile
├ src/
|   ├── assets/
|   ├── entities/
|   ├── features/
|   ├── layouts/
|   ├── middleware/
|   ├── pages/
|   ├── plugins/
|   ├── store/
|   ├── app.vue
|   └── error.vue
├ README.md
├ docker-compose.yml
├ nuxt.config.ts
├ tsconfig.ts
└ yarn.lock
```

### nuxt.config.ts の設定

`nuxt.config.ts`には Nuxt の設定が含まれており、特に Nitro の proxy 設定に注目します。この設定は環境変数とプロキシの設定が正しくなされている必要があります。開発時は`devProxy`を使用しますが、プロダクションモードでは`routeRules`が提供される点に注意が必要です

```
// https://nuxt.com/docs/api/configuration/nuxt-config
import { defineNuxtConfig } from 'nuxt/config'

export default defineNuxtConfig({
  rootDir: 'src',
  nitro: {
    routeRules: {
      '/api/**': {
        proxy: process.env.BASE_API_URL
          ? `${process.env.BASE_API_URL}/**`
          : 'http://localhost:8000/api/**'
      },
    },
    devProxy: {
      '/api': {
        target: process.env.BASE_API_URL || 'http://localhost:8000/api',
        changeOrigin: true,
        prependPath: true
      },
    }
  },
  runtimeConfig: {
    public: {
      BASE_API_URL: process.env.BASE_API_URL || 'http://localhost:8000/api',
    }
  }
})
```

### マルチステージビルドの設定

マルチステージビルドの実装には、Dockerfile の適切な設定が必要です。以下に示すのは、今回したようした Dockerfile の例です

```Dockerfile
# ビルドステージ
FROM node:18.19.0-alpine3.18 as build-stage
WORKDIR /app
COPY . .
RUN yarn install
RUN yarn build

# 実行ステージ
FROM node:18.19.0-alpine3.18
WORKDIR /app

# ビルドステージから必要なファイルのみをコピー
COPY --from=build-stage /app/src/.output ./src/.output
COPY --from=build-stage /app/node_modules ./node_modules
COPY --from=build-stage /app/package.json ./package.json

EXPOSE 3000
CMD ["node", "src/.output/server/index.mjs"]
```

この Dockerfile では、Node の 18 系を基に、ビルドステージと実行ステージの 2 段階に分けています。最初にアプリケーションをビルドし、次に必要なファイルのみを実行ステージにコピーすることで、イメージのサイズを削減し、セキュリティを向上させています。

# まとめと今後の展望

## マルチステージビルドの効果の振り返り

この記事を通じて、マルチステージビルドを利用することで、フロントエンドアプリケーションのデプロイプロセスがどの様に変化するかを見てきました。主な利点は以下の通りです

- イメージサイズの削減
- セキュリの向上
- ビルドプロセスの高速化

これらの改善は、より効率的な開発フローとスムーズなデプロイを実現し、開発者の生産性を高めることに貢献してくれます

## Docker とフロントエンド開発の今後

Docker とマルチステージビルドの利用は、フロントエンド開発のみならず、多くのソフトウェア開発プロジェクトにおいて重要な役割を果たしています。今後は、以下のような分野での発展が期待されます

- クラウドネイティブ技術の更なる統合: Kubernetes やサーバーレスアーキテクチャとの統合を深め、デプロイの自動化と最適化をさらに推進します
- 開発と運用のさらなる一体化(DevOps): CI/CD パイプラインを通じて、ビルドからデプロイまでのプロセスを一層スムーズにする
- エコシステムの成熟: Docker 関連ツールやサービスの拡張により、より多様なニーズに応えるソリューションが提供されることが期待されます

## 最後に

マルチステージビルドを通じて、より効率的なビルドプロセスとデプロイを実現することができるようになりました。今後も Docker とフロントエンド技術の進化に注目し、それらを最大限に活用にしていきましょう
