---
title: lepd项目解析之一 -- cjson框架
tags: [c,json]
category: 编程
date: 2018-05-08 20:57:28
---

## lepd项目

lepd项目是宋宝华之前推荐的一个开源项目，可用于监控linux系统的性能指标，在浏览器中可视化呈现。项目地址 [lepd 项目 github地址](https://github.com/21cnbao/lepd) ，文章将从各个方面详细介绍涉及的技术知识点，以方便参考使用。

lepd使用的信息传递格式是json，便于在浏览器中使用。

## json简介

 介绍json格式和用法

http://www.json.org/

http://www.json.org/json-zh.html

> json基于两种数据结构：1、key/value对， 2、list有序列表（数组）



JSON具有以下这些形式：

### json对象: object

对象是一个无序的“‘名称/值’对”集合。一个对象以“{”（左括号）开始，“}”（右括号）结束。每个“名称”后跟一个“:”（冒号）；“‘名称/值’ 对”之间使用“,”（逗号）分隔。

> *object*
>
> `{}`
>
> `{` *members* `}`

![img](http://www.json.org/object.gif)

### json数组: array

数组是值（value）的有序集合。一个数组以“[”（左中括号）开始，“]”（右中括号）结束。值之间使用“,”（逗号）分隔。

> *array*
>
> `[]` 
>
> `[` *elements* `]`  
>
> *elements*
>
> *value* 
> *value* `,` *elements*

![img](http://www.json.org/array.gif)

### json值: value

值（*value*）可以是双引号括起来的字符串（*string*）、数值(number)、`true`、`false`、 `null`、对象（object）或者数组（array）。这些结构可以嵌套。

> *value*
>
> *string*
> *number*
> *object*
> *array*
> `true`
> `false`
> `null`

![img](http://www.json.org/value.gif)

### json字符串：string

字符串（*string*）是由双引号包围的任意数量Unicode字符的集合，使用反斜线转义。一个字符（character）即一个单独的字符串（character string）。

字符串（*string*）与C或者Java的字符串非常相似。

> *string*
>
> `""`
>
> `"` *chars* `"`
>
> *chars*
>
> *char*
> *char chars*
>
> *char*
>
> *any-Unicode-character-*
>     *except-***"***-or-***\***-or-*
>     *control-character*
> `\"`
> `\\`
> `\/`
> `\b`
> `\f`
> `\n`
> `\r`
> `\t`
> `\u` *four-hex-digits*

![img](http://www.json.org/string.gif)

### json数值： number

数值（*number*）也与C或者Java的数值非常相似。除去未曾使用的八进制与十六进制格式。除去一些编码细节。

> *number*
>
> *int*
> *int frac*
> *int exp*
> *int frac exp*
>
> *int*
>
> *digit*
> *digit1-9 digits* 
> `-` *digit*
> `-` *digit1-9 digits*
>
> *frac*
>
> **.** *digits*
>
> *exp*
>
> *e* *digits*
>
> *digits*
>
> *digit*
> *digit* *digits*
>
> *e*
>
> **e**
> **e+**
> **e-**
> **E**
> **E+**
> **E-**

![img](http://www.json.org/number.gif)

空白可以加入到任何符号之间。 以下描述了完整的语言。

```flow
st=>start: -
e=>end: -
op=>condition: My Operation
op2=>condition: My Operation2
op3=>condition: My Operation3

st(right)->op(yes)->op2(yes,right)->op3(yes,right)->e




```



## cJSON简介

cJSON其实就是对Json格式的字符串进行构建和解析的一个C语言函数库。总共包括2个部分：

