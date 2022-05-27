---
title: "JavascriptでTinymceのようなリッチなエディターのセキュリティ対応"
emoji: "😜"
type: "tech"
topics: ["Javascript", "vue", "sanitize-html"]
published: true
---

# Tinymceとは

リッチエディタの一つ

https://www.tiny.cloud/

以下のサンプルのようにHTMLやCSSに詳しくない人でも簡単にスタイルをつけた文章が書けるようになるツール

https://www.tiny.cloud/docs/demo/basic-example/

このTinymceをとあるプロジェクトに組み込んだ 。しかしリッチエディタを導入するにあたって付きまとうのがセキュリティ問題

今回は書き込み時や読み込み時にフロントエンドでどう対応するか記録に残しておく

# READ

まずは比較的シンプルな読み込み時の対応

今回はVueを使用するが、VueにはHTMLの文字列を展開する便利なディレクティブが存在する

```vue
<template>
  <div v-html="state.html"></div>
</template>

<script lang="ts" setup>
import { defineExpose, reactive } from 'vue'

const state = reactive({
  html: '<p>hello</p>'
})

defineExpose({
  state
})
</script>
```

しかし生のHTMLをそのま展開するのは危険

例えば以下の場合、scriptタグが含まれているとみなし実行できてしまう(XSS)

これを防ぐためにsanitize-htmlを使って対応

https://github.com/apostrophecms/sanitize-html

表示時にはプレーンなテキストとして出力されるようになった

```vue
<template>
  <div v-html="sanitizedHtml"></div>
</template>

<script lang="ts" setup>
import { defineExpose, reactive, computed } from 'vue'
import sanitizeHTML from 'sanitize-html'

const state = reactive({
  html: '<p>hello</p><script>alert("Hello")</>script>'
})

const options = {}
// 実際にはプロジェクトでoptionsをカスタマイズしてしますが、今回は省略します
const sanitizedHtml = computed(() => sanitizedHtml(state.html, options))

defineExpose({
  state,
  sanitizedHtml,
})
</script>
```

# CREATE

続いて登録処理。セキュリティ上の関係で今回はバリデーションの一部だけ紹介する

あるcontentという文字列のdomををバリデーションしたい

ルールは以下

「サーバー側に送信するaタグにはhttpやhttpsしか許可しない」

サンプルとして作ったのが以下のような処理

```ts
export const validateHTML = (html: string) => {
  // htmlに含まれているaタグを全て抜き出す
  const aTags = html.match(/<a(?: .+?)?>.*?<\/a>/g) || []
  const validHrefs = aTags
    // aタグからhrefのみ抽出
    .map((aTag: string) => aTag.match(/href\s*=\s*"([^"]*)"/))
      // 配列の2番目の要素が実際に抽出した文字列
      .filter(
      (matchArray) =>
        matchArray &&
        (matchArray[1].startsWith('http') || matchArray[1].startsWith('https'))
    )
  // 全てのタグの数と有効なタグの数が含まれていなかったらtrue
  return aTags.length === validHrefs.length
}
```

念のためテストもつけておく

```js
describe('validateHTML', () => {
    describe('有効なHTML', () => {
      it('pタグ', () => {
        expect(validateHTML('<p>hello</p>')).toBe(true)
      })

      it('http付きaタグ', () => {
        expect(validateHTML('<a href="http://example.com">hello</a>')).toBe(
          true
        )
      })

      it('https付きaタグ', () => {
        expect(validateHTML('<a href="https://example.com">hello</a>')).toBe(
          true
        )
      })

      it('複数マッチaタグ', () => {
        expect(
          validateHTML(
            '<a href="https://example.com">hello</a><a href="http://example.com">hello</a>'
          )
        ).toBe(true)
      })

      it('混合マッチaタグ', () => {
        expect(
          validateHTML('<p>hello</p><a href="https://example.com">hello</a>')
        ).toBe(true)
      })
    })

    describe('無効なHTML', () => {
      it('無効なプロトコル', () => {
        expect(validateHTML('<a href="ws://example.com">hello</a>')).toBe(false)
      })
    })
  })
```
