---
title: "ReactにStorybookを導入"
free: true
---

# 試しにコンポーネントを作ってみる

今回はアトミックデザインに従ってコンポーネントを構築していきます

まずは単純なボタンコンポーネントを作成しましょう。

```tsx:app/javascript/components/atoms/Button/Button.tsx
import React, {forwardRef} from "react";
import {ButtonProps} from "./Button.type";
import {BUTTON_COLOR, BUTTON_SIZE} from "./Button.const";

export const Button = forwardRef<
    HTMLButtonElement,
    ButtonProps
    >(function ButtonBase({color = BUTTON_COLOR.PRIMARY, size = BUTTON_SIZE.MD, ...props}: ButtonProps, ref) {
    return (
        <button {...props} ref={ref} className={`btn ${color} ${size}`} />
    );
});
```

```tsx:app/javascript/components/atoms/Button/Button.type.ts
import {ComponentPropsWithoutRef, ReactNode} from "react";
import {BUTTON_COLOR, BUTTON_SIZE} from "./Button.const";


export type ButtonColor = typeof BUTTON_COLOR[keyof typeof BUTTON_COLOR];

export type ButtonSize = typeof BUTTON_SIZE[keyof typeof BUTTON_SIZE];

export type BaseButton = {
    color?: ButtonColor;
    size?: ButtonSize;
    children: ReactNode
}

export type ButtonProps = ComponentPropsWithoutRef<"button"> & BaseButton
```

```tsx:app/javascript/components/atoms/Button/Button.const.ts
export const BUTTON_COLOR = {
    PRIMARY: "btn-primary",
    SECONDARY: "btn-secondary",
    DISABLED: "btn-disabled",
} as const;

export const BUTTON_SIZE = {
    XS: "btn-xs",
    SM: "btn-sm",
    MD: "btn-md",
    LG: "btn-lg",
} as const;
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

storybookでのモック用にpostcssを入れておきます

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

storybookをビルドする場合にはstorybook-staticが生成されるので.gitignoreに追加しておきます

```.gitignore
storybook-static
```

またdocker compose up時にstorybookも一緒に起動しておきたいので以下を追記しましょう

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

最後にボタンコンポーネントのストーリーを書いて完了です

このストーリーはドキュメントであり、テストでもあります。これによりドキュメントやテストを書く開発者体験が大きく向上します

```tsx:app/javascript/components/atoms/Button/Button.stories.tsx
import React from 'react';
import { ComponentStory, ComponentMeta } from '@storybook/react';

import { Button } from './Button';
import {BUTTON_COLOR, BUTTON_SIZE} from "./Button.const";

export default {
  title: 'components/atoms/Button',
  component: Button,
  argTypes: {
    color: {
      options: Object.values(BUTTON_COLOR),
      control: { type: 'select' }
    },
    size: {
      options: Object.values(BUTTON_SIZE),
      control: { type: 'select' }
    },
    children: { control: 'text' },
  },
} as ComponentMeta<typeof Button>;

const Template: ComponentStory<typeof Button> = (args) => <Button {...args} />;

export const Default = Template.bind({});
Default.args = {
  children: 'ボタン'
};
```

# StorybookにSnapshotテストを導入

今回はCSS Moduleも使うことがあるのでidentity-obj-proxyを合わせてインストールしています。更にjestは通常nodeのランタイムで実行されるのでブラウザと同じjsdomで実行できる環境を作成するためjest-environment-jsdomが必要です

```shell
yarn add -D jest @storybook/addon-storyshots identity-obj-proxy jest-environment-jsdom babel-jest @babel/preset-env react-test-renderer @babel/preset-typescript @types/jest
```

```js:babel.config.js
module.exports = {
    presets: [
        '@babel/preset-env',
        ['@babel/preset-react', {runtime: 'automatic'}],
        '@babel/preset-typescript',
    ],
};
```

ルートディレクトリにstorybookのjestデフォルト設定となるファイルを配置します

```js:storybook.test.js
import initStoryshots from '@storybook/addon-storyshots';
initStoryshots();
```

ファイルとmdxはテストの範囲外でエラーとなるためモックに置き換えておきます

```js:app/javascript/__mocks__/fileMock.js
module.exports = 'test-file-stub';
```

```js:app/javascript/__mocks__/mdxMock.js
module.exports = {}
```

package.jsonにjestの設定を追加します。resolutionsにreact-test-rendererを指定していますが、これはstoryshots内のreact-test-redererが、2022年8月28日現在react18をサポートしていないためです

```diff json:package.json
"scripts": {
    "build": "node esbuild.config.js",
    "storybook": "start-storybook -p 6006",
    "build-storybook": "build-storybook",
+   "test": "jest"
},
+ "jest": {
+   "testEnvironment": "jest-environment-jsdom",
+   "moduleNameMapper": {
+     "\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$": "<rootDir>/app/javascript/__mocks__/fileMock.js",
+     "\\.(css|sass|scss|less)$": "identity-obj-proxy"
+   }
+ },
  ------------------------------------------------------------
+ "resolutions": {
+   "react-test-renderer": "^18.0.0-rc.0"
+ }
```

これでスナップショットが取れれば成功です。ルートディレクトリに`__snapshots__`が生成されています

あとはスナップショットを撮りたいコンポーネントのストーリーを書けば、簡単に差分テストを実行することができるようになります

```shell
yarn test storybook.test.js
```

:::message
現在続きを執筆中です
:::