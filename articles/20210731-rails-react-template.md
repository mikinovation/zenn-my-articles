---
title: "Rails7とReactをモノリス環境で爆速構築"
emoji: "👌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["自己紹介", "経歴"]
published: false
---

# 今回構築した環境

- Docker v20.10.16
- ruby v3.1.2
- rails v7.0.3.1
- mysql v8.0.30
- nginx v1.23.1
- react-router(react-router-dom) v6.3.0

# dockerの構築に必要なファイルの用意

docker-compose.yml

```yaml
version: '3'
services:
  db:
    image: mysql:${MYSQL_VERSION}
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
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