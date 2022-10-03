---
title: "私達がTailwindに何を期待するのか"
emoji: "✍"
type: "tech"
topics: ["tailwind", "frontend"]
published: false
---

# はじめに

私は普段VueでTailwindを業務で導入し、利用しているフロントエンドエンジニアである。最近TwitterにてTailwind CSSの話題をよく目にするようになり、uhyo氏によって以下の記事が投稿された

https://blog.uhy.ooo/entry/2022-10-01/tailwind/

本記事はこのuhyo氏による記事に対する考察記事となっている。今回は私達がTailwindに対して何を期待しており、何を期待すべきないのか自分なりの考えをまとめてみる

# Tailwind CSSのメリット

Tailwind CSSはユーティリティファーストのCSSフレームワークである

https://tailwindcss.com/docs/utility-first

> This approach allows us to implement a completely custom component design without writing a single line of custom CSS

TailwindはシングルラインでCSSを書くことなく、コンポーネントのデザインを実装できるようにしたものだ。ドキュメントによると以下のように記述されている

- You aren’t wasting energy inventing class names(クラス名を考える時間を無駄にしない)
- Your CSS stops growing(CSSを肥大化させない)
- Making changes feels safer(コードの変更がより安全になる)

まず1点目は開発者がクラスの命名を考える必要がないという点である。この点に関してはuhyo氏の補足によるとTailwindだけ提供できる要素ではなく、インライン全般に言える。この点に関しては後述する

2点目としてCSSを肥大化させないという点である。これはMPAで作ったアプリケーションを想像した方が良さそうだ。MPAで作ったアプリケーションではクラス名を機能毎につけることが多い。そうすると必ず`display: flex`等の同じCSSプロパティに関する記述を膨大に書くこととなる。このようにクラス名だけでグローバルなCSSを定義していくと必ずCSSは肥大化する。

Tailwindははそもそもクラス名をつける必要がないため、`display: flex`のようなCSSは一回のみ定義されたものとみなされ不要なCSSはビルドされたコードには含まれない。この点はTailwindの優れた性能である

3点目としてコードの変更がより安全になるという点である。tailwindでスタイルを記述した場合、スタイルの責務は記述されたDOM要素内に閉じている。よってスタイルの変更があったとしても影響範囲は必ずそのコンポーネントに限定される。これも1点目と同じくTailwindのみが提供できるメリットでなく、インラインスタイル全般に言えるメリットであると考えられる

Tailwindはこれらのメリットによって開発者の生産性を向上させることができると考えられている

# VueにおけるTailwindの功績

「クラス名を考える時間を無駄にしない」ということであるが、これはuhyo氏にも述べられているとおりTailwindだけのメリットではない

この点に関して私はuhyo氏の記事に対して以下のようにTwitterで反論した

https://twitter.com/bohol_engineer/status/1576171624935526401

> コンポーネント設計の粒度の考え方の違いなのかなと思っています。もしスタイリングに関する責務を全て別の関数に分けることを前提とするのであればこれは成り立つと思っているのですが、もしかしたら自分のコンポーネント設計がちょっと粗いからかもしれないですね
> 
> Reactを前提として書かれているかと思うのですが、私は特にVueを主戦場にしてます。なのでSFCで作る関係上スタイルだけの責務を持ったコンポーネントを作るとなるとファイルが膨大な数になってしまうというのがあるのですよね。
> 
> ある程度粗いままのコンポーネントを命名によるクラスで管理しようとすると、やっぱり命名が辛くなってきてしまいます。そこでtailwindのメリットを感じています

実際に同じコンポーネントを書いてみよう

```vue:UserProfile.vue
<template>
  <div class="rounded-2xl p-8 bg-gray-50">
    <slot />
  </div>
</template>
```

```vue:UserIcon.vue
<template>
  <img className="w-24 h-24" :src="props.src" />
</template>

<script setup lang="ts">
import { defineProps } from 'vue'

interface Props {
  src: string
}

const props = defineProps<Props>()
</script>
```

```vue:Profile.vue
<template>
  <UserProfile>
    <UserIcon src="cat.jpeg" />
  </UserProfile>
</template>

<script setup lang="ts">
import UserProfile from './UserProfile.vue'
import UserIcon from './UserIcon.vue'
</script>
```

そしてReactの元記事を少し改修したが、このように書かれている

```tsx:UserProfile.tsx
const UserProfile = ({ children }) => {
  return (
    <div className="rounded-2xl p-8 bg-gray-50">
      {children}
    </div>
  );
};

const UserIcon = ({ src }) => {
  return (
    <img className="w-24 h-24" src={src} />
  );
};

export const Profile = () => {
  return (
    <UserProfile>
      <UserIcon src="cat.jpeg" />
    </UserProfile>
  );
};
```

もし気軽にスタイルの責務を別コンポーネントに分けたい場合は圧倒的にReactの方に優位性がある。これはまさしくJSXの強みだ

Vueは原則SFCで書くことが推奨されている(defineComponentを使ってやろうと思えばできるが...)。気軽にスタイルの責務だけに分離するようなことはできない

当然ながらVueにはstyled-componentやemotionのようなCSS in JSのような選択肢はない。そしてCSSはstyleタグ内にscopedを指定して書く以外の選択肢を選べない

しかしtailwindで流れは変わったと思っている。Vueでもtailwindを利用してDOMに強く結びついたスタイルが書けるようになった。これはTailwindの大きな功績ではなかろうか

## 本当にTailwindは命名の苦しみからあなたを解放しないのか

「Tailwindは命名の苦しみからあなたを解放しない」という点に関してはuhyo氏によって記事の補足が入った

> クラス名はコンポーネントの名前空間に閉じるので、そのコンポーネント内で意味が通じればよい

> 実際に記事の反応を見てみると、wrapperというクラス名ひとつすら付けたくないという意見が結構見かけられました。そのような方にとっては、確かにクラス名がないことは利点になるかなと思います。そのような方にとっては「命名の苦しみからあなたを解放しない」は言いすぎで、ゼロにはなりませんが苦しみを削減できるという利点があるでしょう。

また命名の苦しみはインラインスタイル全般による利点でありTailwindのみが提供できる利点ではないということに関しては同意であるが、少し補足したいところがある

https://blog.uhy.ooo/entry/2022-10-01/tailwind/#%E5%91%BD%E5%90%8D%E3%81%AE%E8%8B%A6%E3%81%97%E3%81%BF%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6

恐らく命名の苦しみから開放するかどうかは、「何のCSSフレームワークと比べてなのか」というコンテキストを合わせなければならない

例えばBootstrapやMaterial DesignといったCSSフレームワークがある。これらはCSSに対してセマンティックなクラス名で管理するフレームワークである。例えば`btn`や`card`のようにDOMに対してクラス名で意味付けをすることができる。またこれらCSSフレームワークはデザインシステムも提供する

これらCSSフレームワークに追加してBootstrap ReactやVuetify(VueのMaterial Design)といったReactやVueのコンポーネントライブラリもある。これらのUIライブラリはスタイルの責務を分離してコンポーネントとして提供できるようにしたライブラリであるが、元のCSSフレームワークと同様に命名をしなければならないことに変わりはない

そしてBootstrapであればTwitterのようなデザイン、Material DesignであればGoogleのようなデザインといったデザインシステムが整えられている

ではこれらのCSSフレームワークはTailwindの代替になるかというとそうはならない。Tailwindはデザインシステムを提供するものではなく、tailwind.config.jsで利用者によってデザインシステムの管理がされなければならない

もしTailwindがBootstrapやMaterial Designの代替となり比較対象となるのであれば、Tailwindは命名の苦しみを解放してくれることになるだろう

それでは別のパターンとしてstyled-componentsやemotionはどうだろうか。これらはCSS in JSによるスタイリングを提供するためのライブラリである。

tailwindはクラスでスタイルを管理するのに対し、styled-componentsやemotionは要素に対してJSでCSSを書けるようにしたものである。なので要素一つに対してスタイルに関する責務を持ったコンポーネントを作り、それらを組み合わせてUIを描画することができる。またstyled-componentではCSSへ直接propsを流し込むようなイメージで条件分岐を追記することができるといった開発体験もいい

```tsx:Button.tsx
const Button = styled.button`
  background: transparent;
  border-radius: 3px;
  border: 2px solid palevioletred;
  color: palevioletred;
  margin: 0 1em;
  padding: 0.25em 1em;

  ${props =>
    props.primary &&
    css`
      background: palevioletred;
      color: white;
    `};
`
```

それではstyled-componentsやemotionと比較して名前の苦しみから開放されるかというと確かにそうなるだろう。なぜならstyled-componentsやemotionは要素に対してスタイルに関する責務を持ったコンポーネントに名前をつけることが多い

`Wrapper`や`OuterWrapper`のようなコンポーネントを生み出す可能性は十分にある

では「Tailwindは命名の苦しみから開放されない」という主張はどこから来ているのか。これは**CSS module**や**CSS in JS**との比較によるものである。先程のCSSフレームワークとの比較からは少し抽象度が上がっている

この観点からであれば「命名の苦しみから開放されない」(Tailwindに特有の問題ではない)という主張には納得できる

それではなぜ初めからインラインスタイルでなくTailwindなのか。この点に関しても公式ドキュメントに回答が載っている

- Designing with constraints(デザインに制約を課すこと)
- Responsive design(レスポンシブデザイン)
- Hover, focus, and other states(ホバーやフォーカス、その他の状態)

最も大事なのがこの1点目だ。Tailwindによってデザインに制約を課すことでデザインシステムに則ったUIを生産的に生み出すことにある。

そして2点目はインラインスタイルにも関わらずレスポンシブデザインに対応できる点である。通常のインラインスタイルはメディアクエリを利用することはできない。だがTailwindにはレスポンシブに対応したユーティリティが備わっている

最後にホバーやフォーカス、その他の状態に対応する点である。これもレスポンシブデザインと同様`hover:`や`focus:`といったprefixをつけることで、インラインスタイルでもスタイリングが実現できるようになっている

# Tailwindを導入する条件

uhyo氏によるとTailwindの採用を検討してもいいケース、しない方がいいケースとしては以下のようなものが挙げられている

- デザインにこだわりがなく、最低限整っていればいい場合は導入を検討してもよい
- 逆に、デザインシステムをしっかりと整備しそれからの逸脱を防ぎたい場合は導入を検討してもよい
- 「デザインはあるけどデザインシステムはない」というパターンにおいてはTailwindを導入する利点がない

先程も述べた通りTailwind自体はデザインシステムを提供せず、利用者がtailwind.config.jsを活用することでデザインシステムが管理されなければならないという点を挙げた

それに対しBootstrapやMaterial Design系は既にデザインシステムが整えられており、開発者側で設定する項目は少なくなるはずである。よってデザインにこだわりがないのであればBootstrapやMaterial Design系を採用する方がTailwindよりも導入コストが低いはずである

これは特に業務システムに言えることである。業務システムはtoC向けのサービスと違い、ある目的を達成できるシステムであればデザインシステムを構築する必要がないケースが多い

それに対しtoC向けのサービスであれば企業や事業のブランドイメージを確立できるようにスタイルガイドが既に整えられていたりするケースがある。その場合はprimary colorやフォントサイズ等をtailwind.config.jsに設定することでデザインシステムとしての制約を与えることができる

よって私は**デザインにこだわりがなく、最低限整っていればいい場合**と**デザインはあるけどデザインシステムはない場合**に関してはTailwindを導入する利点がないと考えている

# Tailwindの弱点

## クラス名が大量になる

この点は公式も認めている

> Now I know what you’re thinking, “this is an atrocity, what a horrible mess!” and you’re right, it’s kind of ugly. In fact it’s just about impossible to think this is a good idea the first time you see it — you have to actually try it

「なんて残虐でひどく汚いコードなんだ！醜くないか」それは正しい。最初はこれが良いアイデアなんで全く思わないだろう。だがとりえあず試してみろ

ここで述べられているとおりクラス名が大量に並んでおり汚く見えるのは正しいが、Tailwindはそれを上回るメリットを享受できると主張している

私自身クラス名が大量になること自体は問題ない。だがクラス名が長くなることである副次的な問題が発生する

それはuhyo氏の記事にも書かれていた

> より即物的な問題として、マークアップは普通のプログラムに比べてインデントが変動しやすい傾向にあり、スタイルが変更される際には大胆に修正されやすいため、破滅的なコンフリクトを生み出しやすいです

これは今私が携わっているプロジェクトでも発生している問題でもあるのだが、デザイナーと共同作業を行う際にDOMとスタイルが分離されていないことによってコンフリクトが発生しやすくなる

このコンフリクトは通常よりもやっかいで大量にあるクラスの差分を目視で判定しなければならない点にある。実際開発中にスタイルのデグレが多発したため、この点に関しては必ずmainブランチを優先的に取り込むという運用でカバーすることになった

## 複雑なUIを構築するのが難しい

典型的な例としては疑似要素やアニメーションを入れたいときである

まず疑似要素に関してはbeforeやafterがtailwindでサポートされている。しかしそれぞれスタイルの定義を細かく定義するとやたらと長いClassになることがある。またアニメーションも業務システムを作るとなるとせいぜいローディングくらいかもしれない

その場合は筆者は次のようにしている。それはtailwindを使わないという方法である。tailwindは決してクラスでのスタイリングを強制するためのものではないと考えている

そこはデザイナーさんと協業しつつ場合によってはコンポーネントを切ってもらったり、あるいはCSSファイルに分離させることで対応している。もちろんなぜそうしなければならないかというコメントもセットで残しておく必要はある

Tailwindにこだわりすぎず、複雑なUIのため逃げの一手は残しておきたいと思っている

## arbitrary values(任意値)が利用できる

arbitrary valuesとはtailwindに新しく追加された機能である。例えば以下のような書き方ができる

```vue:app.vue
<tempate>
  <div class="bg-[#bada55]">
    <p class="text-[22px]">Hello World</p>
  </div>
</tempate>
```

arbitrary valuesを弱点に入れるかどうか迷ったが強みととることもできる。デザインの中でもtailwindの制約を外れて微調整したい箇所が出てくるのはよくある話だ。

この例だとdivの背景色やテキストのフォントサイズを任意の設定値で指定することができる。ただこの機能はあくまでも逃げの一手にとどめておき乱用すべきではない

どの様な優れた技術を使おうとも、開発者が正しく使いこなせなければ宝の持ち腐れになることには変わりない。ただデメリットとして開発者によって容易にデザインシステムを逸脱できるという点は挙げておきたい

# まとめ

今回はuhyo氏の「Tailwind考」に対する私なりの意見をまとめました。

Tailwindに対しては様々な意見が存在し、プロジェクトに採用するかどうかの検討は非常に難しいものだと感じています。この記事を読んでいただいて少しでもTailwindを導入するにあたっての判断材料となるのであれば幸いです

また記事に対する意見や反論はコメントでもTwitterでもいつでもお待ちしております！

