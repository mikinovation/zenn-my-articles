---
title: "ドキュメント類の環境構築"
free: true
---

# Railsにannotateを導入

```Gemfile
group :development do
  gem 'annotate'
end
```

```shell
bundle install
rails g annotate:install
```

```shell
annotate
```