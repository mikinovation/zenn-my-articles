---
title: "[2021年度版]フロントエンドフレームワークの比較"
emoji: "😜"
type: "tech"
topics: ["フロントエンド", "フレームワーク", "React", "Vue", "Angular"]
published: false
---

# Webフロントエンドフレームワークの歴史

近年のWebフロントエンド開発では、あらゆるビジネスにおいてWebサイトやWebアプリケーションのUIやUXの改善、開発生産性の向上という面でWebフロントエンドフレームワークの技術が求められるようになっています

かつてはサーバーサイドでHTMLやCSSを描画しブラウザに返すという処理が一般的でした

その後Javascriptが誕生したのは1995年のことです

この年にはWindows95が誕生し、インターネット時代の幕開けとなる年でもあります

当時ブラウザ上でJavascriptを動かそうとしても、ブラウザやPCのマシンスペックが低く読み込みが遅くなってしまうという問題がありました

現在ではパソコンの処理能力というのも大幅に向上しており、Javascriptを動かしてもブラウザ上である程度の処理であれば、非常に快適な操作が実現可能となっています

そして時代は進み2006年Javascriptのライブラリである「jQuery」がもてはやされることとなります

jQueryを使うことで簡単にDOM操作やイベント処理、AjaxとRest APIを利用した非同期での通信を簡単に実現することができるようになりました

# SPA(Single Page Appplication)の台頭

近年ではSPAによるアプリケーションの構築が人気を集めています

従来型のアプリケーションではユーザーがブラウザから、あるURLにアクセスしようとするとサーバーサイドへリクエストが飛び、

リクエストを受け取ったサーバーはURLの情報に対応した情報をHTMLとして描画、クライアント側(ブラウザ)にレスポンスとして返すという方式をとっていました

しかしブラウザの進化やHTML5の登場によってSPAというフロントエンドのアーキテクチャが台頭し始めます

SPAでは最低限必要なHTMLやCSS、JS等を最初に読み込み、ページが表示された段階でDOMを描画する仕組みとなっています

これにより一度ページを表示するだけでリロードせずに画面を切り替えるといったことができるようになりました

ページを切り替える際にページのリロードがないので、サーバーリクエストから次のレスポンスを受け取るまでの長い待ち時間もないので、非常に快適なユーザー体験を提供することができるようになります

必要なサーバー側のデータに関しては非同期通信でサーバーにリクエストを送信し、ページ全体ではなく必要なデータのみを取得、再描画することが可能になりました

現在ではReact、Vue、Angular、Ember.js、Meteor.jsなど多くのフロントエンドフレームワークやライブラリでこの仕組みが採用されています

# フロントエンドフレームワークによるSSRの登場

# WebAssembly(WASM)の認定

## Javascriptの問題

普段私達が使うChromeやSafariなどのブラウザ上では基本的にJavascriptが使用されます

Javascriptが登場した時代にはHTMLのDOMに対して簡単な動きをちょっとつけるためだけに使われており、Javascriptに対したスペックは求められていませんでした

しかし時代は進みJavascriptによる複雑な処理や計算だったり、HTML5の登場、計算処理の遅いモバイル端末などでも快適なWebページの操作が求められるようになってきます

Javascriptはスクリプト言語ですし、計算処理を早めるのにも限界があります

## asm.jsの登場

そこで登場したのがasm.jsです

http://asmjs.org/

## WebAssemblyの登場

そして遂にWebAssemlyが登場します

これはWebブラウザがバイナリ形式のアセンブリ(機械語)を実行できるようにした新しいフォーマットとなります

WebAssemblyを直接書くのは難しいですが、私達が書く必要は基本的にありません

現在ではCやC++、C#、Rust、Go、TypescriptからコンパイルしてWebAssemblyを出力できるようになっています

私達が外国語話者とコミュニケーションを取るときも通訳がいるときよりも、通訳を介さずに直接メッセージを伝えた方が早く伝わりますよね

それと同じです

WebAssemblyという直接機械に命令を送ることのできる言語を予め用意しておいた方が、Javascriptを機械語に置き換えてから命令をするより早く計算ができるようになります





# Javascriptのフレームワークの特徴

特に規模の大きいプロジェクトや複数人でのシステム開発の場合、コードの書き方に規則が課せられていた方が生産性が向上します

何をどこへ書くかが大まかに決まっているため、メンバー間でのコードの書き方に認識齟齬が生まれにくくなるのです

例えばVueやReactにはSPA(シングルアプリケーション)や

これが

## プロジェクトによってはオーバースペック



# 主なフロントエンドフレームワークやライブラリ

## jQuery

現在最も多くのWebサイトで使われているライブラリです

普段SPA(シングルページアプリケーション)で大規模なサービスを開発しているフロントエンジニアから酷評を受けたりしていますが、jQueryは未だに素晴らしいライブラリとなっています

ただしプロジェクトとして採用するかどうかはよく考えなければなりません

どういったプロジェクトでjQueryを採用するのがいいのか考えてみましょう

jQueryのメリットとして以下のようなものがあります

- DOMの操作を簡単に書ける
- Ajaxを利用した非同期通信に関する処理が簡単に実装できる
- ブラウザ間の差異を吸収してくれる
- 豊富なプラグインがある

デメリット

- 処理が遅くなる可能性がある
- 状態の管理が煩雑になりやすい
- 


## React

Reactは現在最も人気を集めているJavascriptライブラリの一つになります

Reactはフレームワークでなくライブラリではあります