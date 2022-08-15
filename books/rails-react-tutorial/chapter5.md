---
title: "ReactにTypescriptを導入"
free: true
---

# ReactにTypescriptを導入

```shell
yarn add -D typescript @types/node @types/react @types/react-dom
```

ファイルをtsファイルに修正しましょう

- 「main.jsx」を「main.tsx」に変更
- 「application.jsx」を「main.tsx」に変更

tsconfigのサンプルは以下です

```json:tsconfig.json
{
  "compilerOptions": {
    "target": "es6",
    "module": "esnext",
    "lib": [
      "dom",
      "dom.iterable",
      "esnext"
    ],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "react",
    "incremental": true,
    "baseUrl": ".",
    "paths": {
      "@/*": [
        "./app/javascript/*"
      ]
    },
  },
  "include": [
    "**/*.ts",
    "**/*.tsx"
  ],
  "exclude": [
    "node_modules"
  ]
}
```

またimportは相対パスで指定するようにしておきます

```tsx:app/javascript/application.ts
import '@/main';
```

これでTypescriptの導入は完了です