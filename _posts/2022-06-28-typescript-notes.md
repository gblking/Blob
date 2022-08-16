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
  readonly width: string  // 只读属性, 不能对其值进行修改
}
// readonly VS const
// 最简单的判断该用readonly还是const的方法是看要把它做为变量使用还是做为一个属性。做为变量使用的话用const，若做为属性则使用readonly。
```

存在不确定数量的额外属性，定义方式：

```js
interface Config {
  color: string
  [propName: string]: any
}
```

函数类型

```js
interface SearchFunc {
  (source: string, subString: string): boolean;
}
```

```js
let mySearch: SearchFunc;
mySearch = function (source: string, subString: string) {
  let result = src.search(sub);
  return result > -1;
};

// 或者

let mySearch: SearchFunc;
mySearch = function (src: string, sub: string): boolean {
  let result = src.search(sub);
  return result > -1;
};
```

类类型：实现接口(implements)  
TypeScript 能够用它来明确的强制一个类去符合某种契约

```js
    interface nameS {
      name: string
      tick(): boolean
    }
    // 实现接口, 接口中的属性/方法必须全部实现, 可以增加
    class name implements nameS {
      name: string = 'dd'
      tick = () => {
        return true
      }
      click = ():void => {
        console.log('click')
      }
    }
```

继承接口

```js
interface Shape {
  color: string
}

interface Square extends Shape {
  sideLength: number
}

let square = <Square>{}
square.color = 'red'
square.sideLength = 10

// 一个接口可以继承多个接口，创建出多个接口的合成接口
interface Square extends Shape, PenStroke {
  ...
}
```

## 6. 类 class

类用于创建可重用的组件

```js
class Square {
  name: string;
  age: number = 30;
  // 构造函数
  constructor(data: number) {
    this.age = data;
  }
}
const example = new Square(26); // 实例化
```

**继承**

```js
class Square {
  name: string;
  constructor(theName: string) {
    this.name = theName;
  }
  move() {
    console.log("square move");
  }
}
// 继承 extends
class Snake extends Square {
  person: string = "a";
}
const a = new Snake("b");
console.log(a.move()); // square move
console.log(a.name); // b
console.log(a.person); // a
```

Square 被称为`基类、超类`，而 Snake 被称为`子类、派生类`

Snake 继承了 Square 的功能，包括属性和方法，因此我们可以创建一个 Snake 实例，它能够 move()以及访问 name、person

当派生类中也包含一个构造函数 constructor，则必须调用`super()`，它会执行基类的构造函数。而且，在构造函数里访问 `this`的属性之前，我们 一定要调用 `super()`。 这个是 TypeScript 强制执行的一条重要规则。

```js
class Animal {
  name: string;
  constructor(theName: string) {
    this.name = theName;
  }
  move(distanceInMeters: number = 0) {
    console.log(`${this.name} moved ${distanceInMeters}m.`);
  }
}

class Snake extends Animal {
  age: number;
  constructor(name: string, age: number) {
    super(name); // 会执行基类Animal的构造函数
    this.age = age; // this之前需调用super()
  }
  move(distanceInMeters = 5) {
    console.log("Slithering...");
    super.move(distanceInMeters);
  }
}
const a = new Snake("df", 34);
a.move(); // Slithering...    df moved 5m.
// Snake中存在move()，执行Snake中的，其中调用了super.move()即调用基类的move()
```

**public、private、readonly、protected 修饰符**

1.**public** 公共 `默认`  
 在 typescript 中，成员都默认为`public`, 也可以手动明确的将一个成员标记成`public`

```js
class Animal {
  public name: string
  public constructor(theName:string){
    this.name = theName
  }
}
```

2.**private** 私有  
 不能在声明它的类的外部访问

```js
class Animal {
  private name: string
  constructor(theName:string){
    this.name = theName
  }
}

new Animal("cat").name // 错误：'name'是私有的

class Snake extends Animal {
  constructor(name:string){
    super(name)
    this.name = 'ss' // 错误： 'name'为私有属性，只能在Animal类中使用
  }
}
```

3.**protected** 受保护的  
 与 private 修饰符的行为很相似，但有一点不同，`protected`成员在派生类中仍然可以访问。

```js
class Animal {
  protected name: string
  constructor(theName:string){
    this.name = theName
  }
}

class Snake extends Animal {
  constructor(name:string){
    super(name)
    this.name = 'ss' // 正确
  }
}

new Animal("cat").name // 错误：'name'是受保护的
```

4.**readonly** 只读  
 将属性设置为只读，只读属性必须在声明时或构造函数里被初始化。

```js
class Octopus {
    readonly name: string;
    readonly numberOfLegs: number = 8;
    constructor (theName: string) {
        this.name = theName;
    }
}
let dad = new Octopus("Man with the 8 strong legs");
dad.name = "Man with the 3-piece suit"; // 错误! name 是只读的.
```

针对上面这种只读属性，在构造函数中立刻将 theName 的值赋值给 name，推出了`参数属性`简化上面的写法。参数属性可以方便的让我们在一个地方定义并初始化一个成员

```js
class Octopus {
    readonly numberOfLegs: number = 8;
    constructor(readonly name: string) { // 仅在构造函数中使用readonly name: string参数来创建和初始化name成员
    }
}
```

**get/set 存取器**  
TypeScript 支持通过 getters/setters 来截取对对象成员的访问。 它能帮助你有效的控制对对象成员的访问。
为防止成员被随意的设置，将成员设置为`private`,通过 get/set 对成员进行读取/赋值。

```js
 class Animal {
  private _fullName: string='jack'

  get fullName():string {
    return this._fullName
  }

  set fullName(newName:string){
    if(newName==='new name'){
      this._fullName = newName
    } else {
      console.log('Error: set name error')
    }
  }
 }

 new Animal().fullName  // jack
 new Animal()._fullName // 错误： _fullName为私有属性
```

`注意：` 只带有`get`不带`set`的存期器自动被推断为`readonly`。

**abstract** 抽象类  
抽象类做为其他派生类的基类使用，不能直接被实例化。不同于接口，抽象类可以包含成员的实现细节。 abstract 关键字是用于定义抽象类和在抽象类内部定义抽象方法。

```js
abstract class Animal {
  constructor(readonly name: string = "ss") {}
}

new Animal("a") // 错误： 无法创建抽象类的实例

// 做为派生类的基类使用
class Snake extends Animal {
  constructor(){
    super()
  }
}
new Snake()
```

抽象类中的抽象方法不包含具体实现并且必须在派生类中实现。抽象方法的语法与接口方法相似。两者都是定义方法签名但不包含方法体。

```js
abstract class Animal {
  abstract move():void
}

class Snake extends Animal {
  move():void {
    console.log('abc')
  }
}
```

## 7. 接口和类的区别

1. 接口只声明成员、方法，不做实现(成员不能赋默认值、方法只声明，不含具体实现)
2. 类声明并实现方法(成员可默认赋值，方法可包含具体的实现)
