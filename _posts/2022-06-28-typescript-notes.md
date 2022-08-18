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

## 8. 函数

函数包含有名字的函数和匿名函数

```js
// Named function
function add(x, y) {
  return x + y;
}

// Anonymous function
let myAdd = function (x, y) {
  return x + y;
};
```

函数类型

```js
// 参数类型  返回值类型
function add(x: number, y: number): number {
  return x + y;
}

// 无返回值
function add(x: number, y: number): void {
  console.log(x, y);
}

// 可选参数
function add(x: number, y?: number): void {
  console.log(x);
}

// 默认值
function add(x: number, y = 1): void {
  console.log(x);
}

// 剩余参数
function add(first: number, ...rest: string[]): string {
  return first + "" + rest.join(" ");
}
const name = add("jone", "jack", "tom");
```

## 9. 泛型

泛型可以适用于多个类型，且不会丢失信息。使用泛型创建可重用的组件，一个组件可以支持多种类型的数据，这样用户就可以以自己的数据类型来使用组件。

```js
// 不使用泛型

// 固定了number, 不支持number以外的类型
function identity(arg: number): number {
  return arg;
}

// 会导致传入的类型与返回的类型未必相同，任何类型的值都可能被返回
function identity(arg: any): any {
  return arg;
}
```

```js
// 定义泛型
// 我们给identity添加了类型变量T。 T帮助我们捕获用户传入的类型（比如：number），
// 之后我们就可以使用这个类型。 之后我们再次使用了 T当做返回值类型。
// 现在我们可以知道参数类型与返回值类型是相同的了

function identity<T>(arg: T): T {
  return arg;
}

// 使用
// 一、明确指定T是string类型，并做为一个参数传给函数，使用了 <> 括起来而不是 ()
const output = identity < string > "myString"; // type of output will be 'string'

// 二、利用类型推论，即编译器会根据传入的参数自动地帮助我们确定T的类型
const output = identity("myString"); // type of output will be 'string'
```

如果我们想同时打印出 arg 的长度，我们很可能会这样做：

```js
function identity<T>(arg: T): T {
  console.log(arg.length); // Error: T doesn`t have .length
  return arg;
}
```

我们使用了 arg 的.length 属性，但是没有地方指明 arg 具有这个属性。比如 T 为 number 类型，而数字是没有.length 属性的。

```js
function identity<T>(arg: T[]): T[] {
  console.log(arg.length);
  return arg;
}
// 或者
function identity<T>(arg: Array<T>): Array<T> {
  console.log(arg.length);
  return arg;
}
```

## 10. 枚举

使用枚举可以定义一些带名字的常量。使用枚举可以清晰的表达意图或者创建一组有区别的用例。  
**a. 数字枚举**

```js
// 定义了一个数字枚举，默认值从0开始自动增长。即：Direction.up值为0, Direction.down值为1，Direction.left值为2，Direction.right值为3
enum Direction {
  up,
  down,
  left,
  right
}

// 初始化值
enum Direction {
  up=1, // 1
  down, // 2
  left, // 3
  right // 4
}

enum Direction {
  up,   // 0
  down, // 1
  left=0, // 0
  right // 1
}
```

使用枚举

```js
enum Response {
    No = 0,
    Yes = 1,
}

function respond(recipient: string, message: Response): void {
    // ...
}

respond("Princess Caroline", Response.Yes)
```

**b. 字符串枚举**  
字符串枚举没有自增长的行为，每个成员必须用字符串字面量或另外一个字符串枚举成员进行初始化

```js
enum Direction {
  up='UP',
  down     // Error: 枚举成员必须具有初始化表达式
}

// 正确写法
enum Direction {
  up='UP',
  down='DOWN'
}
// 或者(异构枚举)
enum Direction {
  up='UP',
  down='DOWN'
  left = 0,
  right  // 1
}
```

**c. 异构枚举**  
枚举可以混合字符串和数字成员

```js
enum Direction {
  No=0,
  Yes='yes'
}
```

**d. 联合枚举与枚举成员的类型**

```js
enum ShapeKind {
  Circle,
  Square,
}

interface Circle {
  kind: ShapeKind.Circle;
  radius: number;
}

let c: Circle = {
  kind: ShapeKind.Square, // Error: 所需类型来自属性 "kind"，在此处的 "Circle" 类型上声明该属性.  赋值为ShapeKing.Circle才行
  radius: 100,
}
```

```js
enum E {
    Foo,
    Bar,
}

function f(x: E) {
    if (x !== E.Foo || x !== E.Bar) { // Error: x类型E，只有Foo和Bar的值，在这个if里面，x!==E.Bar不会被执行
        // ....
    }
}
```

**e. 运行时的枚举**

```js
enum E {
  X,
  Y
}

function f(obj:{X:number}){
  return obj.X
}
f(E) // E中拥有X属性，且X属性为number类型
```

**f. 反向映射**

```js
enum Enum {
    A
}
let a = Enum.A;
let nameOfA = Enum[a]; // "A"
```

## 11. Symbol 符号

`symbol`一种新的原生类型，就像`number`和`string`一样

```js
const sym1 = Symbol();

const sym2 = Symbol("key"); // 标识为key的，标识只是为了用来区分symbol
```

Symbol 是唯一且不可改变的. Symbol()函数不可以 new

```js
const sym2 = Symbol("key");
const sym3 = Symbol("key");

sym2 === sym3; // false, symbols是唯一的
```

像字符串一样，symbol 可以被用做对象属性的键

```js
let sym = Symbol();

let obj = {
  [sym]: "value",
};

console.log(obj[sym]); // "value"
```

用做类成员

```js
const getClassNameSymbol = Symbol();

class C {
  [getClassNameSymbol]() {
    return "C";
  }
}

let c = new C();
let className = c[getClassNameSymbol](); // "C"
```

## 12. for..of 与 for..in

当一个对象实现了 Symbol.iterator 属性时，我们认为它是可迭代的。 一些内置的类型如 Array，Map，Set，String，Int32Array，Uint32Array 等都已经实现了各自的 Symbol.iterator。 对象上的 Symbol.iterator 函数负责返回供迭代的值。

**for..of 语句**  
for..of 会遍历可迭代的对象，调用对象上的 Symbol.iterator 方法。 下面是在数组上使用 for..of 的简单例子：

```js
let someArray = [1, "string", false];

for (let entry of someArray) {
  console.log(entry); // 1, "string", false
}
```

**for..of 与 for..in 区别**  
均可迭代一个列表；但是用于迭代的值却不同，`for..in`迭代的是对象的`键`的列表，而`for..of`则迭代的是对象的`键对应的值`。

```js
let list = [4, 5, 6];

for (let i in list) {
  console.log(i); // "0", "1", "2",
}

for (let i of list) {
  console.log(i); // "4", "5", "6"
}
```

## 13. export 和 import

**export**  
任何声明（比如变量、函数、类、类型别名或者接口）都能够通过添加`export`关键字来导出。

```js
// 导出接口
export interface StringValidator {
  isAcceptable(s: string): boolean;
}

// 导出类
export class ZipCodeValidator {
  isAcceptable(s: string) {
    return s.length === 5;
  }
}

// 或
class ZipCodeValidator {
  isAcceptable(s: string) {
    return s.length === 5;
  }
}
export { ZipCodeValidator };

// 可重命名 as
class ZipCodeValidator {
  isAcceptable(s: string) {
    return s.length === 5;
  }
}
export { ZipCodeValidator as zip };
```

一个模块可以包裹多个模块，并把他们导出的内容联合在一起通过语法：`export * from "xxx"`。

**import**

```js
// 可重命名 as
import { ZipCom as zip } from "./xxxx";

// 将整个模块导入到一个变量，并通过它来访问模块的导出部分
import * as validator from "./ZipCodeValidator";
let myValidator = new validator.ZipCodeValidator();
```

## 14. 规范

1.  普通类型  
    不要使用`Number`、`String`、`Boolean`、`Object`，应该使用类型`number`、`string`、`boolean`
2.  回调函数返回值类型  
    不要为返回值被忽略的回调函数设置一个`any`类型的返回值类型
    ```js
    // 错误
    function fn(x: () => any) {
      x();
    }
    // 正确
    function fn(x: () => void) {
      x();
    }
    ```
    ......

## 15. 项目配置

[项目配置](https://www.tslang.cn/docs/handbook/tsconfig-json.html)

## 16. .d.ts 文件

声明文件；TypeScript 作为 JavaScript 的超集，在开发过程中不可避免要引用其他第三方的 JavaScript 的库。虽然通过直接引用可以调用库的类和方法，但是却无法使用 TypeScript 诸如类型检查等特性功能。为了解决这个问题，需要将这些库里的函数和方法体去掉后只保留导出类型声明，而产生了一个描述 JavaScript 库和模块信息的声明文件。通过引用这个声明文件，就可以借用 TypeScript 的各种特性来使用库文件了。

假如我们想使用第三方库，比如 jQuery，我们通常这样获取一个 id 是 foo 的元素：

```js
$("#foo");
// 或
jQuery("#foo");
```

但是在 TypeScript 中，我们并不知道 $ 或 jQuery 是什么东西：

```js
jQuery("#foo");
// index.ts(1,1): error TS2304: Cannot find name 'jQuery'.
```

这时，我们需要使用 declare 关键字来定义它的类型，帮助 TypeScript 判断我们传入的参数类型对不对：

```js
declare var jQuery: (selector: string) => any;

jQuery("#foo");
```

declare 定义的类型只会用于编译时的检查，编译结果中会被删除。

上例的编译结果是：

```js
jQuery("#foo");
```

**css 中使用**  
例子：

```js
// style.less.d.ts
export interface StyleLess {
  userCenterWrap: string
  homeWrap: string
}
declare const styles: StyleLess

export default styles

// style.less
. userCenterWrap{
  ...
}
. homeWrap{
  ...
}

// index.tsx
import Styles from './style.less'

<div class={Styles.uerCenterWrap}>
  ...
</div>

```
