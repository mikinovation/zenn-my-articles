---
title: "VueにおけるCSS Modulesの考察"
emoji: "🖼"
type: "tech"
topics: ["vue", "javascript", "frontend"]
published: false
---

# はじめに

Vueではスタイルを適用する際にいくつかの手法がある。特にその中でも人気のある手法がstyleタグをscopedにしてコンポーネントに閉じたスタイルを書く方法だ

その中でも意外と知られていないのがCSS Modulesでスタイルを当てる方法である。CSS ModulesといえばReactでよく使われているスタイリングの手法だと思われている方も多いだろう。実はVueでもできるのだ

今回はこのCSS Modulesについて考察していく

# VueにおけるCSS Modules

## 基本的な使い方

まずはよく使われるScoped CSSの書き方から見ていこう

コードはVueの公式ドキュメントから参照させてもらった

https://vuejs.org/api/sfc-css-features.html

```vue
<template>
  <div class="example">hi</div>
</template>

<style scoped>
.example {
  color: red;
}
</style>
```

これに対して本題のCSS Modulesでは以下のように書ける

```vue
<template>
  <div :class="$style.example">hi</div>
</template>

<style module>
.example {
  color: red;
}
</style>
```

styleタグに`module`をつけることで`$style`というクラス名のユニークなキーが格納されたオブジェクトを利用できるようになった

## 複数モジュールの利用

複数のmoduleを作ることもできるようだ。しかしSFCにおいて常に1ファイル1コンポーネントが原則ではある。使用頻度は少なそうだ

```vue
<template>
  <div>
    <p :class="classes1.red">red</p>
    <p :class="classes2.green">green</p>
  </div>
</template>

<style module="classes1">
.red {
  color: red;
}
</style>

<style module="classes2">
.green {
  color: green;
}
</style>
```

## Composition APIでの利用

Composition APIを使ってJSからCSS Modulesを扱うこともできるらしい。試しに使ってみる

```vue
<script lang="ts" setup>
const cssModule = useCssModule()
console.log('css module', cssModule) // css module {red: '_red_1xn05_3'}
</script>

<template>
  <p :class="$style.red">red</p>
</template>

<style module>
.red {
  color: red;
}
</style>
```

上記のように出力された。オブジェクトのキーをベースとした、プログラマティックに自由に組み合わせたクラスの設計ができそうだ

# scoped styleとCSS Modulesどちらを採用するか

## scoped styleにおけるdeepセレクタとの比較

scoped styleではdeepセレクタを使うことで、子コンポーネントのスタイルを上書きすることができる

```vue:ChildComponent.vue
<template>
  <div>
    <p class="inner">red</p>
  </div>
</template>

<style scoped>
.inner {
  color: red;
}
</style>
```

```vue:ParentComponent.vue
<template>
  <ChildComponent class="parent" />
</template>

<style scoped>
.parent :deep(.inner) {
  color: blue;
}
</style>
```

こうすることで親コンポーネントで書いたスタイルを子コンポーネントで上書きできるようになる

それに対しCSS Modulesの例を見てみよう

```vue:ChildComponent.vue
<script lang="ts" setup>
interface Props {
  color?: string
}

const props = withDefaults(defineProps<Props>(), {
  color: ''
})
</script>

<template>
  <div>
    <p :class="[$style.inner, props.color]">red</p>
  </div>
</template>

<style module>
.inner {
  color: red;
}
</style>
```

```vue:ParentComponent.vue
<template>
  <ChildComponent :color="$style.blueText" />
</template>

<style module>
.blueText {
  color: blue;
}
</style>
```

moduleをpropsから流し込むことで、子コンポーネントのスタイルを上書きすることができる

と、ここまで親コンポーネントで子コンポーネントを上書きできるという機能を紹介しておきながら言うのも難だが親コンポーネントから子コンポーネントを上書きするのはなるべくなら避けたい

そもそも親コンポーネントから子コンポーネントに対してclassを渡すべきなのかという点に関してはReactで既になされている

特にUhyo氏とTakepepe氏の記事は参考になるので細かく知りたい方はぜひ参照してほしい

私の意見としてはUhyo氏と同じく親コンポーネントからclass名を子コンポーネントに流し込むべきではないと考えている

昨今のデザインシステムを構築するという背景においてもスタイルを決定するのは子コンポーネント側であるべきだ。 それでは親コンポーネントは子コンポーネントのスタイルを決定できないのかというとそうではない

デザインシステムはその企業や事業のブランディングという意味と同時に開発においてはデザインに制約を課すことが目的である。子コンポーネントのスタイルを決定するのは子コンポーネントの責務ではあるが、子コンポーネントは親コンポーネントから流し込むpropsの値を制限することはできる


