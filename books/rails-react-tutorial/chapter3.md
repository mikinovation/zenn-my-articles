---
title: "管理者の作成"
free: true
---

# Railsにdevise_token_authの導入

```Gemfile:Gemfile
gem 'devise_token_auth', '>= 1.2.0', git: "https://github.com/lynndylanhurley/devise_token_auth"
```

```shell
bundle install
rails g devise:install
rails g devise_token_auth:install Admin auth
```

```shell
bundle exec rails db:migrate:reset
```


```shell
rails g controller v1/auth/registrations
```

```Gemfile
gem 'rails-i18n', '~> 7.0.0'
```

