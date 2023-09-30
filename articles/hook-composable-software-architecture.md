---
title: "モジュール性の観点で学ぶhooks・composables設計入門"
emoji: "😜"
type: "tech"
topics: ["React", "Blitz.js", "zod"]
published: false
---

:::message
当記事は
:::

# はじめに

## 本記事の目的

## 対象読者

# モジュール性

## ここでいうモジュールとは

## 凝集度(Cohesion)

1. 機能的凝集(Functional Cohesion)
2. 逐次的凝集(Sequential Cohesion)
3. 通信的凝集(Communicational Cohesion)
4. 手続き的凝集(Procedural Cohesion)
5. 時間的凝集(Temporal Cohesion)
6. 論理的凝集(Logical Cohesion)
7. 偶発的凝集(Coincidental Cohesion)

### 機能的凝集(Functional Cohesion)

モジュール内のコードが共通の目的や機能を持ち、関連する処理がまとめられている。

典型的な例としては簡単なカウンターアプリで使われるようなcomposableだ

```ts
const useCounter = () => {
    const count = ref(0)
    
    const increment = () => {
        count.value++
    }
    
    const decrement = () => {
        count.value--
    }
    
    return {
        count,
        increment,
        decrement
    }
}
```

### 逐次的凝集(Sequential Cohesion)

### 通信的凝集(Communicational Cohesion)

### 手続き的凝集(Procedural Cohesion)

### 時間的凝集(Temporal Cohesion)

### 論理的凝集(Logical Cohesion)

### 偶発的凝集(Coincidental Cohesion)





## 結合度

# 