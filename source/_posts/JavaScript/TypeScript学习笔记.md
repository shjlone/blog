---
title: TypeScript入门
tags: TypeScript JavaScript
---


在JavaScript的基础上添加了类型系统，可以在编译时检查类型，提高代码的可读性和可维护性。


约定使用 TypeScript 编写的文件以 .ts 为后缀，用 TypeScript 编写 React 时，以 .tsx 为后缀。

```shell
#全局安装typescript
npm install -g typescript
#编译
tsc xxx.ts
#运行
node xxx.js
```

## 基础

### 数据类型

JavaScript的类型分为：

- 原始数据类型：布尔值、数值、字符串、null、undeined、Symbol、BigInt
- 对象类型
  - 接口
  - 数组

```typescript

const x: boolean = false;
const y: string = 'hello';
const z: number = 123;
const a: bigint = 123n;
const b: symbol = Symbol('foo');
const c: object = { foo: 123 };
let d: undefined = undefined;//表示为定义，以后可能会有定义
const d: null = null;

let x:'hello'; //值类型

x = 'hello'; // 正确
x = 'world'; // 报错

let x:string|number; //联合类型

x = 123; // 正确
x = 'abc'; // 正确

//交叉类型
let obj:
  { foo: string } &
  { bar: string };

obj = {
  foo: 'hello',
  bar: 'world'
};

type A = { foo: number };

type B = A & { bar: number };


type Age = number;

let age:Age = 55;

// 'hello' // 字面量
// new String('hello') // 包装对象


### 接口

// 接口类型
interface Person {
  name: string;
  age?: number; //?表示可选属性
  readonly id: number; //只读属性
  [propName: string]: any; //任意属性，TypeScript 实际上会关闭这个变量的类型检查。即使有明显的类型错误，只要句法正确，都不会报错
  x: unknown; //未知类型，为了解决any的污染问题，unknown类型只能赋值给unknown和any类型，但是不能赋值给其他类型；不能直接调用unknown类型变量的方法和属性
}


let tom: Person = {
  id: 89757,
  name: 'Tom',
  age: 25,
  gender: 'male'
};


```


### 数组

```typescript

let arr: number[] = [1, 2, 3, 4, 5];
let arr: (number|string)[] = [1, 2, 3, 4, '5'];
let arr: Array<number> = [1, 2, 3, 4, 5];

```



### 元组

```typescript

let t:[number] = [1];


//元组的成员可以添加成员名，这个成员名是说明性的，可以任意取名，没有实际作用。
type Color = [
  red: number,
  green: number,
  blue: number
];

const c:Color = [255, 255, 255];

```


### 声明文件

声明文件必需以 .d.ts 为后缀。


库的使用场景主要有以下几种：

- 全局变量：通过 <script> 标签引入第三方库，注入全局变量
- npm 包：通过 import foo from 'foo' 导入，符合 ES6 模块规范
- UMD 库：既可以通过 <script> 标签引入，又可以通过 import 导入
- 直接扩展全局变量：通过 <script> 标签引入后，改变一个全局变量的结构
- 在 npm 包或 UMD 库中扩展全局变量：引用 npm 包或 UMD 库后，改变一个全局变量的结构
- 模块插件：通过 <script> 或 import 导入后，改变另一个模块的结构


在 ES6 模块系统中，使用 export default 可以导出一个默认值，使用方可以用 import foo from 'foo' 而不是 import { foo } from 'foo' 来导入这个默认值

注意，只有 function、class 和 interface 可以直接默认导出，其他的变量需要先定义出来，再默认导出

```ts
    declare var 声明全局变量
    declare function 声明全局方法
    declare class 声明全局类
    declare enum 声明全局枚举类型
    declare namespace 声明（含有子属性的）全局对象
    interface 和 type 声明全局类型
    export 导出变量
    export namespace 导出（含有子属性的）对象
    export default ES6 默认导出
    export = commonjs 导出模块
    export as namespace UMD 库声明全局变量
    declare global 扩展全局变量
    declare module 扩展模块
    /// <reference /> 三斜线指令
```

## 函数

```typescript

//变量被赋值为一个函数的写法
// 写法一
const hello = function (txt:string) {
  console.log('hello ' + txt);
}

// 写法二
const hello:
  (txt:string) => void
= function (txt) {
  console.log('hello ' + txt);
};

type MyFunc = (txt:string) => void;

const hello:MyFunc = function (txt) {
  console.log('hello ' + txt);
};


// 属性类型以分号结尾
type MyObj = {
  x:number;
  y:number;
};

// 属性类型以逗号结尾
type MyObj = {
  x:number,
  y:number,
};

```


### 解构赋值


## import中@的作用

路径映射，可以在tsconfig.json中配置

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@nui/*": ["src/*"]
    }
  }
}
```

使用

```JavaScript

import { YYYY } from '@nui/xxx';

```


## 扩展全局变量的类型

```ts
interface String {
    // 这里是扩展，不是覆盖，所以放心使用
    double(): string;
}

String.prototype.double = function () {
    return this + '+' + this;
};
console.log('hello'.double());


```

## 参考




- [官方手册](www.typescriptlang.org/docs/handbook/basic-types.html)

- [阮一峰TypeScript教程](https://wangdoc.com/typescript/)


- [TypeScript 入门教程](https://ts.xcatliu.com/)