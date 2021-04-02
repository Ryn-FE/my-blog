---
title: TypeScript手册
date: 2021-03-06 16:57:18
tags: TypeScript
categories: 教程 笔记 TypeScript
---

## 基础

在调用之前就应该确保你的所调用的方法是存在的

tsc TypeScript编译器，也就是类型检查器，使用npm i -g typescript进行安装，然后编写你对应ts文件，使用tsc file.ts进行编译你的ts文件。如果ts文件写的不规范，tsc则会在命令行上出现错误。使用tsc --noEmitOnError file.ts对错误进行防御

tsc --target es2015 file.ts可以将输出的代码转成为相应的指定版本的js。noImplicitAny，隐藏类型不推断any，strictNullChecks标志操作null和undefined更加明确

## 日常类型

string、number、boolean是我们平常用的比较多的内置类型，大写的String、Number、Boolean是一些特殊的内置类型将，虽然是合法的，但是大写的很少出现在自己的代码中，始终对类型使用小写的字符

数组，可以用`number[]`，`string[]`，还有一种`Array<number>`，表示相同的意思，这个是用泛型进行的对应数组说明。`T<U>`

TS还有一个特殊的类型any，可以在不希望特定值引起任何类型检查错误的时候使用它，上下文类型，ts会自动判断当前所用的一些类型是从哪里来

```TypeScript
let arr = ['text1', 'text2']
arr.forEach(item => {
    console.log(item.toUpperCase())
})
// 如果调用的不是string的方法则会报错

function fn(arg: {x: number; y: number; z?: number}){ // 分割可以用,或者;
    console.log(arg.x + arg.y)
}
```

可选属性在属性名后面加一个？，如上面的z属性就是可选属性，对于可选属性来说，你必须先对其进行一个非undefined的判断然后再使用它

### 联合类型

可以直接调用他们共有的属性，否则就得使用条件进行对应的判断

```TypeScript
const strAndNum: string | number = 1;
```

### 类型别名

这两个之间可以自由选择，但是还是有一些对应的区别

```TypeScript
type Point = {
    x: number;
    y: number;
}
interface Point = {
    x: number;
    y: number;
}
```

| type        | interface |
| :---------  | :-------- |
| 扩展类型通过& | 扩展类型通过extends |
| 向现有见面增加字段  |  创建后无法改变类型  |
| 不参与声明合并 ｜ 可以声明合并 ｜
| 可以命名基元 ｜ 只能用于声明对象的形状，不能重命名基元 ｜

### 类型断言

如果你要使用document.getElementById()，ts仅知道其返回对应的HTMLElement，但是你知道你页面会一直存在对应的element，此时你就可以使用类型断言

```TypeScript
// 可以使用类型断言来指定具体的类型
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;
// 其中在tsx中你可以这样写
const myCanvas = <HTMLCanvasElement>document.getElementById("main_canvas");

// 当然你还可以这样
const x = "hello" as unknown as number;
const a = (expr as any) as T;
```

### 文字类型

```TypeScript
// 可以使用文字类型来接受一组你想要的值
function printText(s: string, align: 'left' | 'right' | 'center'){}
printText('hi', 'center')
```

数值类型类似，当然其可以与其它类型相互结合使用

### 字面推论

```TypeScript
const req = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method);
// 如果说handleRequest第二个参数只接受GET或者POST那么上述代码是有问题的，因为req自动推断出method为字符串，但是handleRequest第二个参数只能是GET字符或者POST字符，则这样推断有问题，那么如何解决这个问题呢？
// 1
const req = { url: "https://example.com", method: "GET" as "GET" };
// 2
handleRequest(req.url, req.method as "GET");
// 3 可以使用as const，将req变成文字类型
const req = { url: "https://example.com", method: "GET" } as const;
```

### null和undefined

表示缺少或者未初始化的值，strictNullChecks建议开启，用来确保在调用某些方法之前测试这些值的存在

```TypeScript
function doSomething(x: string | undefined) {
  if (x === undefined) {
    // do nothing
  } else {
    console.log("Hello, " + x.toUpperCase());
  }
}
```

### 非空断言运算符（!）

用于从类型中删除null和undefined不进行任何显式检查，实际上是一个类型断言，表明该值不是null或者undefined

```TypeScript
function liveDangerously(x?: number | undefined) {
  // No error
  console.log(x!.toFixed());
}
```

### 枚举

之后进行一个对应的填充

### 不常见的基元

bigint symbol

## Narrowing
