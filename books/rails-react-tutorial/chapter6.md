---
title: "ReactにTailwindとdaisyUIを導入"
free: true
---

# RailsにTailwindとdaisyUIを導入

これからtailwindとtailwindをベースとしたUIフレームワークであるdaisyUIを導入します。

先程railsにはSassを導入しましたが、ここからSassをtailwindとdaisyUIに変更します。Sassを利用したい方はこの章を飛ばしてください


```shell
bundle add tailwindcss-rails
rails tailwindcss:install
```

```diff dev:Procfile.dev
web: bundle exec rails s -p 3000 -b '0.0.0.0'
js: yarn build --watch
- css: yarn build:css --watch
+ css: bin/rails tailwindcss:watch
```

daisyuiをインストールします

```shell
yarn add daisyui
```

tailwindcss.config.jsがconfig内に生成されているので以下のように更新します

```diff js:config/tailwindcss.config.js
const defaultTheme = require('tailwindcss/defaultTheme')

module.exports = {
  content: [
    './public/*.html',
    './app/helpers/**/*.rb',
-   './app/javascript/**/*.js',
+   './app/javascript/**/*.{js,jsx,ts,tsx}',
    './app/views/**/*.{erb,haml,html,slim}'
  ],
  theme: {
    extend: {
      fontFamily: {
        sans: ['Inter var', ...defaultTheme.fontFamily.sans],
      },
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/aspect-ratio'),
    require('@tailwindcss/typography'),
+   require('daisyui'),
  ]
}
```

app/stylesheets/application.scssは使わないので削除します

tailwindのCSSファイルのみ読み込むようにしましょう

```diff erb:app/views/layouts/application.html.erb
  <!DOCTYPE html>
  <html>
    <head>
      <title>App</title>
      <meta name="viewport" content="width=device-width,initial-scale=1">
      <%= csrf_meta_tags %>
      <%= csp_meta_tag %>
      <%= stylesheet_link_tag "tailwind", "inter-font", "data-turbo-track": "reload" %>
-     <%= stylesheet_link_tag "application", "data-turbo-track": "reload" %>
      <%= javascript_include_tag "application", "data-turbo-track": "reload", defer: true %>
    </head>
  
    <body>
-     <main class="container mx-auto mt-28 px-5 flex">
-       <%= yield %>
-     </main>
+     <%= yield %>
    </body>
  </html>
```

ボタンでテストしてみましょう。スタイルが反映されていれば成功です

```diff tsx:app/javascript/main.tsx
root.render(
  <StrictMode>
    <h1 className="text-3xl font-bold underline">
      Hello world!
    </h1>
+   <button className="btn btn-primary">ボタン</button>
  </StrictMode>,
);
```