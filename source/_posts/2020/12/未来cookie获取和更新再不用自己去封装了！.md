---
title: 未来cookie获取和更新再不用自己去封装了！
date: 2020-12-31 14:03:11
tags: cookie
categories: 浏览器
---

## 谷歌浏览器87版本更新

![谷歌浏览器版本](https://gitee.com/RenYaNan/wx-photo/raw/master/2020-12-31/1609407871306-Chrome-version.png)

谷歌浏览器的最新版本在2020年末最后一天是87版本，就在这个版本中，有一个针对于cookie的API，Cookie Store，这个API代替了我们常用的document.cookie的方法，87版本开始就可以在谷歌浏览器中使用，给大家做一个简单的介绍。如果你要使用的话记得判断对应的浏览器版本，必须大于等于87版本

判断浏览器版本有很多方法，总的来说就是通过navigator用正则去做对应的匹配，在这里就不多说了

## cookie方法简述

目前来看，大家在cookie方面都是统一的document.cookie，在此基础上做一些额外的操作，但是通常我们不知道的是，当你设置完成一个cookie信息之后，是否设置成功。你还需要再去获取一下你设置的那个cookie的值，看是否已经成功设置，麻烦不说，还影响我们的执行效率。而且对于正则匹配我们拿出来的所有的cookie信息这种方式很尴尬。cookie是否发生了变更，也没有对应的监听，这些都是要解决的

自然，有杠精说我存储到localStorage不就完事么？场景不一样，用的内容也不一样，如果有需要，可以复习一下localStorage和cookie的区别

![cookie](https://gitee.com/RenYaNan/wx-photo/raw/master/2020-12-31/1609407891910-cookie.jpeg)

## 获取cookie

刚才我们也提到了，目前都是统一通过document.cookie的方式获取的cookie，然后通过对应的匹配形式。为什么我们只想要name这个对应的cookie信息，但总是要先把所有的拿出来，并且还很开心的觉得自己封装了一个好的获取cookie的方法，这总是过于复杂，而且效率很低

附上一个我们平时封装的获取cookie信息的方式

```javascript
function getCookie (name) {
    let result
    return (result = new RegExp('(?:^|; )' + encodeURIComponent(name) + '=([^;]*)').exec(document.cookie)) ? decodeURIComponent(result[1]) : null
}
```

那么如果我们用cookieStore的方式该怎么做呢？

新的方式的话，需要使用cookieStore并调用其get方法，它返回的是一个promise，所以，当你设置失败的时候，它会告诉你失败并返回失败原因，具体的调用实例如下

```javascript
try {
    const cookieValue = await cookieStore.get('cookieName')
    if(cookieValue){
        console.log('cookieName: ', cookieValue)
    } else {
        console.log('cookieValue is null')
    }
} catch(e) {
    console.error('cookieStore is error: ' + e)
}
```

可以发现，这种方式不需要再去遍历一整遍cookie的字段了，当然也不会影响你的同步操作，毕竟这个获取方式是异步的

## 设置cookie

之前我们设置cookie的方式

```javascript
document.cookie = 'cookieName=cookieValue; domain: xxx.com'
```

当然你可以把这种形式封装一下，会有一种更好的写法，但是写法是舒服了，本质还是一样的

就像之前说的，在设置完成之后，如果我们想知道是否设置成功，那么就需要用getCookie获取一遍，如果能够获取到，那说明没问题

那么在cookieStore中，我们如何设置呢？

```javascript
try {
    await cookieStore.set({
        name: 'cookieName',
        value: true,
        domain: 'xxx.com',
        expires: Date.now()
    })
} catch (e) {
    console.error('falied:' + e)
}
```

通过上述方式就可以设置一个对应的cookie，当然，只要你不经过catch，它就是设置成功

## 删除cookie

之前的方式，我们通常是通过setCookie的形式将对应的cookie的值设置成空，然后将expires的值设置成过期的时间，这样依靠浏览器就会自动删除其对应的内容，这里不再列举

那么新的方式如何做呢？

```javascript
try {
    await cookieStore.delete('cookieName')
} catch (e) {
    console.error('falied:' + e)
}
```

只要没走到catch，那表明删除已经发生并且是正常的

## 监控cookie

当然，新的API肯定有新的方法，那就是你可以监控cookie，当cookie内容发生变化的时候会执行此操作

```javascript
cookieStore.addEventListener('change', event => {
    console.log(`${event.changed.length} changed cookies`)
    for (const cookie in event.changed) {
        console.log(`${cookie.name} changed`)
    }
    console.log(`${event.deleted.length} deleted cookies`)
    for (const cookie in event.deleted) {
        console.log(`${cookie.name} deleted`)
    }
})
```

## cookie有哪些属性

平时的cookie获取可以使用document.cookie进行获取，是由键值对组成，一个;号和一个空格隔开。但是这个方法只能获取非HttpOnly类型的cookie

设置cookie属性的时候，也是由一个分号和一个空格隔开

- expires：设置cookie有效期，必须是GMT格式时间，可通过new Date().toGMTString()来获得，如果没有设置时间，那么是会话级cookie，浏览器关闭，cookie消失。http/1.1中由Max age代替，有效期为创建时间加Max age
- domain 域名，子域名
- path 路径，子路径 跨域请求满足但是cookie也不会自动被添加
- size cookie的大小
- secure cookie只有在确保安全的请求才会发送，当请求是https或者其他安全协议的时候，包含这个选项的cookie才会被发送，默认这个选项为空，所以任何请求都会携带。如果想要设置这个值，必须确保网页是https协议的才能够去设置
- HttpOnly 设置cookie是否能够通过js访问，有这个选项的表示客户端无法通过js代码去读取，修改，以及删除的操作。而且这个选项是不能通过js修改的，必须服务端才能够设置这个相关的值。并且document.cookie也是获取不到有这个属性的cookie内容的

## 总结

针对以上，似乎chrome团队已经解决了我们的对于cookie的问题，并且这些内容在我们认为理所应当应该存在的东西，但是还是能够希望大家能够很好的去使用，有机会体验一次如果你的项目中有需求的话。

兼容性别忘记，还是要考虑一下的，但是相信，随着科技的不断发展，这些东西肯定终将不会是问题！
