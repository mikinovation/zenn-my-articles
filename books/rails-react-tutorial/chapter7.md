---
title: "ReactにStorybookを導入"
free: true
---

# 試しにコンポーネントを作ってみる

今回はアトミックデザインに従ってコンポーネントを構築していきます

まずは単純なボタンコンポーネントを作成しましょう。

```tsx:app/javascript/components/Button/Button.tsx
import clsx from "clsx";
import React, { ComponentPropsWithoutRef, forwardRef } from "react";

export const Button = forwardRef<
    HTMLButtonElement,
    ComponentPropsWithoutRef<"button">
    >(function ButtonBase({ className, ...props }, ref) {
    return (
        <button {...props} ref={ref} className={clsx(className, 'btn btn-primary')} />
    );
});
```

# Storybookの導入

それでは本題です。storybookを導入してみましょう。

```shell
npx storybook init
```

storybookの設定をプロジェクトに合わせて更新します

```js:.storybook/main.js
module.exports = {
  "stories": [
    "../app/javascript/**/*.stories.mdx",
    "../app/javascript/**/*.stories.@(js|jsx|ts|tsx)"
  ],
  "addons": [
    "@storybook/addon-links",
    "@storybook/addon-essentials",
    "@storybook/addon-interactions"
  ],
  "framework": "@storybook/react"
}
```

ルートディレクトに配置されたstoriesディレクトリもjavascript内に配置しておきましょう

```stories```→```app/javascript/stories```


```diff js:.storybook/main.js
"addons": [
    "@storybook/addon-links",
    "@storybook/addon-essentials",
    "@storybook/addon-interactions",
+   {
+     name: '@storybook/addon-postcss',
+     options: {
+       postcssLoaderOptions: {
+         implementation: require('postcss'),
+       },
+     },
+   }
]
```

```shell
yarn add -D @storybook/addon-postcss postcss
```

```.gitignore
storybook-static
```

```diff yml:docker-compose.yml
  web:
    build:
      context: .
      dockerfile: ./docker/web/Dockerfile
    tty: true
    volumes:
      - .:/app
    ports:
      - "${WEB_PORT}:3000"
+     - "${STORYBOOK_PORT}:6006"
    depends_on:
      - db
```

```.env
# storybook
STORYBOOK_PORT=6006
```