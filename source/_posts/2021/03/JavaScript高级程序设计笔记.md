---
title: JavaScript高级程序设计笔记
date: 2021-03-14 16:51:43
tags: 红宝书
categories: 红宝书
---

## HTML中的JavaScript

### script元素属性

- async：立即开始下载脚本，不能阻止页面其它动作，只适用于外部脚本，异步脚本保证会在页面的load之前执行，但可能会在DOMContentLoaded的之前或之后
- charset：指定代码字符集
- crossorigin：配置相关请求的cors设置，也就是跨域资源共享
- defer：在文档解析和显示完成后再执行脚本，但是会立即开始下载，推迟执行，但是会在DOMContentLoaded事件之前执行，只对外部脚本有效
- integrity：允许对比接收到的资源和指定的加密签名以验证子资源完整性
- src：要执行代码的外部文件
- type：代码块中脚本语言的内容类型，一般为'text/javascript'，如果为'module'，则代码为es6模块，只有这个时候代码中才能够出现import和export关键字

一般来说代码块会被保存在解释器环境中，并且js代码在被解释的过程中，页面的其余内容不会被加载也不会被展示。在行内的js代码中，代码不能出现`</script>`字符串，否则会报错，需要转义字符反斜杠，也就是js代码会阻塞页面的渲染。

如果`script`标签中包含行内代码并且src有值，浏览器会下载并执行脚本文件而忽略行内代码

页面在浏览器解析到body标签的时候才开始渲染

如果想让浏览器的预加载器知道一些动态请求文件的存在，可以在文档头部显示声明它们

```HTML
<link rel='preload' href='preload.js'>
```

`<noscript>`标签中可以写入html，以下两种情况会走`<noscript>`中的逻辑
1. 浏览器不支持脚本
2. 浏览器对脚本的支持被关闭

## 语言基础

1. 区分大小写
2. 标识符，也就是变量名称，函数名称等内容，最好驼峰命名法
3. 注释，多行，单行等
4. 严格模式，use strict
5. 语句，最好加上分号，不然浏览器会自己判断在哪里加上分号，影响性能，代码块由{}包裹
6. 关键字与保留字

### 变量

局部作用于定义全局变量很难维护，定义多个变量可用逗号分割进行定义

- var变量提升，可多次定义同一个变量，声明的范围是函数作用域，全局作用域下会挂在到window上
- let为块级作用域，暂时性死区（在let声明前的执行瞬间才叫暂时性死区），全局作用域下不会在window上
- 混用let和var都会冗余报错
- const在声明的时候必须付值，在for循环中，每次迭代创建一个新的变量是OK的，但是如果赋值那就fail

### 数据类型

简单数据类型：null，undefined，number，string，boolean，Symbol
复杂类型：object

typeof是一个操作符而不是一个函数。

```JavaScript
typeof null // 'object null值表示一个空对象指针'
typeof function // 'function'
typeof {} // 'object'
```

对于未声明或者未初始化的变量，只能对它执行一个操作就是typeof，总是会返回undefined。所以在做对应的声明的同时也要对应的对其做出初始化的操作，这样在使用typeof的时候就知道到底是未声明还是未初始化

undefined是由null派生来的，因此其表面上相等。所以null == undefined为true，任何时候只要是保存对象，而其又没有对应的值，就将其初始值设置为null，这样能够和undefined区分开

可以用Boolean()方法将其余的所有类型转换成boolean类型

```JavaScript
Boolean(NaN) // false
```

#### 数字

- 八进制：第一个数字为0并且后续的数字为0～7，如果后续的数字不为0-7，则会忽略之前的0，es6的八进制的数字前缀为0o
- 十六进制：数值前缀为0x，然后是十六进制的数字，0-9 A-F/a-f
- 当其用作数学操作中的时候，则都会被视为10进制数值
- 科学计数法：1.3e10表示1.3乘以10的10次幂，ECMAScript会将小数点后至少包含6个0的数字转成科学计数法的形式
- 值的范围：最小值Number.MIN_VALUE（5e-324）最大值Number.MAX_VALUE（1.797 693 134 862 315 7e+308）如果某个计算数值超过了MAX_VALUE，那么这个数值会被自动转换成Infinity（无
穷），Infinity不可用于任何计算
- 如何确定某个值是否有限大，使用isFinite()函数，Number.NEGATIVE_INFINITY 和 Number.POSITIVE_INFINITY 也可以获取正、负 Infinity
- NAN表示本来要返回数值的操作失败了，0/0便会返回NAN，NAN不等于包括NAN在内的任何值。NAN == NAN为false，任何涉及NAN的计算都会返回NAN，isNaN()函数可以判断一个参数是不是数字，注意isNaN('10')为true，因为其可以转换成数字
- isNaN可用来测试对象，此时首先会调用valueOf()，看其返回值是否是数字，否则再调用toString()，测试其返回值
- 将非数值转换成数值方式：Number、parseInt、parseFloat，Number可以用于任何数据类型，但是后面两个主要用于将字符串转换成数值
  - Number可以将true以及false转换成数值，null返回0，undefined返回NaN，字符串转成数字，空字符串返回0，否则返回NaN。对象来说先调用valueOf的方法，按照上述规则进行转换。如果是NaN则调用toString的方式进行转换
  - parseInt会从第一个非空格字符开始转换，如果第一个字符不是数字字符，加减号，立即返回NaN，所以空字符串也会是NaN，但是'1234blue'会返回1234，后续不是数字的字符会被忽略，类似22.5会被转换成22，其第二个参数是要解析的字符的进制数
  - parseFloat会忽略字符串开头的0，十六进制数值始终返回0.parseFloat只解析十进制，不能指定底数，字符串表示整数，它会返回整数

#### 字符串

字符字面量

- \n 换行
- \t 制表
- \b 退格
- \r 回车
- \f 换页
- \\ 反斜杠
- \' 单引号
- \" 双引号
- \` 反引号
- \xnn 以十六进制编码nn表示的字符
- \unnnn 以十六进制编码nnnn表示的Unicode字符

转义序列表示一个字符，如果字符包含双字节字符，那么length属性返回的可能是不准确的字符数

转换成字符串可使用toString方法，null和undefined没有toString方法，toString可以穿入一个参数，表示得到数值的哪种进制字符串。如果你不确定一个值是不是







