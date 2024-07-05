---
title: JavaScript入门
tags: JavaScript
---


先了解一下JS的历史：[Javascript诞生记](https://www.ruanyifeng.com/blog/2011/06/birth_of_javascript.html)
。基本的语法，从官方文档开始：[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript)

## 一些知识点


原义字符|等价字符引用
--|--|
`<` |`&lt;`
`>` |`&gt;`
`"` | &quot;
'  | &apos;
&  | &amp;



```html
<p>HTML 中用 <p> 来定义段落元素。</p>

<p>HTML 中用 &lt;p&gt; 来定义段落元素</p>

```

### HTML头

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="utf-8" />
    <title>我的测试页面</title>
    <link rel="icon" href="favicon.ico" type="image/x-icon" />
    <link rel="stylesheet" href="my-css-file.css" />
    <script src="my-js-file.js" defer></script>
  </head>
  <body>
    <p>这是我的页面</p>
  </body>
</html>


```

- title：表示页面标题
- meta：元数据
- link：自定义图标
- script：引入需要加载的js文件


## 常见标签

```html
<h1>标题元素标签，数字表示不同大小</h1>
<h2></h2>
<h3></h3>
<h4></h4>
<h5></h5>
<h6></h6>

<!--无序列表 -->
<ul>
    <li>1</li>
    <li>2</li>
    <li>3</li>
</ul>

<p>
  我创建了一个指向 <a href="https://www.mozilla.org/zh-CN/">Mozilla 主页</a>的链接。
</p>

<!--上标和下标 -->
<p>
  咖啡因的化学方程式是
  C<sub>8</sub>H<sub>10</sub>N<sub>4</sub>O<sub>2</sub>。
</p>
<p>如果 x<sup>2</sup> 的值为 9，那么 x 的值必为 3 或 -3。</p>



<div> </div>

<!-- 可在段落中进行换行  -->
<br/>

<!-- 元素在文档中生成一条水平分割线  -->
<hr/>

<img src="images/dinosaur.jpg"
     alt="一只恐龙头部和躯干的骨架，它有一个巨大的头，长着锋利的牙齿。"
     width="400"
     height="341"
     title="A T-Rex on display in the Manchester University Museum" >


<video controls>
  <source src="rabbit320.mp4" type="video/mp4">
  <source src="rabbit320.webm" type="video/webm">
  <p>你的浏览器不支持 HTML5 视频。可点击<a href="rabbit320.mp4">此链接</a>观看</p>
</video>


<iframe src="https://developer.mozilla.org/zh-CN/docs/Glossary"
        width="100%" height="500" frameborder="0"
        allowfullscreen sandbox>
  <p> <a href="https://developer.mozilla.org/zh-CN/docs/Glossary">
    Fallback link for browsers that don't support iframes
  </a> </p>
</iframe>


<picture>
  <source type="image/svg+xml" srcset="pyramid.svg" />
  <source type="image/webp" srcset="pyramid.webp" />
  <img src="pyramid.png" alt="regular pyramid built from four equilateral triangles" />
</picture>

<table>
  <tr>
    <td>&nbsp;</td>
    <td>Knocky</td>
    <td>Flor</td>
    <td>Ella</td>
    <td>Juan</td>
  </tr>
  <tr>
    <td>Breed</td>
    <td>Jack Russell</td>
    <td>Poodle</td>
    <td>Streetdog</td>
    <td>Cocker Spaniel</td>
  </tr>
  <tr>
    <td>Age</td>
    <td>16</td>
    <td>9</td>
    <td>10</td>
    <td>5</td>
  </tr>
  <tr>
    <td>Owner</td>
    <td>Mother-in-law</td>
    <td>Me</td>
    <td>Me</td>
    <td>Sister-in-law</td>
  </tr>
  <tr>
    <td>Eating Habits</td>
    <td>Eats everyone's leftovers</td>
    <td>Nibbles at food</td>
    <td>Hearty eater</td>
    <td>Will eat till he explodes</td>
  </tr>
</table>


```

## HTML文档的组成部分

- 页眉：<header>  是简介形式的内容。如果它是 <body> 的子元素，那么就是网站的全局页眉。如果它是 <article> 或<section> 的子元素，那么它是这些部分特有的页眉
- 导航栏：<nav> 包含页面主导航功能。其中不应包含二级链接等内容
- 主内容：<main> <main> 存放每个页面独有的内容。每个页面上只能用一次 <main>，且直接位于 <body> 中。最好不要把它嵌套进其他元素
- 侧边栏：<aside> 包含一些间接信息（术语条目、作者简介、相关链接，等等）
- 页脚：<footer>


## CSS

CSS能定义网页中特定元素样式的一组规则

使用CSS的方式

### 改变元素的默认行为

```css
li {
  list-style-type: none;
}
```

### 使用类名

```css
<ul>
  <li>项目一</li>
  <li class="special">项目二</li>
  <li>项目 <em>三</em></li>
</ul>

.special {
  color: orange;
  font-weight: bold;
}

#对类名是special，标签类型是li和span使用样式
li.special,
span.special {
  color: orange;
  font-weight: bold;
}


```

### 根据元素在文档中的位置确定样式

```css
/*选择<li>内部的任何<em>元素*/
li em {
  color: rebeccapurple;
}

```

### 根据状态确定样式

```css
a:link {
  color: pink;
}

a:visited {
  color: green;
}

```


### 同时使用选择器和选择器

```css
/* selects any <span> that is inside a <p>, which is inside an <article>  */
article p span { ... }

/* selects any <p> that comes directly after a <ul>, which comes directly after an <h1>  */
h1 + ul + p { ... }


body h1 + p .special {
  color: yellow;
  background-color: black;
  padding: 5px;
}


```

### 冲突规则

- 层叠：后面的样式会覆盖前面的
- 优先级：类被认为更具体，优先级高于元素选择器

## 主流JavaScript框架

- Ember：Ember 于 2011 年 12 月发布，最初作为 SproutCore 项目的延续而开始。比其新式的替代品（例如 React 和 Vue），作为老框架，它的用户人数要少得多。但因其稳定性、社区支持以及编程原则都非常良好，它仍然享有很高的知名度
- Angular：一种基于组件的框架，使用声明式的 HTML 模板。在应用构建时，框架的编译器将 HTML 模板转换为优化好的 JavaScript 指令，这一过程对开发者是透明的。Angular 使用 TypeScript
- Vue
- React：2013年Facebook发布




## .js .jsx .ts .tsx

- .js：JavaScript文件
- .jsx：JavaScript文件并使用JSX语法
- .ts：TypeScript文件
- .tsx：TypeScript文件并使用JSX语法

JSX 就是Javascript和XML结合的一种格式。
React发明了JSX，利用HTML语法来创建虚拟DOM。
当遇到<，JSX就当HTML解析，遇到就当JavaScript解析。JSX 只是为 React.createElement(component, props, …children) 方法提供的语法糖。



## JavaScript



### 数据类型

#### null, undefined 和布尔值

1995年 JavaScript 诞生时，最初像 Java 一样，只设置了null表示"无"。根据 C 语言的传统，null可以自动转为0。

```JavaScript
Number(null) // 0
5 + null // 5

```

上面代码中，null转为数字时，自动变成0。

但是，JavaScript 的设计者 Brendan Eich，觉得这样做还不够。首先，第一版的 JavaScript 里面，null就像在 Java 里一样，被当成一个对象，Brendan Eich 觉得表示“无”的值最好不是对象。其次，那时的 JavaScript 不包括错误处理机制，Brendan Eich 觉得，如果null自动转为0，很不容易发现错误。

因此，他又设计了一个undefined。区别是这样的：null是一个表示“空”的对象，转为数值时为0；undefined是一个表示"此处无定义"的原始值，转为数值时为NaN。

经典使用场景：

```JavaScript
// 变量声明了，但没有赋值
var i;
i // undefined

// 调用函数时，应该提供的参数没有提供，该参数等于 undefined
function f(x) {
  return x;
}
f() // undefined

// 对象没有赋值的属性
var  o = new Object();
o.p // undefined

// 函数没有返回值时，默认返回 undefined
function f() {}
f() // undefined

```


布尔值代表“真”和“假”两个状态。“真”用关键字true表示，“假”用关键字false表示。布尔值只有这两个值。

如果 JavaScript 预期某个位置应该是布尔值，会将该位置上现有的值自动转为布尔值。转换规则是除了下面六个值被转为false，其他值都视为true。

- undefined
- null
- false
- 0
- NaN
- ""或''（空字符串）


#### 数值

JavaScript 内部，所有数字都是以64位浮点数形式储存，即使整数也是如此。所以，1与1.0是相同的，是同一个数。


```JavaScript




//JavaScript 内部会自动将八进制、十六进制、二进制转为十进制
0xff // 255
0o377 // 255
0b11 // 3

//NaN是 JavaScript 的特殊值，表示“非数字”（Not a Number），主要出现在将字符串解析成数字出错的场合。
5 - 'x' // NaN
//上面代码运行时，会自动将字符串x转为数值，但是由于x不是数值，所以最后得到结果为NaN，表示它是“非数字”（NaN）。


//Infinity表示“无穷”，用来表示两种场景。一种是一个正的数值太大，或一个负的数值太小，无法表示；另一种是非0数值除以0，得到Infinity。
// 场景一
Math.pow(2, 1024)
// Infinity

// 场景二
0 / 0 // NaN
1 / 0 // Infinity


```

#### 字符串和数组

```JavaScript

// 可以用双引号或单引号，项目中保持统一风格
'aa'
"bb"

var s = 'hello';
s[0] // "h"
s[1] // "e"
s[4] // "o"

// 直接对字符串使用方括号运算符
'hello'[1] // "e"

s.length // 5

```



#### 对象


对象就是一组“键值对”（key-value）的集合，是一种无序的复合数据集合。

```JavaScript
var obj = {
  foo: 'Hello',
  bar: 'World'
};

//获取属性的两种方式
obj.foo 
obj['foo'] 

//键名不用加引号，如果键名是数值，会被自动转为字符串。

//如果键名不符合标识名的条件（比如第一个字符为数字，或者含有空格或运算符），且也不是数字，则必须加上引号，否则会报错。
// 报错
var obj = {
  1p: 'Hello World'
};

// 不报错
var obj = {
  '1p': 'Hello World',
  'h w': 'Hello World',
  'p+q': 'Hello World'
};


//对象的每一个键名又称为“属性”（property），它的“键值”可以是任何数据类型。如果一个属性的值为函数，通常把这个属性称为“方法”，它可以像函数那样调用。

var obj = {
  p: function (x) {
    return 2 * x;
  }
};

obj.p(1) // 2


var obj = {
  key1: 1,
  key2: 2
};

Object.keys(obj);
// ['key1', 'key2']


var obj = { p: 1 };
'p' in obj // true
'toString' in obj // true

```


#### 函数

##### 函数的声明

```JavaScript

//1、function命令声明的代码区块，就是一个函数。function命令后面是函数名，函数名后面是一对圆括号，里面是传入函数的参数。函数体放在大括号里面。
function print(s) {
  console.log(s);
}

//2、匿名函数赋值给变量。这时，这个匿名函数又称函数表达式（Function Expression），因为赋值语句的等号右侧只能放表达式。

var print = function(s) {
  console.log(s);
};

//采用函数表达式声明函数时，function命令后面不带有函数名。如果加上函数名，该函数名只在函数体内部有效，在函数体外部无效。
var print = function x(){
  console.log(typeof x);
};

x
// ReferenceError: x is not defined

print()
// function


//3、不好用的声明方式
var add = new Function(
  'x',
  'y',
  'return x + y'
);

// 等同于
function add(x, y) {
  return x + y;
}



//JavaScript 语言将函数看作一种值，与其它值（数值、字符串、布尔值等等）地位相同。凡是可以使用值的地方，就能使用函数。比如，可以把函数赋值给变量和对象的属性，也可以当作参数传入其他函数，或者作为函数的结果返回。函数只是一个可以执行的值，此外并无特殊之处。

function add(x, y) {
  return x + y;
}

// 将函数赋值给一个变量
var operator = add;

// 将函数作为参数和返回值
function a(op){
  return op;
}
a(add)(1, 1)
// 2


//JavaScript 引擎将函数名视同变量名，所以采用function命令声明函数时，整个函数会像变量声明一样，被提升到代码头部。所以，下面的代码不会报错。
f();
function f() {}


//函数的属性
function f1() {}
f1.name // "f1"

//返回参数个数
function f(a, b) {}
f.length // 2

```



#### 数组


本质上，数组属于一种特殊的对象。typeof运算符会返回数组的类型是object。

```JavaScript
var arr = ['a', 'b', 'c'];


//任何类型的数据，都可以放入数组。
var arr = [
  {a: 1},
  [1, 2, 3],
  function() {return true;}
];

arr[0] // Object {a: 1}
arr[1] // [1, 2, 3]
arr[2] // function (){return true;}



```




### 标准库




## export

```JavaScript
// 导出“（声明的）名字”
export <let/const/var> x ...;
export function x() ...
export class x ...
export {x, y, z, ...};


// 导出“（重命名的）名字”
export { x as y, ...};
export { x as default, ... };


// 导出“（其它模块的）名字”
export ... from ...;


// 导出“值”
export default <expression
```