---
title: "Nuxt3で「eslintrcとprettier」から「flat configとeslint stylistic」へ移行した話"
emoji: "📏"
type: "tech"
topics: ["Typescript", "Vue", "Nuxt", "Frontend"]
publication_name: gerunda
published: true
---

# はじめに

こんにちは、株式会社 Gerunda の齋藤です。

昨年 2023 年の 10 月に Flat config への移行に関するロードマップの発表からしばらく経ちました。

https://eslint.org/blog/2023/10/flat-config-rollout-plans/#main

2024 年 4 月に eslint の v9 がリリースされます。現在 flat config が標準となり、eslintrc は非推奨となりました。次の v10 では完全に eslintrc は削除され使えなくなる予定になっています。そこで今回 eslintrc から flat config へ移行したので、ここに記録を残すこととします

# flat config とは

そもそも eslint の目指す flat config とは何なのでしょうか。

eslint 公式の[The goals of flat config](https://eslint.org/blog/2022/08/new-config-system-part-2/#the-goals-of-flat-config)には以下のように書かれています

1. Logical defaults
2. One way to define configs
3. Rules configs shoould remain unchanged
4. Use native loading for everything
5. Better organized top-level keys
6. Existing plugins should work
7. Backwards compatibility should be a priority

eslint が flat config を目指す理由はいくつかあります。

ECMAScript による ES Modules が主流になりつつある状況で、開発者がよりシンプルにプラグインを設定できるというのが主な目的のようです。Typescript を活用することで静的な型解析により、保守性も高めることもできるようになります

また既存の flat config に対応できていないプラグインの資産も今まで通り使えるようになっており、後方互換性を意識した作りになっているようです。また ES Modules だけでなく、CommonJS での利用可能であり手厚いサポートがされてます。

既に flat config を詳しく説明するような記事は存在するので今回はこの辺で端折りますが、詳しく知りたい方は以下のような記事を参考にされると良いかと思います

まず公式のブログはこちら

https://eslint.org/blog/2022/08/new-config-system-part-2/

そして日本語の記事だと uhyo さんの記事が参考になります

https://zenn.dev/babel/articles/eslint-flat-config-for-babel

# Nuxt ESLint

それでは本題に入りましょう。今回は Nuxt で flat config の対応をしていきます

そこで活用するのが Nuxt の公式から提供されている ESLint Module です。

https://eslint.nuxt.com/

こちらは本当に素晴らしい Nuxt のモジュールです。個人的には lint の設定を可能な限りゼロコンフィグで使いたいと思っています。

> All-in-one ESLint integration for Nuxt. It generates a project-aware ESLint flat config and provides the ability to optionally run ESLint check along side the dev server.

とドキュメントにも書いているように Vue、Nuxt、JavaScript、TypeScript、ESLint Stylistic(後ほど解説する formatter)、JSDoc、Unicorn 等のカスタム済みルール全てが盛り込まれた最高なオールインワンパックです

Nuxt ESLint は以下の 3 つのパッケージから成り立っています

- @nuxt/eslint
- @nuxt/eslint-config
- @nuxt/eslint-plugin

## @nuxt/eslint

@nuxt/eslint は Nuxt のモジュールとして利用することができます。Nuxt と eslint を統合する機能を担っています

開発時にはランタイムで eslint の分析を行うことができる checker という機能や dev-tools から lint ルールを確認するための@eslint/config-inspector を利用できるようにした package です

## @nuxt/eslint-config

@nuxt/eslint-config は Nuxt ESLint のルールを集約した package です。各 eslint のルールのカスタマイズとそのルールを提供しています。具体的にどのような設定があるのか確認してみましょう

- configs
  - [eslint-plugin-vue](https://github.com/vuejs/eslint-plugin-vue)
  - [@typescript-eslint/parser](https://github.com/typescript-eslint/typescript-eslint/tree/main/packages/parseri)
  - [@typescript-eslint/eslint-plugin](https://github.com/typescript-eslint/typescript-eslint/tree/main/packages/eslint-plugin)
  - [@stylistic/eslint-plugin](https://github.com/eslint-stylistic/eslint-stylistic)
  - [@nuxt/eslint-plugin](https://github.com/nuxt/eslint/tree/main/packages/eslint-plugin)
  - [@eslint/js](https://github.com/eslint/eslint/tree/main/packages/js)
  - [eslint-plugin-import-x](https://github.com/un-ts/eslint-plugin-import-x)
  - ignores
    - dist や node_modules 等の ignore
  - disables
    - vue/multi-word-component-names を一部無効化するための処理(pages や layouts は 1word のコンポーネントで問題ない)
- configs-tooling
  - [eslint-plugin-jsdoc](https://www.npmjs.com/package/eslint-plugin-jsdoc)
  - [eslint-plugin-unicorn](https://github.com/sindresorhus/eslint-plugin-unicorn)

今回はカスタム内容まで確認しませんが、それぞれどのようにカスタマイズされているか気になる方は以下のリンクから確認できます。

https://github.com/nuxt/eslint/tree/main/packages/eslint-config/src/flat

## @nuxt/eslint-plugin

最後に@nuxt/eslint-plugin です。@nuxt/eslint-config にも出てきましたが、Nuxt 固有の lint ルールが集約されています

といっても 2024 年 4 月 30 日現在は`prefer-import-meta`のみです。

```
Replace `process.{{ suffix }}` with `import.meta.{{ suffix }}`.
```

このメッセージの通り、例えば`process.env`というコードを書くと`import.meta.env`から呼び出しましょうというエラーが発生するようになります

# 従来までの Nuxt と ES Lint

念のため従来までの Nuxt と ESLint はどうだったかのお話もしたいと思います。以下は nuxtjs で使用されていた package です。つまり Nuxt2 の頃のモジュールですね。この Nuxt2 のモジュールが拡張されて Nuxt3 に対応するという形で運用されていました

JS であれば[@nuxtjs/eslint-config](https://eslint.nuxt.com/legacy/eslint-config)、TS であれば[@nuxtjs/eslint-config-typescript](https://eslint.nuxt.com/legacy/eslint-config-ts)というモジュールを利用することで似たような機能が利用できました

https://github.com/nuxt/eslint/blob/main/packages-legacy/nuxt2-eslint-config/index.js

https://github.com/nuxt/eslint/blob/main/packages-legacy/nuxt2-eslint-config-typescript/index.js

コードを読んでいただいても分かるように比較的シンプルな eslintrc だったことが分かります。しかし eslint の flat config に対応することでよりシンプルに拡張性の高い設定が書けるようになりました。

また nuxt + eslint でググると nuxt の eslint module を利用せずに vue3 や typescript の eslint プラグインを個別にインポートして利用するという記事をよく見かけます。もちろん自分で一からカスタマイズしたいということであれば、その方法も問題ありません。しかし自分のようにプラグインの管理を Nuxt ESLint に丸投げして lint の管理コストは減らす方向で運用したいのであれば Nuxt ESLint を活用した方が良いのではないかと思っています。

# Prettier から ESLint Stylistic に乗り換える

ここまで eslint についてお話してきましたが、この記事のタイトルにもある通り Prettier と ESLint Stylistic にも触れます

Prettier といえば今最も人気のある Javascript のフォーマッターです

https://prettier.io/

しかし私はこの prettier から ESLint Stylistic に乗り換えることにしました

https://eslint.style/

ESLint Stylistic は`eslint`、`@typescript-eslint/eslint-plugin`、`eslint-plugin-react`からフォーマットに関するルールを分離したモジュールです

その中にも Linter vs Formatter に関する意見が述べられています。簡単に要約すると以下のようなことが述べられています

- Formatter は Linter から分離されるべきである
- Prettier や dprint の"read-and-reprints"というアプローチはスタイルによる情報(意味のある改行等)を捨て去ってしまう
  - 人間にとって読みやすいコードとは何かを考える必要がある
  - 例として Prettier や dprint の printWidth のような強力な改行ルールが挙げられており、それは人間にとっての可読性を落とすことになる

というような主張がなされています。先日話題になった以下の記事も記憶に新しいです

https://zenn.dev/praha/articles/7ed629d0d9da53

上記記事の主張としてもコードは機械的に幅を決めて改行するのではなく、意味で改行したいという考えが述べられていました。ただ私は prettier に対して特に不満があったわけではありませんし、Stylictic が完全に Prettier を代替できるものかというとそうではないと思っています

Stylistic へ移行する決め手としては、やはりゼロコンフィグで、よりシンプルに設定できるところです。移行できた理由として、今私が受け持っているプロジェクトに関してもフロントエンドは私一人で保守・管理しており、技術選定もほぼ私が決定権を持てていたということも大きな要因かと思っています

Nuxt ESLint でも公式に採用されているためとても簡単に設定ができます

```ts:nuxt.config.ts
export default defineNuxtConfig({
  modules: [
    '@nuxt/eslint'
  ],
  eslint: {
    config: {
      stylistic: true
    }
  }
})
```

設定はこれだけです。もし stylistic のルールを個別でカスタマイズしようと思ったら以下のように書くことができます

```ts:nuxt.config.ts
export default defineNuxtConfig({
  modules: ["@nuxt/eslint"],
  eslint: {
    config: {
      stylistic: {
        indent: "tab",
        semi: true,
        // ...
      },
    },
  },
});
```

もし Prettier を入れようと思ったら、eslint-config-prettier のような干渉を調整するためのプラグインも管理する必要があります。それであればいっそ stylistic だけでも良さそうです

もちろんチームで話しあったうえで prettier に必要なルールがほしいのであれば、prettier を採用するのは全く問題ありません。実際 Nuxt ESLint でも prettier が非推奨になっているわけではありません。ドキュメントにも「Prettier を入れたかったら設定してね」としか書いていないので、Prettier を採用しても全く問題ないということです。

https://eslint.nuxt.com/packages/module#prettier

私のようにフォーマットの形式自体にこだわりはないが、軽めにフォーマットは統一しておきたいというだけであれば、Stylistic に任せるという選択肢はありかなと思っています

:::message
おまけの理由: 私は普段Vimを使っているのでVimで編集しやすいように改行できると嬉しいです。コードはカーソル移動をするための足場だと思っています
:::

# Nuxt で eslintrc から flat config に移行してみる

それでは実際に移行の手順をご紹介します。今回のケースは Nuxt が 1 プロジェクトもしくはモジュラーモノリスのケースになります。手順自体はかなり簡単です

1. `.eslintrc.js`、`.eslintignore`、`.prettierrc`、`.prettierignore`を削除する

元ファイルはこんな感じ

```js:.eslintrc.js
module.exports = {
  env: {
    es2021: true,
    node: true,
    browser: true
  },
  extends: [
    'plugin:vue/vue3-recommended',
    'eslint:recommended',
    '@nuxtjs/eslint-config-typescript',
    '@vue/prettier',
     ...
  ],
  overrides: [],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module'
  },
  plugins: ['vue', '@typescript-eslint'],
  rules: {}
}
```

```.eslintignore
node_modules
*.log*
**/.nuxt
.nitro
.cache
src/.output
.env
dist
.idea
.vscode
...
```

2. package.json から eslint と prettier 関連のプラグインを削除する
3. `npx nuxi module add eslint`を実行
4. eslint を v9 にバージョンアップ

最低限必要なモジュールは eslint と@nuxt/eslint のみ

5. nuxt.config.ts で stylistic を true にする

```ts
export default defineNuxtConfig({
  modules: ["@nuxt/eslint"],
  eslint: {
    config: {
      stylistic: true,
    },
  },
});
```

6. eslint.config.mjs の編集

```js:eslint.config.mjs
// @ts-check
import globals from "globals"
import withNuxt from "./.nuxt/eslint.config.mjs";

export default withNuxt({
  globals: {
    ...globals.browser,
    ...globals.node
  },
  rules: {
    // TODO: @nuxt/eslintがpageディレクトリに対するignoreを対応したら有効化する
    "vue/multi-word-component-names": "off",
  },
  ignores: [
    ".idea",
    ".vscode",
    "/src/public",
    ...
  ],
});
```

ルールで`vue/multi-word-component-names`がオフにしています。理由としては現時点で pages ディレクトリにエラーが出てしまっていました(バグ？)。なので応急処置としていったん off にし、対応されたらルールを外そうと思っています

また eslint や prettier で Nuxt ESLint の設定に含まれていないプラグインを設定されている方はそれぞれ追加で設定しましょう。

eslint を無効化する ignores も設定しましょう。Nuxt ESLint にてデフォルトで指定されている ignores は以下を参照してください

https://github.com/nuxt/eslint/blob/main/packages/eslint-config/src/flat/configs/ignores.ts

ついでに global も設定しています。ECMAScript のバージョン等も指定があれば適宜設定してください

7. 最後にフォーマットを掛け直して、エラーを修正したら移行完了

非常に簡単ですね。これで移行作業は完了になります

# まとめ

それでは皆さん、、、ふらっと config 設定してみましょう！
