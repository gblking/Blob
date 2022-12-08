---
layout: post
title: typescript 报错
categories: typescript
tags: [typescript]
excerpt: 开发中碰到的错误，以及解决方法
---

## hidden article

## 1. 元素隐式具有 "any" 类型，因为类型为 "any" 的表达式不能用于索引类型 "xxx"

```js
  const formData = new FormData()

  interface ReqDataInterface {
    app_id: string
    version: string
    charset: string
  }

  const reqData: ReqDataInterface = {
    app_id: 'dzyzgzptkrzje',
    version: '1.0',
    charset: 'UTF-8',
  }

  for (let key in reqData) {
    formData.append(key, reqData[key]) // reqData[key]: 元素隐式具有 "any" 类型，因为类型为 "any" 的表达式不能用于索引类型 "ReqDataInterface"
  }
```

#### 解决方案

keyof 将 ReqDataInterface 这个对象类型映射成了一个联合类型

```js
let key: keyof ReqDataInterface  // "app_id" | "version" | "charset"
 for (key in reqData) {
    formData.append(key, reqData[key])
  }
```
