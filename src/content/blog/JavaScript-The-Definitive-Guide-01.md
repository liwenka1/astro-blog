---
author: 李文凯
pubDatetime: 2023-10-15T21:38:24+08:00
title: 《JavaScript 权威指南》犀牛书阅读（一）
postSlug: JavaScript-The-Definitive-Guide-01
featured: true
draft: false
tags:
  - JavaScript
  - 阅读
ogImage: ""
description: "词法结构 1.JavaScript程序的文本 2.注释 3.字面量 4.标识符和保留字 5.Unicode 6.可选的分号 7.小结"
canonicalURL: ""
---

# 词法结构

## JavaScript 程序的文本

---

JavaScript 区分大小写。

```
比如， online Online OnLine ONLINE 是四个不同的变量名。
```

## 注释

---

JavaScript 分为单行注释和多行注释。

```js
// 这是单行注释
/* 这也是注释 */
/**
 * 这是多行注释
 * 每行开头的 * 其实只是为了美观，可以不加
 */
```

## 字面量

---

这些都是简单的字面量。

```js
12 // 数字12
1.2 // 数字1.2
("hello world") // 字符串
("Hi") // 字符串
true // 布尔值
false // 布尔值
null // 无对象
```

## 标识符和保留字

---

### 标识符

标识符就是一个名字。在 JavaScript 中，通常用于在 JavaSCript 代码中命名常量、变量、属性、函数和类。以下都是合法的标识符：

```js
i
my_name
v13
_dummy
$str
```

### 保留字

以下列出的单词是 JavaScript 语言的一部分。不要使用这些单词作为标识符，但 form、set、target 除外，因为使用他们很安全，而且也很常见。

```js
as    const    export   get        null   target void
async continue extends  if         of     this   while
await debugger false    import     return throw  with
break default  finally  in         set    true   yield
case  delete   for      instanceof static try
catch do       from     let        super  typeof
class else     function new        switch var
```

JavaScript 也保留和限制对某些关键字的使用，这些关键字当前并未被语言所用，但将来的某个版本又可能会用到：

```js
enum implements interface package private protected public
```

由于历史原因，某些情况下也不允许 arguments 和 eval 作为标识符，因此最好不要使用。

## Unicode

---

JavaScript 程序是使用 Unicode 字符集编写的，因此在字符串和注释中可以使用任意 Unicode 字符。

```js
const Π = 3.14
const α = "阿尔法"
```

### Unicode 转义序列

Unicode 转义序列以 \u 开头。后跟 4 为十六制数字（包括大写或小写的字母 A ~ F）或包括在一对花括号内的 1~6 位十六制数字。Unicode 转义序列可以出现在 JavaScript 字符串字面量、正则表达式字面量和标识符中（不能出现在关键字中）。

```js
let café = 1 // 使用 Unicode 字符 'é' 定义一个变量
caf\u{e9} // =>1 使用转义序列访问这个变量
cafe\u{301} // =>1 使用转义序列的另一种形式
```

JavaScript 的早期版本只支持 4 位数字转义序列。带花括号的版本是 ES6 新增的，目的是为了更好地支持大于 16 位的 Unicode 码点，比如表情符号：

```js
console.log("\u{1f600}") // 打印出一个 😀
```

Unicode 转义序列也可以出现在注释中，但因为注释会被忽略，所以注释中的转义序列会被当做 ASCII 字符处理，不会被解释为 Unicode。

### Unicode 归一化

Unicode 允许用多种编码方式表示同一个字符。

```js
const café = 1 // 这个变量名为 "caf\u{e9}"
const café = 2 // 这个变量名为 "cafe\u{301}"
```

Unicode 标准为所有字符定义了首选编码并规定了归一化例程，用于把文本转换为适合比较的规范形式。JavaScript 假定自己解释的源代码已经归一化，它自己不会执行任何归一化。如果想在 JavaScript 程序中使用 Unicode 字符，应该保证使用自己的编辑器或者其他工具对自己的源代码执行 Unicode 归一化，以防其中包含看起来一样但实际不同的标识符。

## 可选的分号

JavaScript 使用分号（;）分隔语句。但在 JavaScript 中，如果两条语句分别写着两行，通常可以省略它们之间的分号。

```js
// 可以省略
a = 3
b = 4

// 分号是必须的
a = 3; b = 3;
```

但 JavaScript 并非任何时候都把换行符当作分号，而只是在不隐式添加分号就无法解析代码的情况下才这么做。

```js
let a
a
=
3
console.log(a)

// JavaScript将以上代码解释为：
let a; a = 3; console.log(a);
```

之所以把第一个换行符当作分号，是因为如果没有分号，JavaScript 就无法解析代码 let a a。第二个 a 本身是一个独立的语句，但 JavaScript 并没有把第二个换行符当作分号，因为它还可以继续解析更长的语句 a = 3;。

这些语句终止规则会导致某些意外情形。

```js
let y = x + f
(a + b).toString()

// 由于第二行的圆括号可以被解释为第一行 f 的调用，所以 JavaScript 将这两行代码解释为：
let y = x + f(a + b).toString();
```

而这很有可能不是代码作者的真实意图。为了保证代码被解释成两条语句，这里必须要明确添加一个分号。

通常，如果语句以 (、[、/、+、- 开头，就有可能被解释为之前语句的一部分。实践中，以 /、+、- 开头的语句比较少，但以 (、[ 开头的语句则并不鲜见。

```js
let x = 0 // 这里省略分号
;[x.x+1.x+2].forEach(console.log) // 添加防御 ; 保证这条语句独立
```

JavaScript 在不能把第二行解析为第一行的连续部分时，对换行符的解释有第三种例外情况。

第一种情况涉及 return、throw、yield、break、continue 语句，这些语句经常独立存在，但有时候后面会跟一个标识符或表达式。

```js
// 如果你这么写
return
true

// JavaScript 假设你的意图是：
return; true;

// 但你的意图可能是：
return true;
```

这意味着，一定不能在 return、break、continue 等关键字和它们的表达式之间加入换行符。如果加入了换行符，那代码出错后的调试会异常麻烦，因为错误并不明显。

第二种情况涉及 ++ 和 -- 操作符。这些操作符既可以放在表达式前面，也可以放在表达式后面。如果把这两个操作符作为后置操作符，那它们必须与自己的操作的表达式位于同一行。

第三种情况涉及到简介的 "箭头" 语法定义的函数：箭头 => 必须跟参数列表在同一行。

## 小结

---

本章讲解了在最低层面上应该如何编写 JavaScript 程序。
