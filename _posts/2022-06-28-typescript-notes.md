---
layout: post
title: typescript 笔记
categories: typescript
tags: [typescript]
excerpt: typescript 笔记
---

## 1. "?" 和 "!"

**`a. 可选属性 ?`**

```js
interface Test {
  height?:number // 等价于height: number | undefined
  width:number
}

function testfunc(test: Test){
  test.height = undefined // success
  test.width = undefined  // fail Type 'undefined' is not assignable to type 'number'.
}
```

**`b. 非空断言操作符 !`**  
[!] 表示在此处告诉编译器，此成员不会为 null 和 undefined

```js
class Test {
  height!: number
  test(){
    this.height = null // fail Type 'null' is not assignable to type 'number'.
  }
}
```

## 2. 基础类型

```js
// 布尔值
let isDone: boolean = true;

// 数字
let decLiteral: number = 6;

// 字符串
let name: string = "bob";

// 数组
let data: number[] = [1, 2, 3];
let list: Array<number | string> = [1, 2, "a"];

// any
let notSure: any = 4;

// 元组 Tuple  表示一个已知元素数量和类型的数组
let x: [string, number];
x = ['hello', 10]; // OK
x = [10, 'hello']; // Error
x = ['hello', 10, 20]; // Error

// 枚举 enum
enum Color {Red, Green, Blue}
let c: Color = Color.Green;

enum Color {Red = 1, Green, Blue} // 指定下标
enum Color {Red = 1, Green = 2, Blue = 4}
let colorName: string = Color[4] // Blue
```

## 3. Void

表示没有任何类型，当一个函数没有返回值时，你通常会见到其返回值类型是`void`

```js
function warnUser(): void {
  console.log("This is my warning message");
}
```

## 4. 类型断言

你会比 TypeScript 更了解某个值的详细信息。 通常这会发生在你清楚地知道一个实体具有比它现有类型更确切的类型。

通过类型断言这种方式可以告诉编译器，“相信我，我知道自己在干什么”。

```js
let someValue: any = "this is a string";

let strLength: number = (<string>someValue).length;

```

```js
//as 语法
let someValue: any = "this is a string";

let strLength: number = (someValue as string).length;
```

## 5. 接口 interface

接口的作用就是为这些类型命名和为你的代码或第三方代码定义契约。

```js
interface Color {
  label: string
  value: string
}

const color: Color = {
  label: '颜色',
  value: 'red'
}
```

```js
interface Config {
  color?: string  // 可选属性
  readonly width: string  // 只读属性
}
```

存在不确定数量的额外属性，定义方式：

```js
interface Config {
  color: string
  [propName: string]: any
}
```
