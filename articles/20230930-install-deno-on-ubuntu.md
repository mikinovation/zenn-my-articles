---
title: "DenoをLinuxにインストールする"
emoji: "🦖"
type: "tech"
topics: ["deno"]
published: true
---

# 1. はじめに

DenoはNode.jsの作者であるRyan Dahl氏が開発した新しいJavaScript/TypeScriptのランタイムである

今回はvimでのプラグイン管理にShougoさんの開発されているdein.vimの後継であるDpp.vimを導入するために必要であった

そこでUbuntu環境にDenoをインストールする方法を紹介する

# 2. インストール方法

まずはドキュメントどおりにDenoをインストールしてみる

```bash
# 事前にunzipをインストールしておく
sudo apt install unzip
# denoをインストールする
curl -fsSL https://deno.land/x/install/install.sh | sh
# .bashrcにパスを通す
echo 'export DENO_INSTALL="/home/ユーザー名/.deno"' >> ~/.bashrc
echo 'export PATH="$DENO_INSTALL/bin:$PATH"' >> ~/.bashrc
# .bashrcを再読み込み
source ~/.bashrc
```

以下のコマンドでバージョンを確認してみよう。バージョンが表示されればインストール成功だ

```bash
deno --version
```


