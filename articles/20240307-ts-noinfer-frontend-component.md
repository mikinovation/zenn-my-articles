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

```
createStreetLight(["red", "yellow", "green"], "blue");
```

配列の要素に含まれていない`blue`という要素を第二引数の`defaultColor`に使用しました。実はこれで typecheck を走らせても問題なく通ってしまいます。これは困りました。配列に存在しない値はあらかじめ第二引数に指定できないようにした方がより安全がバグの少ない、より型安全なコードを書けるはずです。

それでは今まではどうしていたか。以下のように書くことで配列の要素に含まれている型のみを第二引数に指定できるようにしていました。

```ts
function createStreetLight<C extends string, D extends C>(colors: C[], defaultColor?: D) {
  ...
}
```

少し複雑になりましたね。まあ読めないことはないですが、少しだけ型が複雑になったので可読性はどうしても落ちてしまいます。そこで今回登場した**NoInfer**を利用してみましょう

```ts
function createStreetLight<C extends string>(colors: C[], defaultColor?: NoInfer<C>) {
  ...
}
// Argument of type '"blue"' is not assignable to parameter of type '"red" | "yellow" | "green" | undefined'.
```

先程の D という generics を新しく定義する方法に比べたらだいぶスッキリしました。NoInfer のお陰で**C が"blue"も含む可能性があるという型推論を防ぐ**ことができるようになりました。嬉しいですね！

# フロントのコンポーネント設計に応用する

それでは NoInfer の旨味が分かったところで、これをどうフロントエンドコンポーネントの設計に活用するか考えてみましょう

今回は私が普段使用している Vue を例にご紹介します。ただし、基本的には generics を使えるフレームワークであれば同じように NoInfer を扱えます。なので「自分は React ユーザーだよー」という方も参考にしてください

# まとめ

NoInfer で今までよりシンプルに型安全なコードを書けるようになりました

とても便利なのでどんどん活用していきましょう