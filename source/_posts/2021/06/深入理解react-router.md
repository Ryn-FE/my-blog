---
title: 深入理解react router
date: 2021-06-05 11:17:23
tags: react
categories: router
---

## 浏览器导航相关

URI（统一资源标识符）的通用语法： `URI = scheme:[//authority]path[?query][#fragment]`

其中authority可由以下三部分组成：`authority = [userinfo@]host[:port]`

fragment为片段标识符，通常标记为已获取资源的子资源，可选

URL为URI的一种，统一资源定位器

未保留字符不需要进行百分号编码`（a~z, A~Z, 0~9, - _ . ~）`共66个，保留字符需要进行对应的编码，因为其有特殊的含义在URL中。`（!*'();:@&=+$,/?#[]）`共18个

encodeURI对66个未保留字符，18个保留字符，除去`[]`，不对这82个字符编码，对于非ASCII字符，其将转换成UTF-8编码字节序，然后放置%进行编码，也就是会将中文等字符进行编码

encodeURIComponent将转义除字母、数字、( ) . ! ~ * ' - _ 以外的所有字符。encodeURI适合对一个完整的URI进行编码，而encodeURIComponent则被用作对URI中的一个组件或者一个片段进行。

浏览器记录没有直接的api可以获取，可以通过window.history.length来获取当前记录栈的长度信息。由浏览器统一管理，不属于哪一个具体的页面。

```JavaScript
interface History{
    readonly length: number;
    scrollRestoration: 'auto' | 'manual';
    readonly state: any;
    back(): void; // 栈指针后退一位
    forward(): void; // 跳转到当前栈指针所指前一个记录的方法，等同于history.go(1)，是否刷新取决于栈记录是如何得到的
    go(delta?: number): void; // -1表示后退到上一个页面，1表示前进一个页面，0表示刷新当前页面，与location.reload方式行为一致，此方法会刷新页面，会触发popstate事件，pushState永远产生新的栈顶并指向它
    pushState(data: any, title: string, url?: string | null): void; // 无刷新增加历史栈记录，改变浏览器的url，有中文也会URF-8编码，即便浏览器显示的还是中文，但是已经是编码过的。第一个参数是传入的状态，设置了第一个参数之后，可以通过history.state读取。firfox中state大小限制640k，并且跳转的url有同源策略。执行一次会增加一个历史栈，history.length会发生对应的变化，即使url参数不传。data对象采用了结构化拷贝算法，对象中不能设置函数
    replaceState(data: any, title: string, url?: string | null): void; // 类似pushState，但是是修改当前的历史记录，不处于栈顶的情况下修改的也是当前的，且不会将指针指向栈顶，history.length不会发生变化。url同pushstate都支持绝对和相对路径

    // 当前路径为/one/two/three
    // window.history.pushState(null, null, './four')
    // 当前路径为/one/two/four
    // window.history.pushState(null, null, './')
    // 当前路径为/one/two/

    // 当前路径为/one/two/
    // window.history.pushState(null, null, 'four')
    // 当前路径为/one/two/four

}
```

调用history.pushState改变search的值时，hash的值会被清理

base元素存在的情况下，进行添加和修改浏览器记录`<base href='/base/bar'>`，pushState以/开头的绝对路径跳转的时候，base是被忽略的，如果是相对路径，则会使用base作为基准

window.location.href与window.location行为一致，与pushState不同的是其可以刷新对应的页面并重新加载URL指定的内容

window.location.hash改变URL的hash值，改变hash同样会产生新的历史栈记录，如果设置的hash与当前的hash值相同，则不会产生任何事件和历史记录，如果改变hash不进行入栈的操作，通过location.repalce来实现

window.location.replace替换栈记录，设置绝对路径时，会刷新页面

当移动栈指针的时候会触发popstate事件，通过window.addEventListener进行监听，对于回调函数的event事件，event.state是重点关注的，其值为移动后栈中记录的state对象。pushState以及repalceState不会触发popstate事件。当然你也可以使用history.state来获取state对象。location.href设置hash的时候，一样的hash也会出发popstate，但是栈没变，location.hash则不会。

hashchange用来监听浏览器的hash值变化，pushState不触发hashchange事件

可以调用window.dispatchEvent(new PopStateEvent('popstate'))来主动出发popstate事件

## history库详解

history是一个单独的库，提供了createBrowserHistory，createHashHistory，creatMemoryHistory。其中个历史对象都具备

- 监听外界地址变化的能力
- 获取当前地址
- 增加和修改历史栈
- 在栈中移动当前历史栈指针
- 阻止跳转，定义跳转提示
- 获取当前历史栈长度以及最后一次导航行为
- 转换地址对象

```JavaScript
interface History<HistoryLocationState = LocationState> {
    length: number;
    action: Action; //最后一次导航的导航行为
    location: Location<HistoryLocationState>; // 当前历史地址
    push(path: Path, state?: HistoryLocationState): void; // 添加历史栈记录
    push(location: LocationDescriptorObject<HistoryLocationState>): void; //重载方法
    replace(path: Path, state?: HistoryLocationState): void; // 修改历史栈记录
    replace(location: LocationDescriptorObject<HistoryLocationState>): void; // 重载方法
    go(n: number): void;
    goBack(): void;
    goForward(): void;
    block(prompt?: boolean | string | TransitionPromptHook): UnregisterCallback; // 阻止导航行为
    listen(listener: LocationListener): UnregisterCallback; // 监听地址变化，传入一个回调函数，参数1为location，参数2为 action（不同的跳转类型），页面初始化不会触发。location前后值一致的情况下也会触发
    createHref(location: LocationDescriptorObject<HistoryLocationState>): Href; // 地址对象转换
}

interface BrowserHistoryBuildOptions {
    basename?: string;
    forceRefresh?: boolean; // 跳转是否刷新页面，默认不刷新
    getUserConfirmation?: typeof getUserConfirmation;
    keyLength?: number; // 历史栈中栈记录的key字符串的长度，默认6
    // ...通用配置
}

// browser history.push 底层使用history.pushState，无刷新，但是push会触发history的listen监听的回调
interface History<any> {
    push(path: string, state?: any): void;
    push(location: {
        pathname?: string;
        search?: string;
        state?: any;
        hash?: string;
    }): void;
}

// hashHistory 在创建hashHistory时，除了一些公共的配置，还可以设置hash类型，不可设置keyLength，forceRefresh配置
type HashType = 'hashbang' | 'noslash' | 'slash'; // 默认slash：其#号后面都会跟上/，noslash则#号后面都没有/，hashbang为#号后会跟上!和/，页面信息抓取用hashbang更好，页面初始化的过程中，encodedPath和hashPath不同，所以会进行一次初始化，因而每次会在页面后追加/#/
```



## 之后必备

- 结构化克隆算法
- loadhash




