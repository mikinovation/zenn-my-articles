---
title: "Nuxt3で個人的にuseFetchよりuseAsyncDataを推したい理由"
emoji: "⚔"
type: "tech"
topics: ["frontend", "vue", "Nuxt", "ofetch"]
published: true
---

# はじめに

Nuxt3の安定版がリリースされ、しばらく経った。現在筆者もいくつかのプロジェクトでNuxt3を活用し、本番のサービスにも導入している

Nuxt3は以前の2系にも増して更に強力な機能を公開しており、快適な開発体験を提供してくれる

代表的な機能としては

- Auto-imports
- Data-fetching utilities
- Zero-config TypeScript support
- Universal rendering (Server-side rendering and hydration)

といったものがある。またNitroという新しいサーバーエンジンが導入され、より高いパフォーマンスでのレンダリングが可能になった

そしてNuxt3では2つの新しいAPIであるuseFetchとuseAsyncDataが導入されている(useLazyFetchとuseLazyAsyncDataも含めれば4つ)。useFetchとuseAsyncDataはNuxt3が登場してからしばしば比較され、useFetchを推す人もいれば、useAsyncDataを推す人もいる

今回の記事ではなぜ私がuseAsyncDataを推したいのかを紹介したい

# useFetchとuseAsyncDataとは

まずは簡単にuseFetchとuseAsyncDataとは何か説明する

useFetchとuseAsyncDataとはNuxt3でAPIにリクエストを送り、リスポンスを受け取って返すというAPIだ。

他にもaxiosやReactであればuseSWRなどのライブラリも存在するが、Nuxt3ではuseFetchやuseAsyncDataというAPIが推奨されている

それではuseFetchよりもuseAsyncから説明したほうがわかりやすいかと思うので、まずはuseAsyncDataから説明してみよう

## useAsyncData

https://nuxt.com/docs/api/composables/use-async-data

> Within your pages, components, and plugins you can use useAsyncData to get access to data that resolves asynchronously
>
> "ページやコンポーネント、プラグイン内にて、非同期でデータを取得するために使うことができる。"

非常にシンプルな説明であるが、この点は先程も述べたとおりだ。もう少し深掘りするためにサンプルコードを見てみよう

```ts
const { data, pending, error, refresh } = await useAsyncData(
  'mountains',
  () => $fetch('https://api.nuxtjs.dev/mountains')
)
```

useAsyncDataは2つの引数を取る。

第一引数にはキーを指定している。これは一体何であろうか。useAsyncDataは内部でキャッシュを保持している。このキャッシュを利用することで2回目以降はAPIにリクエストを送らずとも前の値と同じ結果を返してくれる。このキャッシュはNuxt内部で保持するため、キャッシュを区別するための一意なキーが必要だ。このキーが第一引数に指定されている

第二引数には非同期処理を行う関数を指定している。このコールバック内でWeb APIにリクストを送信し、useAsyncDataの結果として値を返すことができる。

第二引数には`$fetch`という関数が登場した。Nuxt3ではHTTP Clientに`ofetch`というライブラリが推奨されている。これはブラウザのFetch APIを拡張したライブラリであり、ブラウザだけでなく、NodeやWeb Worker上でも利用することができる軽量なHTTP Clientだ。

https://github.com/unjs/ofetch

この$fetchを利用することで、非同期処理を扱うuseAsyncData内でWeb APIにリクエストを送信することができる

それではuseAsyncDataの戻り値を見てみよう

useAsyncDataが返す値で使うのは主にこの4つだ

- data
- pending
- error
- refresh

dataは第二引数で指定した非同期な処理から、その結果を取り出して返すリアクティブな値だ。pendingは非同期処理が完了したかどうかを判定する。特にローディング処理には必須の値となる。errorは非同期処理が完了した段階でエラーが発生した場合の結果を返してくれるリアクティブな値である。最後にrefreshであるが、これは第二引数で指定している非同期処理を再実行する際に利用できる関数である

この返り値を見ればわかるが、useAsyncDataとはHTTP Requestを送るためのAPIというよりは、非同期処理の状態を管理するためのAPIということができるだろう

ちょっと話は逸れるが最近Reactに**use**というAPIが話題となった。ReactのuseとuseAsyncDataは一見似ているが役割が違う

useAsyncDataはキャッシュ管理をした上で非同期で取得した値を取り出すが、useは単純に非同期処理から結果を取り出し、非同期コンポーネントとしてSuspenseを利用できるようにするAPIだ。Reactは今までuseQueryやuseSWRのライブラリを使わなければSuspenseを利用して非同期コンポーネントを作ることが難しかった

それに対してVue3ではトップレベルawaitを用いるだけで簡単に非同期コンポーネントを作ることができる。VueはやっぱりEasyだ

```vue:Parent.vue
<template>
  <Suspense>
    <Child />
    <template #fallback>
      <LoadingSpinner />
    </template>
  </Suspense>
</template>
```

```vue:Child.vue
<script lang="ts" setup>
  const { data } = await new Promise((resolve) => {
    setTimeout(() => {
      resolve({ data: 'Hello World' })
    }, 1000)
  })
</script>

<template>
  <h1>
    {{ data }}
  </h1>
</template>
```

## useFetch

useAsyncDataについて理解できたところでuseFetchについて説明する

https://nuxt.com/docs/api/composables/use-fetch

> This composable provides a convenient wrapper around useAsyncData and $fetch.
> It automatically generates a key based on URL and fetch options, provides type hints for request url based on server routes, and infers API response type.
>
> "このcomposableは、useAsyncData と $fetch の便利なラッパーを提供する
>  URLとfetch optionsに基づいてキーを自動的に生成、server routesに基づいてリクエストURLの型ヒントを提供する。
>  また、API Responseを型推論することができる"

これを読むと非常に便利なAPIであることが分かる

先程参照したuseAsyncDataのコードを見てみよう

```ts
const { data, pending, error, refresh } = await useAsyncData(
  'mountains',
  () => $fetch('https://api.nuxtjs.dev/mountains')
)
```

これがuseFetchを使うと以下のように書けるようになる。

```ts
const { data, pending, error, refresh } = await useFetch('https://api.nuxtjs.dev/mountains')
```

コードがすごく短くなった。いちいちキーを書く必要もないし、$fetch書く必要もない。

useFetchは **useAsyncData** と **\$fetch** をラップした非常に便利なAPIだ。キーの生成はフレームワークに任せておけばいいし、$fetchのオプションも渡そうと思えば第二引数から渡すことができる

```ts
const param1 = ref('value1')
const { data, pending, error, refresh } = await useFetch('https://api.nuxtjs.dev/mountains',{
    query: { param1, param2: 'value2' }
})
```

それでは「ワーイめでたし！めでたし！」なのか。

ここで本記事のタイトルを思い出してほしい。**Nuxt3で個人的にuseFetchよりuseAsyncDataを推したい理由**である

# Easy VS Simple

ここで改めて尽きない話題ではあるが、Easy VS Simpleについて考えてみよう

ソフトウェアの設計は常にトレードオフだ。どのような設計をするかは、何を作るかによる

今回の話題はまさにそれである。**「useFetch」はEasyだが、「useAsyncData + $fetch」はシンプル**だ

useFetchは非常にEasyなのだが、たまに困ることがある。例えば一度に複数のAPIを叩く場合にどうするか

まずは以下のように連続してAPIを取得することができるだろう

```ts
// 山を取得するAPI
const { 
    data: mountaionsData, 
    pending: moutainsPending, 
    error: moutainsError, 
    refresh: moutainsRefresh 
} = await useFetch('https://api.nuxtjs.dev/mountains')
// 川を取得するAPI
const {
    data: riversData,
    pending: riversPending,
    error: riversError,
    refresh: riversRefresh
} = await useFetch('https://api.nuxtjs.dev/rivers')
```

2つの処理が軽い場合は問題ないかもしれないが、重い処理の場合はどうだろうか。以下のように並列にした方が効率的かもしれない(Javascriptの並列処理うんぬんに関する説明は割愛)

```ts
const { data, pending, error, refresh } = await useAsyncData(
    'getMountainsAndRivers',
    async () => {
        return await Promise.all([
            $fetch('https://api.nuxtjs.dev/mountains'),
            $fetch('https://api.nuxtjs.dev/rivers')
        ])
    }
)
```

将来的にofetchより魅力的なライブラリが登場したらどうするだろうか。useFetchだとNuxtのフレームワークに依存するため、切り離しが難しく、気軽に他のライブラリを試したりすることができないかもしれない

```ts
const { data, pending, error, refresh } = await useAsyncData(
    'getMountainsAndRivers',
    () => betterFetchAPI('https://api.nuxtjs.dev/mountains')
)
```

ここで話を戻そう。ここまでuseAsyncDataのシンプルな面のメリットばかりを推してしまった。何度もいうがソフトウェア設計は常にトレードオフである

例えば使い捨てのツールを作りたいときや短期で使うWebサイトにはわざわざuseAsyncDataを使う必要はないだろう。useFetchを使えばいい

もしくはserverディレクトリにWeb APIの処理を書く場合はどうだろうか。この場合もuseFetchが活用できる。pathが型サポート受けられるというメリットがあるからだ。pathの型サポートに魅力を感じられるかどうかはプロジェクトによるかもしれない

私自身の前提が入ってしまって恐縮ではあるが、**中長期的にメンテされることが見越されていたり、サービスをスケールさせていこうとしている場合にuseAsyncDataを推したい**というのが私の主張である

# ofetchを活用する

おまけでofetchの使い方に関して軽く紹介したいと思う

## ofetchの特徴

- Works with Node.js
- Parsing Response
- JSON Body
- Handling Errors
- Auto Retry
- Type Friendly
- Interceptors

等、基本的なHTTP Clientとして使えそうな機能は揃っている。中にはAuto Retryといったあまり見たことない機能もあるので紹介したいと思う

## サンプルコード
ここに私の使っているサンプルコードを一部紹介する。APIのfetcherを以下のようにそれぞれ定義しておく

```ts:shared/api/fetcher.ts
import { FetchContext, FetchOptions } from 'ofetch'
import { config } from "@/shared/config"

const onRequest = ({ request, options }: FetchContext) => {
  // Log request
  console.info('[fetch request]', request, options)
}

const onRequestError = ({ request, error }: FetchContext) => {
  // Log error
  console.error('[fetch request error]', request, error)
  // Parse request body with zod...
}

const onResponse = async ({ response }: FetchContext) => {
  // Log response
  console.info('[fetch response]', response)
}

const onResponseError = ({ request, response }: FetchContext) => {
  // Log error
  console.error(
    '[fetch response error]',
    request,
    response?.status,
    response?.body
  )
  // Handle response error...
}

const defaultOptions: FetchOptions = {
  baseURL: config.BASE_URL,
  parseResponse: JSON.parse,
  retry: 3,
  onRequest,
  onRequestError,
  onResponse,
  onResponseError,
}

export const fetcher = async <T>(
  path: string,
  fetchOptions: FetchOptions = defaultOptions
): Promise<T> => {
  return await $fetch<T>(path, {
    ...defaultOptions,
    ...fetchOptions,
  })
}
```

### fetcherの定義

```ts
export const fetcher = async <T>(
  path: string,
  fetchOptions: FetchOptions = defaultOptions
): Promise<T> => {
  return await $fetch<T>(path, {
    ...defaultOptions,
    ...fetchOptions,
  })
}
```

基本的には **\$fetch** 関数を実行しているだけだ。エラーハンドリングとfetchOptionsのラップをした関数を定義している。また$fetchはgenericsを指定することで型安全になる

### fetchOptionsの定義

```ts
const defaultOptions: FetchOptions = {
  baseURL: config.BASE_URL,
  parseResponse: JSON.parse,
  retry: 3,
  onRequest,
  onRequestError,
  onResponse,
  onResponseError,
}
```

今回はdefaultOptionsを用意した。parseResponseを使うことでresponseで受け取ったJSONをオブジェクトにparseすることができる。

また先程も紹介したがAuto Retryも使用することができる。例えばnetwork errorが発生した場合には最大3回までリクエストをリトライできるようになっている

ただしPOSTリクエストのようなバックエンドのDBに大きな影響を及ぼすようなリクエストに対しては適用されないので安心してほしい

### 便利なInterceptors

今回は以下4つ全てに対して、仮のログ出力のみ処理を入れているが、これらのhookは非常に便利だ

- onRequest
- onRequestError
- onResponse
- onResponseError

# まとめ

今回は「Nuxt3で個人的にuseFetchよりuseAsyncDataを推したい理由」を紹介した

少しだけfetchというインフラ層の部分をNuxtの外に追い出したいという気持ちが強くなってしまった気もするが、そこはプロジェクトに合わせて柔軟に使用するAPIを選定して頂きたい

改めて選定基準としては以下である

- useFetchを使う場合
    - 短期間での開発や使い捨てツールのようなものを作る場合
    - Nuxtに依存、追従して問題ない場合
    - serverディレクトリにバックエンドのAPIを定義し、pathの型ヒントを得たい場合
- useAsyncData + $fetchを使う場合
    - 中長期での開発やサービスのスケールが見込まれる場合
    - Nuxtに依存せず、インフラ層をNuxtから切り離したい場合