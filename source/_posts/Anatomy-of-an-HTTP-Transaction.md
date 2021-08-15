title: Anatomy of an HTTP Transaction 翻译
toc: true
date: 2016-09-21 20:38:05
categories: Coding
thumbnail: /images/nodejs-logo.png
banner: /images/nodejs-logo.png
tags: [Nodejs, JavaScript]

---

# 前言

在学习 Nodejs 过程中，发现官方文档里讲 Node.js 对 http 请求的处理过程相当不错，而且一直以来没有看英文文档的习惯，为了强迫自己养成习惯，就想翻译一次官方文档，看看效果如何，相信能学到很多，原文链接[Anatomy of an HTTP Transaction](https://nodejs.org/en/docs/guides/anatomy-of-an-http-transaction/#create-the-server)。

<!--more-->

# 全文开始

本指南的目标在于阐述  Node.js 对 HTTP 请求的处理过程，我们假设你知道 HTTP 请求是如何工作的,不管语言或编程环境，并且对 Node.js 的 EventEmitters 和 Streams 有一定了解，如果你不是很熟悉他们,最好快速了解一下有关 API。

# 创建服务器

任何 Node 服务都需要在某一处通过 createServer 来创建一个 Web 服务对象。

```
var http = require('http');

var server = http.createServer(function(request, response) {
  // magic happens here! 见证奇迹的时刻！
});
```

每一次的 HTTP 请求都会传入一个参数给 createrServer 函数，事实上 createrServer 返回 Server 的对象是一个 EventEmitter ，我们在这里简写一下上面的程序，并在之后添加监听。

```
var server = http.createServer();
server.on('request', function(request, response) {
  // the same kind of magic happens here!同样见证奇迹的时刻！
});
```

当请求来时，Node.js 会调用请求处理函数，同时封装两个常用对象 request 和 response ，我们以后会经常碰到它们。

为了能接接收到 HTTP 请求，还需要调用 server 对象的 listen 方法。大部分情况下，你只需传递给 listen 一个端口号。其他的一些选项，可以参考这里[ API reference](https://nodejs.org/api/http.html)

# Method, URL 和 Headers

处理请求时，你首先想知道的事可能事请求的 method 和 URL，然后才会有相应的动作，Node.js 把常用的对象放进了 request 对象里，直接调用就可以。

```
var method = request.method;
var url = request.url;
```

<strong>Note:</strong>request 对象是一个[IncomingMessage](https://nodejs.org/api/http.html#http_class_http_incomingmessage)实例</center>
这里的 method 永远是一个正常的 HTTP method/verb。 url 是没有服务器，协议或端口的完整 URL，即网址后包括第三斜线后的一切。

Headers 也在 request 对象里：

```
var headers = request.headers;
var userAgent = headers['user-agent'];
```

需要注意的是，无论客户端发送什么，Node.js 都会将所有的头部信息小写化，简化的同时减少的因分歧出错的可能性。如果有重复的头信息，取决于头信息有些会覆盖，或者合并到逗号分开的字符串里。在一些情况下会有问题，没关系，还有 [rawHeaders](https://nodejs.org/api/http.html#http_message_rawheaders) 可以用。

# 请求体 Request Body

当收到 POST 或者 PUT 请求时，请求体就会特别重要，获取请求体又比获取请求头信息需要知道更多点。request 对象实现了[ReadableStream](https://nodejs.org/api/stream.html#stream_class_stream_readable)的接口，所以该 stream 能像其他流一样能被被监听或者管道化，我们能通过监听流的 'data' 和 'end' 事件来获取流内数据。

'data'事件发出的数据块都是[Buffer](https://nodejs.org/api/buffer.html)。如果你知道传过来的数据是字符串，那么最好用一个数组来收集它们，在'end'中，合并并字符化它们。

```
var body = [];
request.on('data', function(chunk) {
  body.push(chunk);
}).on('end', function() {
  body = Buffer.concat(body).toString();
  // at this point, `body` has the entire request body stored in it as a string
});
```

<strong>Note:</strong>:在一些情况下，这看起来有一些乏味。但是幸运的是，有一些能将这些逻辑隐藏的优秀的模块比如[ concat-stream](https://www.npmjs.com/package/concat-stream)[body](https://www.npmjs.com/package/body)在[npm](https://www.npmjs.com/)上。但是还是希望你能好好的细节一下它们在深入之前，这也是你为什么在这里的原因</center>

# 错误信息 A Quick Thing About Errors

既然 request 对象是一个[ReadableStream](https://nodejs.org/api/stream.html#stream_class_stream_readable), 也是一个[EventEmitter](https://nodejs.org/api/events.html#events_class_eventemitter)，当错误发生时，会触发 'error' 事件。

当发生错误时，在 stream 中将会产生一个 'error' 事件，如果你没有监听该事件，错误抛出，可能会终止你的程序。因此，你需要新增一个 'error' 监听给 requset ，在做日记记录的同时，给客户端返回对应的错误码，这个后面会提到。

```
request.on('error', function(err) {
  // This prints the error message and stack trace to `stderr`.console的错误标准输出
  console.error(err.stack);
});
```

还有其他的方式来处理错误，请参考[这里](https://nodejs.org/api/errors.html),但请警惕错误随时会发生，对其有对应的处理总是好的。

# 总结

到目前，我们已经创建了一个 web 服务器，并且的到了请求的 method ， url ， headers 还有请求体内容。现在我们把它们放在一起：

```
var http = require('http');

http.createServer(function(request, response) {
  var headers = request.headers;
  var method = request.method;
  var url = request.url;
  var body = [];
  request.on('error', function(err) {
    console.error(err);
  }).on('data', function(chunk) {
    body.push(chunk);
  }).on('end', function() {
    body = Buffer.concat(body).toString();
    // At this point, we have the headers, method, url and body, and can now
    // do whatever we need to in order to respond to this request.
  });
}).listen(8080); // Activates this server, listening on port 8080.
```

如果我们运行这个例子，我们能够接收请求，但不能回应它。事实上，如果你在浏览器发出请求会超时，没有东西会返回客户端。

至今我们还没有接触 response 对象，它是一个[ ServerResponse ](https://nodejs.org/api/http.html#http_class_http_serverresponse)实例，也是一个[ WritableStream ](https://nodejs.org/api/stream.html#stream_class_stream_writable), 为了将数据传回客户端，有很多种方法，我们慢慢来介绍。

# HTTP 状态码

response 默认的状态码是 200，但是有些情况下，你需要返回不同的状态码。为了实现它，你需要设置 statusCode，例如：

```
response.statusCode = 404; //告诉客户端未找到资源
```

后面我们还会学到简略写法。

# 设置响应头

通过[setHeader](https://nodejs.org/api/http.html#http_response_setheader_name_value)来设置响应头

```
response.setHeader('Content-Type', 'application/json');
response.setHeader('X-Powered-By', 'bacon');
```

值得注意的是，响应头对关键词大小写不敏感，如果我们重复的设置一个响应头，客户端取最后一个。

# 显式发送响应头

上面提到的 statusCode 和 setHeader 属于隐式头部。意思是在发送 boby 数据前，依赖 Node.js 来发送头部数据。

如果你需要，也可以显式的将头部信息写到响应流里，writeHead 为此而生。

```
response.writeHead(200, {
  'Content-Type': 'application/json',
  'X-Powered-By': 'bacon'
});
```

一旦设置完头部，就可以发送响应数据了。

# 发送响应数据

既然 response 对象是个 WritableStream ，就可以使用流方法来向客户端写数据了。

```
response.write('<html>');
response.write('<body>');
response.write('<h1>Hello, World!</h1>');
response.write('</body>');
response.write('</html>');
response.end();
```

可以直接用 end 简写成下面这样：

```
response.end('<html><body><h1>Hello, World!</h1></body></html>');
```

<strong>Note:</strong>响应体在响应头之后，在往 response 里写入数据之前设置好状态码和头信息，才会有意义。</center>

# 另外一个错误处理

与 request 一样，response 也会触发 error 事件。所以，有关 request 错误处理的最佳建议，也同样适用于 response。

# 总结 2

现在我们已经学会 HTTP response 了，那么把代码放到一起，我们让服务器把浏览器的请求组织好所有数据再发送给用户，用 JSON.stringify 格式化
JSON 数据。

```
var http = require('http');

http.createServer(function(request, response) {
  var headers = request.headers;
  var method = request.method;
  var url = request.url;
  var body = [];
  request.on('error', function(err) {
    console.error(err);
  }).on('data', function(chunk) {
    body.push(chunk);
  }).on('end', function() {
    body = Buffer.concat(body).toString();
    // BEGINNING OF NEW STUFF

    response.on('error', function(err) {
      console.error(err);
    });

    response.statusCode = 200;
    response.setHeader('Content-Type', 'application/json');
    // Note: the 2 lines above could be replaced with this next one:
    // response.writeHead(200, {'Content-Type': 'application/json'})

    var responseBody = {
      headers: headers,
      method: method,
      url: url,
      body: body
    };

    response.write(JSON.stringify(responseBody));
    response.end();
    // Note: the 2 lines above could be replaced with this next one:
    // response.end(JSON.stringify(responseBody))

    // END OF NEW STUFF
  });
}).listen(8080);
```

# Echo 服务器

简化上面的代码来创建一个简单的 Echo 服务器，即请求什么数据，就返回什么数据，跟上面代码差不多，我们只需从请求里面获取数据写入响应里。

```
var http = require('http');

http.createServer(function(request, response) {
  var body = [];
  request.on('data', function(chunk) {
    body.push(chunk);
  }).on('end', function() {
    body = Buffer.concat(body).toString();
    response.end(body);
  });
}).listen(8080);
```

这个太简单了，现在我们增加下面条件：

- 请求的 method 是 GET.
- URL 是 /echo.
  另外则给出 404.

```
var http = require('http');

http.createServer(function(request, response) {
  if (request.method === 'GET' && request.url === '/echo') {
    var body = [];
    request.on('data', function(chunk) {
      body.push(chunk);
    }).on('end', function() {
      body = Buffer.concat(body).toString();
      response.end(body);
    })
  } else {
    response.statusCode = 404;
    response.end();
  }
}).listen(8080);
```

<strong>Note:</strong>检查 URL 实际上就是一种路由功能，其它路由选择形式简单的有 swtich 语句，复杂的有 Express 框架。如果你在寻找纯路由功能，可以试试[Router](https://www.npmjs.com/package/router)。</center>

现在，我们进一步简化。记住， request 对象是一个 ReadableStream ，response 对象是一个 WritableStream 。这意味着可以使用管道（pipe）直接将数据从一端传到另一端。那正是我们需要的：

```
var http = require('http');

http.createServer(function(request, response) {
  if (request.method === 'GET' && request.url === '/echo') {
    request.pipe(response);
  } else {
    response.statusCode = 404;
    response.end();
  }
}).listen(8080);
```

到这里，事情并没有完呢，正如本篇指南里经常提的，如果出现了一场怎么办。
为了处理请求体的错误，我们打印出错误，并且发送 400 状态码给失效的请求。通常的错误处理机制参考[这里](https://nodejs.org/api/errors.html)

```
var http = require('http');

http.createServer(function(request, response) {
  request.on('error', function(err) {
    console.error(err);
    response.statusCode = 400;
    response.end();
  });
  response.on('error', function(err) {
    console.error(err);
  });
  if (request.method === 'GET' && request.url === '/echo') {
    request.pipe(response);
  } else {
    response.statusCode = 404;
    response.end();
  }
}).listen(8080);
```

最好，我们已经学习了 Node.js 如何处理 HTTP 请求的基础知识，总结如下

- 实例化一个 HTTP 服务器，包含一个请求处理函数，另外别忘了监听一个端口。
- 从 request 获取 headers , url , method , body 等信息。
- 在 request 对象里根据 url 或者其它信息路由。
- 通过 response 发送响应头、状态码和数据。
- 管道化数据从 request 到 response 。
- 对 request 和 response 设置错误处理机制。

有了这些基础，我们已经能为很多典型应用案例构建  Node.js HTTP 服务器了，还有一些提供的 APIs ，参考如下: [EventEmitter](https://nodejs.org/api/events.html) [Streams](https://nodejs.org/api/stream.html) [HTTP](https://nodejs.org/api/http.html)。
