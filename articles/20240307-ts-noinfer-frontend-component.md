---
title: "【Typescript5.4】NoInferを活用して型安全なUIコンポーネントを設計する"
emoji: "🧩"
type: "tech"
topics: ["Typescript", "Vue"]
publication_name: gerunda
published: false
---

# はじめに

株式会社 Gerunda の齋藤です。最近 Typescript の 5.4 がリリースされました

https://devblogs.microsoft.com/typescript/announcing-typescript-5-4/

その中でも注目度が高い機能が**NoInfer**という新しい Utility Type です。今回はフロントエンドのコンポーネント設計において、この NoInfer をどう活用できるのかを考えていきたいと思います。

# NoInfer とは

そもそも**NoInfer**とは何なのでしょうか。NoInfer は不要な型推論を抑制してくれる Utility Type です。言葉で説明してもピンとこないかと思うので、Typescript5.4 リリースノートに記載されているサンプルコードを読んでみましょう

```ts
function createStreetLight<C extends string>(colors: C[], defaultColor?: C) {
  ...
}

createStreetLight(["red", "yellow", "green"], "red");
```

どうやら信号機に関する関数のようです。信号機には「赤」「黄色」「緑」の三色で表現されます。ちなみに英語だと「青」ではなく「緑」と表現するので注意しましょう。少々話はそれますが、テストの際にも成功したテストに関しては「グリーンになった」、失敗したテストに関しては「レッドになった」というように表現しますね

本題に戻りましょう。この関数に typecheck を走らせるとどうなるでしょうか。もちろん通ります。

それでは以下のようにするとどうでしょうか

```ts
createStreetLight(["red", "yellow", "green"], "blue");
```

配列の要素に含まれていない`blue`という要素を第二引数の`defaultColor`に使用しました。実はこれで typecheck を走らせても問題なく通ってしまいます。これは困りました。配列に存在しない値はあらかじめ第二引数に指定できないようにした方がより安全なコードを書けるはずです。

それでは今まではどうしていたか。以下のように書くことで配列の要素に含まれている型のみを第二引数に指定できるようにしていました。

```ts
function createStreetLight<C extends string, D extends C>(colors: C[], defaultColor?: D) {
  ...
}
```

少しコードが複雑になりましたね。まあ読めないことはないですが、少しだけ型が複雑になったので可読性はどうしても落ちてしまいます。D という新しい変数が登場するのはスマートな解決策とは言えません。そこで今回登場した**NoInfer**を利用してみましょう

```ts
function createStreetLight<C extends string>(colors: C[], defaultColor?: NoInfer<C>) {
  ...
}
// Argument of type '"blue"' is not assignable to parameter of type '"red" | "yellow" | "green" | undefined'.
```

先程の D を新しく定義する方法に比べたらだいぶスッキリしました。NoInfer のお陰で**C が"blue"も含む可能性があるという型推論を防ぐ**ことができるようになります。これは嬉しいですね！

# フロントのコンポーネント設計に応用する

それでは NoInfer の旨味が分かったところで、これをどうフロントエンドコンポーネントの設計に活用するか考えてみましょう

今回は私が普段使用している Vue(Nuxt)を例にご紹介します。ただし、基本的には generics を使えるフレームワークであれば同じように NoInfer を扱えます。なので「自分は React ユーザーだよー」という方も参考にしてみてください

まずよくあるタブの UI を作成してみました。コードは[Nuxt UI](https://ui.nuxt.com/components/tabs)のサンプルをお借りします

https://ui.nuxt.com/components/tabs

```vue
<script setup lang="ts">
const items = [
  {
    label: "Tab1",
    content: "This is the content shown for Tab1",
  },
  {
    label: "Tab2",
    content: "And, this is the content for Tab2",
  },
  {
    label: "Tab3",
    content: "Finally, this is the content for Tab3",
  },
];
</script>

<template>
  <UTabs :items="items" />
</template>
```

このサンプルの UI をアプリケーション上で使いやすいようにリファクタリングしてみましょう。props からは `items` とデフォルトのラベルを指定する`selected`を流し込めるようにします。コンポーネント名は`TheTabs.vue`とでもしておきましょう

```vue
<script setup lang="ts" generic="T extends string">
const props = defineProps<{
  items: { label: T; content: string }[];
  selected: T;
}>();

const selected = computed(() =>
  props.items.findIndex((item) => item.label === props.selected)
);
</script>

<template>
  <UTabs :items="items" :default-index="selected" />
</template>
```

できました。親コンポーネントから実際にコンポーネントを利用してみましょう。今回はテスト用です。なので

- ラベルの型に含まれる要素(Tabs2)をデフォルト値に指定した TheTabs
- ラベルの型には含まれない要素(Tabs4)をデフォルト値に指定した TheTabs

の 2 つを用意することにしました

https://github.com/mikinovation/sandbox/blob/main/vue/ts-no-infer/src/app.vue#L1-L38

それでは typecheck を走らせてみましょう

```bash
npx nuxi typecheck
```

現時点では typecheck が問題なく通りました。なぜなら 2 つ目の TheTabs で Tab4 を指定することで Typescript は「T が Tab4 になる可能性もあるのだな」と推論してくれるからです。ただし今回のコンポーネントでその推論というのはありがた迷惑になってしまいます。先程と同じように NoInfer で推論を防いでみましょう。最終的な完成形が以下になります

https://github.com/mikinovation/sandbox/blob/main/vue/ts-no-infer/src/components/TheTabs.vue#L1-L14

それではもう一度 typecheck を走らせてみましょう

```
src/app.vue:26:29 - error TS2322: Type '"Tab4"' is not assignable to type '"Tab1" | "Tab2" | "Tab3"'.
```

エラーが発生するようになりました。これでうっかり label の型に指定されていない値を指定しても CI 上で必ずミスに気づけるようになりました。これは型は仕様であり、テストであるということを体現した素晴らしいコードだと思います。ハッピーですね！

# まとめ

NoInfer で意図的に型推論を防ぐことにより、よりシンプルに型安全なコードを書けるようになりました

とても便利なのでぜひ活用してみてください
