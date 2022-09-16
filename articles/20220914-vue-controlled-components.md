---
title: "Vue3で単方向データフローと双方向データバインディングによるフォームのコンポーネント制御に立ち向かう"
emoji: "✌"
type: "tech"
topics: ["vue", "frontend"]
published: false
---

# はじめに

# 単方向データフローと双方向データバインディング

## 単方向データフロー

## 双方向データバインディング

# v-modelの仕様

https://vuejs.org/guide/essentials/forms.html

## valueとinputイベント

## 

# フォームコンポーネントはどうあるべきか

## テキストボックスとテキストエリア

## Checkbox

### 

## Checkboxコンポーネントで考える

### 単一Checkboxコンポーネント


簡単な例

```vue
<template>
  <p>Message is: {{ message }}</p>
  <input v-model="message" placeholder="edit me" />
</template>
```