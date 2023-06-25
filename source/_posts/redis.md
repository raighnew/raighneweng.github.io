---
title: redis的封装
toc: true
tags: [Coding, IT, JavaScript]
categories: Coding
date: 2018-02-22 23:14:40
cover: /images/redis.png
banner:
---

最近整理项目，发现用到的 redis 的地方很多，对于数据的存储一直没有规范，对于不同的 db 使用 redis.createClient 新建一个数据库链接，然后再分别写方法，其实有一些存查删的方法是共用的，所以诞生了封装 redis，让不用的 db 共用一套方法。思路如下：

<!--more-->

# 实现

- 我要实现：new 一个对象，传入目标数据库，然后所有新建的对象共用一套方法。

用面向对象编程的思想使用构造函数初始化一个对象，传入的目标数据库，在内部新建一个 redis 链接。

```js
function Constructor(db) {
  this.redisConfig = {
    db, // 使用第几个数据库
    host: config.redis.host,
    port: config.redis.port,
    maxage: 24 * 60 * 60, // 缓存时间
  };

  if (config.redis.password !== "") {
    this.redisConfig.password = config.redis.password;
  }

  this.Redis = redis.createClient(this.redisConfig);

  this.Redis.on("error", (err) => {
    console.log("redis error start");
    console.log(err);
    console.log("redis error end");
  });
}
```

使用 prototype 属性，给创建的对象提供共享的属性和方法。

```js
RedisCtrl.prototype = {
  saveCommonData(key, value, expire) {
    const tkey = `${key}`;
    return new Promise((resolve, reject) => {
      this.Redis.set(tkey, JSON.stringify(value), (err, data) => {
        this.Redis.expire(tkey, expire);
        if (err) {
          return reject(err);
        }
        return resolve(data);
      });
    });
  },
};
```

这样 redis 的封装就成功了。

# 问题

编码的过程中遇到了一个问题，写 prototype 内写方法的时候，一开始使用的箭头函数，函数内的 this 指向的并不是我想要的对象本身，所以 this.Redis 根本取不到：

- 箭头函数中的 this 在定义它的时候已经决定了（执行定义它的作用域中的 this），与如何调用以及在哪里调用它无关，包括 (call, apply, bind) 等操作都无法改变它的 this。
- 箭头函数不同于传统 JavaScript 中的函数，箭头函数并没有属于自己的 this ，它的 this 是捕获其所在上下文的 this 值，作为自己的 this 值，并且由于没有属于自己的 this，箭头函数是不会被 new 调用的

# ES6 中的 Class

众所周知，ES6 中提供了类的支持，来用 class 的方法试写一下。

```js
class RedisInstance {
  constructor(db) {
    this.redisConfig = {
      db, // 使用第几个数据库
      host: config.redis.host,
      port: config.redis.port,
      maxage: 24 * 60 * 60, // 缓存时间
    }

    if (config.redis.password !== '') {
      this.redisConfig.password = config.redis.password
    }
    this.Redis = redis.createClient(this.redisConfig)

    this.Redis.on('error', (err) => {
      console.log('redis error start')
      console.log(err)
      console.log('redis error end')
    })
  }

  saveCommonData(key, value, expire) {
    const tkey = `${key}`
    return new Promise((resolve, reject) => {
      this.Redis.set(tkey, JSON.stringify(value), (err, data) => {
        this.Redis.expire(tkey, expire)
        if (err) {
          return reject(err)
        }
        return resolve(data)
      })
    })
  }
```

果然很方便。

写代码的时间还不长，有一些编程思想还是需要多写才能体会出来。