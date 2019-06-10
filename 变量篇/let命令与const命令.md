# let命令与const命令

<!-- TOC -->

- [let命令与const命令](#let命令与const命令)
    - [let命令](#let命令)
        - [1. 基本用法](#1-基本用法)
        - [2. 不存在遍历提升与暂时性死区](#2-不存在遍历提升与暂时性死区)
        - [3. 不允许重复声明](#3-不允许重复声明)
    - [const命令](#const命令)
        - [1. 基本使用法](#1-基本使用法)
        - [2. 与let一样的用法](#2-与let一样的用法)
        - [3. 本质](#3-本质)

<!-- /TOC -->

## let命令

### 1. 基本用法

ES6 新增了let命令，用来声明变量。它的用法类似于var，但是所声明的变量，只在let命令所在的代码块内有效。

```let
{
  let a = 10;
  var b = 1;
}

a // ReferenceError: a is not defined.
b // 1
```

for循环的计数器，就很合适使用let命令。

```let
for (let i = 0; i < 10; i++) {
  // ...
}

console.log(i);
// ReferenceError: i is not defined
```

下面的代码如果使用var，最后输出的是10。

```let
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 10
```

如果使用let，声明的变量仅在块级作用域内有效，最后输出的是 6。

```let
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6
```

另外，for循环还有一个特别之处，就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域。

```let
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i);
}
// abc
// abc
// abc
```

### 2. 不存在遍历提升与暂时性死区

var命令会发生“变量提升”现象，即变量可以在声明之前使用，值为undefined。

```let
// var 的情况
console.log(foo); // 输出undefined
var foo = 2;

// let 的情况
console.log(bar); // 报错ReferenceError
let bar = 2;
```

只要块级作用域内存在let命令，它所声明的变量就“绑定”（binding）这个区域，不再受外部的影响。

```let
var tmp = 123;

if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
}
```

总之，在代码块内，使用let命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）。

```let
if (true) {
  // TDZ开始
  tmp = 'abc'; // ReferenceError
  console.log(tmp); // ReferenceError

  let tmp; // TDZ结束
  console.log(tmp); // undefined

  tmp = 123;
  console.log(tmp); // 123
}
```

有些“死区”比较隐蔽，不太容易发现。

```let
function bar(x = y, y = 2) {
  return [x, y];
}
```

另外，下面的代码也会报错，与var的行为不同。

```let
// 不报错
var x = x;

// 报错
let x = x;
// ReferenceError: x is not defined
```

### 3. 不允许重复声明

let不允许在相同作用域内，重复声明同一个变量。

```let
// 报错
function func() {
  let a = 10;
  var a = 1;
}

// 报错
function func() {
  let a = 10;
  let a = 1;
}
```

因此，不能在函数内部重新声明参数。

```let
function func(arg) {
  let arg;
}
func() // 报错

function func(arg) {
  {
    let arg;
  }
}
func() // 不报错
```

## const命令

### 1. 基本使用法

const声明一个只读的常量。一旦声明，常量的值就不能改变。

```const
const PI = 3.1415;
PI // 3.1415

PI = 3;
// TypeError: Assignment to constant variable.
```

const声明的变量不得改变值，这意味着，const一旦声明变量，就必须立即初始化，不能留到以后赋值。

```const
const foo;
// SyntaxError: Missing initializer in const declaration
```

### 2. 与let一样的用法

const的作用域与let命令相同：只在声明所在的块级作用域内有效。

```const
if (true) {
  const MAX = 5;
}

MAX // Uncaught ReferenceError: MAX is not defined
```

const命令声明的常量也是不提升，同样存在暂时性死区，只能在声明的位置后面使用。

```const
if (true) {
  console.log(MAX); // ReferenceError
  const MAX = 5;
}
```

const声明的常量，也与let一样不可重复声明。

```const
var message = "Hello!";
let age = 25;

// 以下两行都会报错
const message = "Goodbye!";
const age = 30;
```

### 3. 本质

const实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址所保存的数据不得改动。对于简单类型的数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量。但对于复合类型的数据（主要是对象和数组），变量指向的内存地址，保存的只是一个指向实际数据的指针，const只能保证这个指针是固定的（即总是指向另一个固定的地址），至于它指向的数据结构是不是可变的，就完全不能控制了。

```const
const foo = {};

// 为 foo 添加一个属性，可以成功
foo.prop = 123;
foo.prop // 123

// 将 foo 指向另一个对象，就会报错
foo = {}; // TypeError: "foo" is read-only
```

下面是另一个例子。

```const
const a = [];
a.push('Hello'); // 可执行
a.length = 0;    // 可执行
a = ['Dave'];    // 报错
```

如果真的想将对象冻结，应该使用Object.freeze方法。

```const
const foo = Object.freeze({});

// 常规模式时，下面一行不起作用；
// 严格模式时，该行会报错
foo.prop = 123;
```

除了将对象本身冻结，对象的属性也应该冻结。下面是一个将对象彻底冻结的函数。

```const
var constantize = (obj) => {
  Object.freeze(obj);
  Object.keys(obj).forEach( (key, i) => {
    if ( typeof obj[key] === 'object' ) {
      constantize( obj[key] );
    }
  });
};
```