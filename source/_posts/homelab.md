---
title: 家庭网络的升级
date: 2022-01-03 20:51:53
tags: [智能家居, ubnt, unifi]
categories: HomeLab
cover: /images/unifi_dashboard.png
---

经常逛论坛，很早就心水了 Ubnt AC+AP 全家桶的便捷，然而第一套房子装修的时候并没有预留网线，现在又旅居蓉城，想的是装修第二套房子的时候再购买 Ubnt 全家桶。

<!--more-->

现在出租屋的网络架构是 R2S _(前同事离职前给我的福利，200 的价格收了一套金属壳的 R2S，还送了一张 C10 的卡)_ +K2P，R2S 当主路由经常会因为 passwall 挂了整个网络都跟着挂，诟病很久了，但一直拖着没有升级，压倒骆驼的最后一根稻草是上周是领导 1:1 视频会议，我的网络挂了十分钟才恢复。不得已把网络升级计划提上日程。

可能是疫情的关系，这几年 UBNT 的设备升级很慢，选购 AP 的时候看论坛里反馈 U6-Pro 好像有 BUG，所以先选购 U6-Lite，等新房子装修好 U6-Lite 放主卧，再购买 U6-Pro 放客厅用。网关和交换机选购的时候一度想购买 UDM-PRO，但是 UDM-PRO 没有 POE+电源，购买之后还需要购买 POE+的交换机。(_看欧洲官网，有一款[UMD-PRO-SE](https://eu.store.ui.com/products/dream-machine-pro-se-ea?variant=40354517844150)在早期测试，带 POE+电源，真是真正的 Dream Machine_)

设备列表:
[U6-Lite](https://eu.store.ui.com/products/unifi-ap-6-lite)
[USG-3P](https://eu.store.ui.com/collections/unifi/products/unifi-security-gateway?_pos=1&_sid=e3df07597&_ss=r)
[USW-Lite-8-PoE](https://eu.store.ui.com/collections/unifi-network-routing-switching/products/unifi-switch-lite-8-poe)

我的需求：

- 一条给领导用，不需要翻墙。网关 10.10.10.1
- 一条给我和家里电视用，需要用 R2S 做单臂路由，需要翻墙。网关 10.10.20.1
- 一条给智能设备用，要求与其他网络隔离。网关 10.10.30.1

回来跟着张大妈上的教程设置了一下，发现有一些坑，总结如下：

- USG 默认为 192.168.1.1 网段，会跟电信猫抢网关，我的建议是一劳永逸的将电信猫设置为 192.168.0.1
- USG，USW，WIFI 在 10.10.0.1 网段，AC 放在同一网段也比较好。
- AC 里的 WAN 默认网关也是 192.168.1.1，需要你修改后再采用 USG，不然 USG 的设置会被修改，然后陷入死循环。

最后贴几张我的配置图。
![](/images/network_setting.png)
![](/images/wifi_setting.png)

使用了快一周了，UBNT 真好。
