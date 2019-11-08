title: 正则表达式笔记 
date: 2015-09-13 00:42:53
categories: 笔记
tags: [计算机, 笔记]
---
> 正则表达式，又称正规表示法、常规表示法（英语：Regular Expression，在代码中常简写为regex、regexp或RE），计算机科学的一个概念。正则表达式使用单个字符串来描述、匹配一系列符合某个句法规则的字符串。在很多文本编辑器里，正则表达式通常被用来检索、替换那些符合某个模式的文本。

<!--more-->
#### 1
__\d__可以匹配一个数字，__\w__可以匹配一个字母或数字, __.__可以匹配任意字符。

eg：

- '00\d'可以匹配'007'，但无法匹配'00A'；
- '\w\w\d'可以匹配'py3'；
- 'py.'可以匹配'pyc'、'pyo'、'py!'等等。




#### 练习

请尝试写一个验证Email地址的正则表达式。版本一应该可以验证出类似的Email：

````
someone@gmail.com
bill.gates@microsoft.com
````
版本二可以验证并提取出带名字的Email地址：

````
<Tom Paris> tom@voyager.org
````

`````
# -*- coding: utf-8 -*-
#author=raighneweng
import re
#import re

s1 = 'someone@gmail.com'
s2 = 'bill.gates@microsoft.com'
s3 = '<Tom Paris> tom@voyager.org'

re_email=re.compile(r'^(<[\w\s]+>)?\s*[\w\.]+@[\w]+\.[\w]+$')

print(re_email.match(s1).group(0))

print(re_email.match(s2).group(0))

print(re_email.match(s3).group(0))
`````
