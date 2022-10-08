---
title: "Vueのtemplate、上から書くか?下から書くか?"
emoji: "💡"
type: "idea"
topics: ["javascript", "vue", "frontend"]
published: true
---

# はじめに

VueはコンポーネントをSFC(シングルファイルコンポーネント)で書くことが推奨されているUIフレームワークである。VueにおけるSFCではHTML、CSS、JSを一つのファイルにまとめることで一つのコンポーネントを作ることができる

また最近ではVue3が盛り上がってきている。2020年10月8日現在、Vue2のサポートは来年2023年で終了するという発表もあり長期的にメンテが必要なプロジェクトに関してはVue3への移行が慌ただしく行われていることだろう

そんな中ある一つの疑問が浮かんだ。「人やプロジェクトによってSFCの書き方が異なる。templateタグ、scriptタグを書く順番が違う」と。

どういうことか具体的に見てみよう。以下はVue3公式から引用したコードだ

https://vuejs.org/guide/introduction.html#api-styles

```vue
<script setup>
import { ref, onMounted } from 'vue'

// reactive state
const count = ref(0)

// functions that mutate state and trigger updates
function increment() {
  count.value++
}

// lifecycle hooks
onMounted(() => {
  console.log(`The initial count is ${count.value}.`)
})
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>

<style scoped>
button {
  font-weight: bold;
}
</style>
```

しかし私はVue2に慣れ親しんでいた、なので以下のような方法でVue3も書いていた

```vue
<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>

<script setup>
import { ref, onMounted } from 'vue'

// reactive state
const count = ref(0)

// functions that mutate state and trigger updates
function increment() {
  count.value++
}

// lifecycle hooks
onMounted(() => {
  console.log(`The initial count is ${count.value}.`)
})
</script>

<style scoped>
button {
  font-weight: bold;
}
</style>
```

templateとscriptタグの順番が逆になっている

そこで今回のタイトルだ。**Vueのtemplate、上から書くか?下から書くか?**(「打ち上げ花火、下から見るか? 横から見るか?」風)

## 結論

最初に結論を述べると**SFCにおいてはscriptタグ→templateタグ→styleタグの順番で書く**ことを推奨したい

## scriptタグを先に書くメリット

「scriptタグ→templateタグ→styleタグ」の順番で書く場合には次のようなメリットがある

### コンポーネントロジックに対する関心とデザインに対する関心の分離

プロジェクトにおいてデザイナーとの協業が必要なことがある。もしくは一人がフロントのロジックとデザイン両方を担当していたとしてもロジックに対する関心とデザインに対する関心でタスクを分離するはずだ

ロジックに対する関心が高い場合はscriptタグとtemplateタグに意識が集中する。デザインに対する関心が高い場合にはtemplateタグとstyleタグに意識が集中するはずである

もし「templateタグ→scriptタグ→styleタグ」の順番で書いた場合はどうなるだろうか。デザインに対する関心が高い場合に困ってしまう。

先程の例を見てみよう

```vue
<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>

<script setup>
import { ref, onMounted } from 'vue'

// reactive state
const count = ref(0)

// functions that mutate state and trigger updates
function increment() {
  count.value++
}

// lifecycle hooks
onMounted(() => {
  console.log(`The initial count is ${count.value}.`)
})
</script>

<style scoped>
button {
  font-weight: bold;
}
</style>
```

デザインに対する関心はtemplateタグとstyleタグに集中する。しかしその間にscriptタグが挟まれることによって無関心な領域を挟まなければならない。これは僅かかもしれないがデメリットではある

もちろんComposition APIを使って、コンポーネントからロジックを切り離すことはできるが特にライフサイクルフックやprops、emit等定義する以上は無関心の領域を間に挟みたくはない

例外としてTailwind CSSのようなインラインスタイルの定義をする場合がある。この場合デザインはtemplateのみに関心を持つためscriptタグとの順番を問う必要はない

少なくともstyleタグにCSSを定義するのであれば、templateタグはscriptタグよりも下に書いたほうが生産性が上がるであろう

## 他フレームワークとの親和性

他のフレームワークとの親和性を考えるとやはりscriptタグ→templateタグ→styleタグの順番で書くことを推奨したい

例えばReactの場合は以下のように書くことができる(スタイリングにはCSS Modulesを使っている)

```jsx
import React from 'react'
import styles from './styles.module.css'

export const Button = () => {
  const [count, setCount] = useState(0)
    
  const increment = () => {
    setCount(count + 1)
  }
    
  return (
    <button className={styles.btn} onClick={increment}>
      Count is: {count}
    </button>
  )
}
```

JSXの場合にはtemplateタグがなく、関数の返り値としてViewが返される。この場合自然とロジック→Viewの順番で定義せざるを得ない。またこれはJSXを利用したライブラリやフレームワーク全般に言えることでSolid.jsも同様である

ではSvelteはどうであろう。Svelteは記法がVueよりシンプルではあるが、書き方は比較的近い。SFCでの設計方針は変わらない

```svelte
<script>
  let count = 0;
	
  const increment = () => {
    count++
  }
</script>

<button on:click={increment} >
  Count is: {count}
</button>

<style>
  button {
    font-weight: bold;
  }
</style>
```

Svelteの場合はscriptタグとstyleタグ以外は全てtemplate扱いになる。サンプルコードを見る限りは上記のように書くことが多いようだ。

Angularに関してはHTML、CSS、Scriptが全て別ファイルに分かれてしまっているため参考にはできそうにない

現在人気のあるフレームワークの多くはロジック→Viewの順番で書くことが多いため、Vueの場合にもscriptタグ→templateタグ→styleタグの順番で書くというのは立派な理由になり得るだろう

# まとめ

今回はVueのSFCにおけるタグの順番に関して考察してみた。

最後にもう一度結論を述べると、Vueのタグは**scriptタグ→templateタグ→styleタグ**の順で書くことを推奨する

もちろんこれは私自身の意見なのでどの順番で書くかは各プロジェクトのチームメンバー同士で決めて頂きたい。

もしこれ以外にtemplateタグを先に書いたほうが良いという意見もあればZennのコメントやTwitterでも歓迎である

