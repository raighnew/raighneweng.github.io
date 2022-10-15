---
title: 大金ModBus模块homebridge
toc: true
tags: [智能家居]
categories: HomeLab
date: 2021-02-06 13:48:10
thumbnail:
banner:
---

家里工作两年的网关出了问题，尝试恢复的时候发现还是有点难度，缺乏可用的文档。所以整理一下备份出来。

<!--more-->

当初看到马老师用的中弘网关一直很心水，然而中弘网关太贵了，一直观望没有购买，偶尔有一次在论坛上看到了 yonsm 的方案，价格合适就立刻入手了。

硬件连接的部分参考两位大牛的文章:
[ModBus 空调组件及中央空调接入 Home Assistant 简述](https://yonsm.github.io/modbus/)
[ModBus 空调组件及中央空调接入 Home Assistant 细节](https://agassiyzh.github.io/2018/10/29/HA-climate-modbus/)

我这里重点贴一下有人模块的配置。

![](/images/usr01.jpeg)

![](/images/usr03.jpeg)
![](/images/usr02.jpeg)

踩雷点:
1: 一定要看附送的使用手册，U1 是输出端口，美的、格力使用 U2 作为输入端口，大金和其他使用 U3
2: 信号线 AB 要区分
3: 建议先使用 USB 转 RS232 模块先用 modbuspoll 通过 COM 端口调试
4: 有人转接模块工作模式应该是透传
5: modbuspoll 测试有人模块时时应该使用 modbus RTU over TCP 测试
