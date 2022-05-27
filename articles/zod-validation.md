---
title: "【Blitz.js】Reactとzodから理解を深める関数型プログラミング"
emoji: "😜"
type: "tech"
topics: ["React", "Blitz.js", "zod"]
published: false
---

:::message
当記事は関数プログラミングとはこういうものであるという定義をするための記事ではありません
また圏論やモナド等やたらめったらに難しい用語を羅列されて全く意味が分からないという苦痛を私自身経験しているので、少しでもその感覚が和らいでほしいと思っております
今回は最近話題のBlitz.jsで使用されているzodというバリデーションライブラリを題材とし、関数型プログラミングでよく使われる技法となぜそうするのかを理解することで関数型プログラミングに触れようという内容です
本格的に関数型プログラミングを学びたいという方は、このような記事でなくHaskell等の純粋関数型言語に触れてみてください
:::

# Blitz.jsとは

Blitz.jsとは最近話題となっているReact製のフルスタックフレームワークとなっております

今までフルスタックフレームワークといえばNext.jsが代表的なものとして存在しましたが、サーバーサイドに関してはAPIルートのみという非常に簡素な作りでした

そこでBlitz.jsではSSRのコンポーネント作成からAPIルート以上にデフォルトの設計が充実しており

Blitz.js wayに乗っかることでアプリケーションロジックの作成に集中できる作りとなっております

またBlitz.jsにはORMとしてPrismaが採用されています

Prismaがやばいという件に関しては以前こちらで記事を書きました。もし時間があれば読んでみてください

https://zenn.dev/mikinovation/articles/nextjs-prisma

最初のCLIによるボイラープレート作成段階でデータベースへの接続やORMによるマイグレーション、認証周りのサインアップやサインイン、Typescript対応やLint、ユニットテスト

そういった一般的なWebアプリケーションの開発に必要な設定がされた状態でプロジェクトを開始することができます

（私はまさにこういうのを求めていましたが皆さんもそうですよね！？）

どうやらRails版Next.jsとも呼ばれているそうです

とりあえず「Blitz.jsすげーよ」とだけ言っておきます

# zodとは

さて今回のトピックにしたのが「zod」です

このzodとはBlitz.jsで採用されているバリデーションライブラリです。Reactに限らずVueやサーバーサイドのNodeでも問題なく使えます

https://github.com/colinhacks/zod

「TypeScript-first schema validation with static type inference」

Typescriptファーストで静的型推論によるスキーマのバリデーションを提供しますよとのことです

私の日本語もよくわからないのでzodの特徴をあげてみます

## 型推論

:::message
zodは以下のようにライブラリをimportできますがこれ以降は「z」として記述を省略します

ご了承ください

```ts
import { z } from "zod";
```
:::

まずは基本的なところです

zodは型推論をすることができます

```ts
const A = z.string();
type A = z.infer<typeof A>; // string

const u: A = 12; // TypeError
const u: A = "asdf"; // compiles
```

## 関数型プログラミングでバリデーションをしてみる

今回は実際にBlits.jsでデフォルトで生成されるLogin機能のバリデーションを見てみましょう

以下app/auth/validations.tsの内容です

このファイルにはLoginというスキーマが定義されています

```ts
export const email = z
    .string()
    .email()
    .transform((str) => str.toLowerCase().trim())

export const password = z
    .string()
    .min(10)
    .max(100)
    .transform((str) => str.trim())

export const Login = z.object({
    email,
    password: z.string(),
})
```

ここで疑問に思う方もいるかもしれません。

「Loginにz.objectという関数の返り値を代入する時点でemailやpasswordのバリデーションが実行されてしまうのではないか」と。

ここで関数型の大きな特徴である「遅延評価」というキーワードが出てきます

遅延評価とは「値が必要になるまで、値の評価を後回しにする」ということです

上記のスキーマを定義している段階では関数を合成しているにすぎず値が評価されることはありません

ここでログインボタンをクリックイベントで発火するmutationの中身を見ていきます

そもそもなぜ私がzodに興味を持ったのかという話ですが、以前このようなツイートをしました

https://twitter.com/bohol_engineer/status/1526094226949169157

ElmとはJavascriptにコンパイルできる関数型のプログラミング言語です

今回Elmに関して深く触れませんが、上記のツイートの記事では「フォームデコーディング」に関して触れています

通常Reactでフォームバリデーションをする際には常に文字列で値を取得するため、バリデーション時に型を変換する必要があります

それでは具体的な例を見てみましょう

以下のように実装するとconsoleには、何も入力せずにバリデーションボタンを押したとき以外trueが出力されます

eventから取得したinputに入力された値は常に文字列で取得されます

```tsx
const App = () => {
  const [number, changeNumber] = useState<string>('')
  const handleClick = () => {
    console.log(typeof count === 'string')
  }
  return (
    <>
      <label>数値を入力してください</label>
      <input
        type="text"
        value={number}
        onChange={e => changeNumber(e.target.value)}
      />
      <button type="button" onClick={handleClick}>バリデーション</button>
    </>
  )
}
```

それではこれにバリデーション処理を追加してみましょう。今回は入力された値が10以上であればエラーとします

```ts
const handleClick = () => {
  if(count >= 10){
      alert('入力された値は10以上です')
  }
}
```

typescriptだとここで問題に直面します。inputのchangeイベントで返ってくるeventオブジェクトに含まれるvalueの値は常に文字列です

なので「count >= 10」はコンパイルエラーとなります

そこでcountが数値であることを保証するため追加の処理を書かなければなりません

```ts
const handleClick = () => {
    
  const intCount = Number(count)
    
    if(typeof intCount !== 'number') {
        alert('入力された値は数値ではありません')  
    }
    
    if(count >= 10){
      alert('入力された値は10以上です')
    }
}
```

文字列を数値に変換してその型が数値であるか検証するという過程はコードをより複雑化させます

ここでzodに戻ってみましょう。zodでは型を検証するのではなくparseします

先程の処理をzodでバリデーションするとなると以下のようになります

```ts
const num = z
    .number({
      required_error: "数値は必須です",
      invalid_type_error: "入力された値は数値ではありません",
    })
    .lt(10, {message: '入力された値は10以上です'}) // ltは「未満」, lteは「以下」

const handleClick = () => {
    const data = num.safeParse(count);
    if (!data.success) return
    alert(data.error.issues.message);
}
```

numというスキーマにはどういうデータの形式かではなく、どういう形式であるべきかを書きます

非常にシンプルな形になりましたね。これで型の恩恵を受けたまま正しくparseされたかどうかを評価してくれます

ここで一つ関数型プログラミングに利用される「遅延評価」について説明します

上記のコードに関して疑問を持たれた方もいるかもしれません。「numを定義した時点でnumberやltという関数が実行されてしまうのではないか」と。

しかし実際はsafeParseという関数が実行された時点で式が評価されます。これが「遅延評価」です

遅延評価ができることによって何が嬉しいのか考えてみましょう

遅延評価においては計算が必要になった時点で評価されます。つまり計算が必要になるまで式の評価はされません

これが大きなパフォーマンスの改善に繋がり、かつ関数型プログラミングの式を短くシンプルに保てる秘訣です

先程の例をもう一度見てみましょう

```ts
const num = z
    .number({
      required_error: "数値は必須です",
      invalid_type_error: "入力された値は数値ではありません",
    })
    .lt(10, {message: '10未満の数値で入力してください'}) // ltは「未満」, lteは「以下」
```

safeParseを実行した時点で最初にnumberという式が評価されます

もし仮に「test」という文字列を入力していたとするとnumberで評価は止まり、以降のltという式の評価はされません

同じ処理を手続き型で書いてみたいと思います

```ts
const validate = () => {
    let hasError: boolean = false
    
    if(!hasError && exists(count)) { // 0を含めたい
        hasError = true
        alert('入力は必須です')
    }
    
    if(!hasError && isNumber(count)) {
        hasError = true
        alert('入力された値は数値ではありません')
    }

    if(!hasError && count >= 10) {
        hasError = true
        alert('10未満の数値で入力してください')
    }
}
```

ここで目につくのがif文による処理とhasErrorという変数の管理による煩雑さです

あるいはこう書く方もいるかもしれません。これもかなりひどいコードですね

```ts
const exists = (): boolean => {/*...*/}

const isNumber = (): boolean => {/*...*/}

const isBiggerThan = (num: number): boolean => {/*...*/}

const validate = (): boolean => {
    if(!exists()) {
        if(!isNumber()) {
          if(!isBiggerThan(10)) {
              return true         
          } else {
              alert('10未満の数値で入力してください')      
          }
        } else {
            alert('入力された値は数値ではありません')    
        }
    } else {
        alert('入力は必須です')
    }
    return false
}
```

お分かりの通り非常に煩雑です。この様なコードは管理コストが高くバグも起きやすくなります

しかし関数型であれば遅延評価によって、必要になってから値を評価できます。if文による煩雑さに悩まされることもありません。今回の例だと保守性や拡張性、可読性に優れていることが分かります

:::message
誤解を招かないように。if文を無くそうとするのが関数型ではありません。
ただ関数型の技法を採用すると自然とif文の数は減ります
:::

## モナド

さてここでもう一つ「モナド」という抽象的で非常に難しい概念に触れましょう

まずモナドを理解する前にJavascriptにおけるtry-catch文の問題点をあげます

try-catchはある動作の予測不能な処理のあるコードを囲ってしまうことで、その処理中に起きた事象をキャッチできるような仕組みです

このtry-catchは非常に強力で便利な一方、関数のように合成やチェーン化することができません

