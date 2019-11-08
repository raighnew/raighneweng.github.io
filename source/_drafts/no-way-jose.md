---
title: 拒绝JOSE!我们都应该拒绝Javascript的错误签名加密标准
toc: true
tags: [javascript, 翻译]
categories: Web
date: 2018-01-05 11:30:40
thumbnail:
banner:
---

如果你决定使用 javascript 对象签名和加密, 不管是想要使用 JSON Web Tokens, JSON Web Encryption (JWE), or JSON Web Signatures (JWS)，你应该质疑这个决定, 你可能做了一个错误的决定.
<!--more-->
本文所有的观点都是基于当前的标准（RFC 7519, RFC 7515, and RFC 7516），不排除以后新的标准会解决这些问题。

如果你对基础的密码学和概念不是很熟悉，为了简洁起见，点击[这里](https://paragonie.com/blog/2015/08/you-wouldnt-base64-a-password-cryptography-decoded)。

#### 为什么你需应该使用JavaScript对象签名和加密

JOSE家族的标准有一些问题，这些并不是针对任何特定实现才有的缺陷，而是事实上许多库都是基于这些缺陷的标准。

#### JSON Web Tokens 经常被滥用
很多开发者尝试用JWT来避免服务器端存储session。

A lot of developers try to use JWT to avoid server-side storage for sessions. This is almost always a terrible mistake and invites developers to come up with clever explanations and workarounds instead of careful engineering.

The two linked posts explain succinctly why this is a bad move, so I won't delve further into the systems architecture issues. There are more pressing issues at stake: The standard itself is bad and leads to insecurity.

