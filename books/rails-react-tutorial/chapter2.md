---
title: "RailsとReactの環境を構築する"
free: true
---

# 今回構築する環境

- Docker v20.10.16
- ruby v3.1.2
- rails v7.0.3.1
- mysql v8.0.30
- nginx v1.23.1
- react v18.2.0

# プロジェクトのディレクトリを作成する

まずはお好きなプロジェクト名を付けましょう。今回は「rails-react-practice」というアプリ名をつけます

```shell
mkdir rails-react-practice
cd rails-react-practice
```

# 必要なファイルの用意

初期のディレクトリの構成は以下のようになっています

```
- docker
  - nginx
    - Dockerfile
    - nginx.conf
  -web
    - Dockerfile
    - entrypoint.sh
- .env
- .env.example
- docker-compose.yml
- Gemfile
- Gemfile.lock
```

docker-compose.ymlのファイルを用意します。一部環境変数が埋め込まれていますが、後ほど.envファイルを作成します

```yml:docker-compose.yml
version: '3'
services:
  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: password
    ports:
      - "${MYSQL_PORT}:3306"
    volumes:
      - ./tmp/db:/var/lib/mysql

  web:
    build:
      context: .
      dockerfile: ./docker/web/Dockerfile
    tty: true
    volumes:
      - .:/app
    ports:
      - "${WEB_PORT}:3000"
    depends_on:
      - db

  nginx:
    build:
      context: .
      dockerfile: ./docker/nginx/Dockerfile
    volumes:
      - ./docker/nginx/nginx.conf:/etc/nginx/conf.d/nginx.conf
    ports:
      - "${NGINX_PORT}:80"
    depends_on:
      - web
```

RailsのDockerfileです。中にはyarnやmysql-clientも入れてあります

```dockerfile:docker/web/Dockerfile
FROM ruby:3.1

RUN mkdir /app && \
    apt-get update -y && \
    apt-get install default-mysql-client nodejs npm -y && \
    npm uninstall yarn -g && \
    npm install yarn -g -y && \
    npm install n -g && \
    n stable && \
    apt-get purge -y nodejs npm

WORKDIR /app

COPY Gemfile /app/Gemfile
COPY Gemfile.lock /app/Gemfile.lock

RUN bundle install

COPY . /app

COPY ./docker/web/entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]
```

Webコンテナ起動時のエントリーポイントです。起動時には毎回bundleとyarnで必要なpackageをinstallするようにしています

```shell:entrypoint.sh

```sh:docker/web/entrypoint.sh
#!/bin/bash
set -e

# Remove a potentially pre-existing server.pid for Rails.
rm -f /app/tmp/pids/server.pid

bundle install
yarn install

bundle exec rails s -p 3000 -b '0.0.0.0'

# Then exec the container's main process (what's set as CMD in the Dockerfile).
exec "$@"
```

nginxのDockerfileです。Webアプリを提供する際には通常NginxのようなWebサーバーを通してサービスを提供するのが一般的かと思います

開発環境に必要かどうかはプロジェクトにもよるかと思いますが、今回は入れておきたいと思います

```Dockerfile:docker/nginx/Dockerfile
FROM nginx:1.23

RUN rm -f /etc/nginx/conf.d/*

COPY docker/nginx/nginx.conf /etc/nginx/conf.d/nginx.conf

CMD /usr/sbin/nginx -g 'daemon off;' -c /etc/nginx/nginx.conf
```

nginxの設定ファイルです

```nginx:docker/nginx/nginx.conf
upstream app {
    server web:3000;
}

server {
  listen 80;
  server_name localhost;

  access_log /var/log/nginx/access.log;
  error_log  /var/log/nginx/error.log;

  root /app/public;

  client_max_body_size 100m;
  error_page 404             /404.html;
  error_page 505 502 503 504 /500.html;
  try_files  $uri/index.html $uri @app;
  keepalive_timeout 5;

  location @app {
    proxy_pass http://app;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Client-IP $remote_addr;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```

Gemfileにはrailsのバージョンを指定しておきます

```Gemfile:Gemfile
source 'https://rubygems.org'
gem "rails", "~> 7.0.3"
```

Gemfile.lockに使う空ファイルを用意しておいてください

```Gemfile.lock:Gemfile.lock
```

環境変数を用意しておきます。通常gitにはコミットしないので.gitignoreへ追加するのを忘れずにしておきましょう

ついでに開発チームへの共有用として.env.exampleファイルも用意しておきます。こちらは共有用なのでAPIキー等重要な情報を追記しないように注意しましょう

.envと.env.example

```.env:.env
# mysql
MYSQL_USERNAME=root
MYSQL_PASSWORD=password
MYSQL_HOST=db
MYSQL_ROOT_PASSWORD=password
MYSQL_PORT=3306

# web
WEB_PORT=3000

# nginx
NGINX_PORT=80
```

# Railsプロジェクトの作成

ここまでで必要なファイルは揃ったのでプロジェクトを作成します。以下のコマンドを叩きましょう

```shell
docker-compose run web rails new . --force --no-deps --database=mysql
```

筆者はWindowsのUbuntuを使っておりアプリの作成時に権限エラーが出ました

```shell
failed to solve: error from sender: open /home/mikinovation/projects/rails-react-template/tmp/db/#innodb_redo: permission denied
```

なので必要であれば権限を付与してきましょう。これからもコンテナ上で作成したファイルへ権限を与えることになります。このようなコマンドはチートシートにコピーしておくと何かと便利です

```shell
sudo chown $USER:$USER -R *
```

これでプロジェクトの作成が完了しました。ビルドして起ち上げるとデータベースの接続エラーが出てくるかも知れませんが、http://localhostにアクセスしてRailsのエラーページがでてくれば成功になります！

```shell
docker compose build
docker compose up
```

![rails](https://storage.googleapis.com/zenn-user-upload/26682e0bb791-20220806.png)

# データベースの設定

それではデータベースの設定を進めます。データベース設定でdefaultを以下のように変更しておきましょう

```yml:config/database.yml
default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: <%= ENV["MYSQL_USERNAME"] %>
  password: <%= ENV["MYSQL_PASSWORD"] %>
  host: <%= ENV["MYSQL_HOST"] %>
```

また環境変数をrailsで取得できるようにするためgemを追加します

**Gemfile**

```shell
gem 'dotenv-rails'
```

以下のコマンドでdockerのwebコンテナに入り、gemのインストールをしましょう。gemをinstallしたらデータベースを作成します

```shell
docker compose exec web /bin/sh
bundle install
bundle exec rails db:create
```

railsのトップ画面が表示できれば完了です

![rails-top](https://storage.googleapis.com/zenn-user-upload/b1f760aa5261-20220806.png)

# Reactの環境構築

続いてReactの環境構築を行います。Rails7ではWebpackerではなく、esbuildが推奨されています

esbuildは何と言ってもビルド速度が高速で開発体験がとても良いです。

まずは必要なGemを追加しましょう

```Gemfile:Gemfile
gem 'jsbundling-rails'
gem 'cssbundling-rails'
```

以下のコマンドで初期設定に必要なファイルをインストールします

```shell
rails javascript:install:esbuild
rails css:install:sass
```

またrails serverのコマンドだけでなくJSやCSSのファイル変更をwatchして開発する必要があります

以下のようにentrypoint.shを編集します

```diff sh:docker/entrypoint.sh
-  ./bin/dev
+  bundle exec rails s -p 3000 -b '0.0.0.0'
```

先程esbuild関連のファイルをインストールして生成されたProcfile.devを以下のように更新します。こうすることでlocalhostからのアクセスできるようになりました

```dev:Procfile.dev
web: bundle exec rails s -p 3000 -b '0.0.0.0'
js: yarn build --watch
css: yarn build:css --watch
```

続いてrailsアプリを表示するためのhome画面を作成します。まずはコントローラーの作成です

```shell
rails g controller home index
```

コントローラーが作成されているのでtemplateに```main/index```をレンダリングするよう指定しましょう

これ以降もReactのViewを表示するためのパスには指定する場合は```main/index```を指定するようにします

```rb:app/controllers/home_controller.rb
class HomeController < ApplicationController
  def index
    render template: 'main/index'
  end
end
```

続いてreactのビューをレンダリングするためのrootとなるdomを用意します。各画面に``id="app"```の付いたViewを使い回すことができるようになります

```erb:app/views/main/index.html.erb
<div id="app"></div>
```

それではreactをinstallしましょう

```shell
yarn add react react-dom
```

初期の状態だとapplication.jsを読み込むためmain.tsxをimportしてきます

またルートコンポーネントをmain.jsxに追加します

```js:app/javascript/application.js
import './main';
```

```jsx:app/javascript/main.jsx
import React, { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';

const container = document.getElementById('app');
const root = createRoot(container);

root.render(
  <StrictMode>
    <h1>Hello World</h1>
  </StrictMode>,
);
```

ついでにesbuildはいつでも設定を変えられるようにしておいた方が便利なため、esbuild.config.jsを作成しておきます

```js:esbuild.config.js
const path = require('path');

require("esbuild").build({
    entryPoints: ["./application.js"],
    bundle: true,
    outdir: path.join(process.cwd(), "app/assets/builds"),
    absWorkingDir: path.join(process.cwd(), "app/javascript"),
    watch: true
}).catch(() => process.exit(1));
```

これでコンテナを起ち上げ直して「Hello World」が表示されれば完了です！ お疲れ様でした

```shell
docker compose up
```

![Hello World](https://storage.googleapis.com/zenn-user-upload/0a39585db1ba-20220807.png)