---
title: 以开源项目的标准写代码
toc: true
date: 2022-12-30 22:27:05
tags: [Coding]
categories: Coding
---

来新司办公已经两个月了，除了业务之外，对职业本身的素质要求也有了新的认知。

<!--more-->

### 关于注释

我一直有一个认知“好的代码本身就是注释”，拥有好的命名的方式，清晰的调用关系的代码，不需要额外的注释，阅读者也可以轻松看懂。类似于下面的注释，不仅又臭又长，还会破坏代码的干净整洁。

```java
/**
* Increment a value by delta and return the new value.
*
* @param  delta   the amount the value should be incremented by
* @return     the post-incremented value
*/
```

然而每次 PR 给领导，Leader 给我的回复都是“put more comments. node/http 会自动将传入的 headers 全部改写成小写，你输入 `Prject-Secret`，用的时候就得 `prject-secret`, 没有注释确实会让其他人对这中间发生的什么难以理解。

```js
const projectSecret = req.headers["project-secret"];
```

New

```js
/**
 * node/http auto converted HTTP header fields to lowercase
 * so when user put Project-Secret in request Headers
 * in Express, you will get Project-Secret
 * put Content-Type will be converted to content-type
 * details: https://www.rfc-editor.org/rfc/rfc2616#section-4.2
 */
const projectSecret = req.headers["project-secret"];
```

所以我的总结:

- 注释要优秀，注释排版必须要好看
- 把思路（复杂的、关键的）写清楚，比啥 Author、Date 重要多了

### 不放过任何一段代码，不满足于能 Work

对自己的要求过于低，满足于“又不是不能用”。有时从 stackoverflow 贴下来的代码，运行可以用之后就不会去深究为什么这一段要这样写？还有直接复制开源项目的部分函数也是经常发生，根本不去想为什么对方需要这样用，我是不是需要完全 copy 别人的实现？Leader 提醒我 “有时候编码太快，不是一件好事情，有时候是要慢下来，思考每一段代码，不能一股脑的复制粘贴”。

对此，要提高自己的要求，对于自己写的每一段代码，都要明白它到底做了什么，把自己的代码都想象成随时要开源出去，怎么写才能对得起自己的署名。
