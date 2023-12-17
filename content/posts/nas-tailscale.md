+++
title = '使用Tailscale Funnel 实现内网穿透'
date  = '2023-06-25T10:09:17+08:00'
tags  = ['NAS']
+++

前段时间网络上一篇 tailscale 的技术[博文](https://tailscale.com/blog/how-tailscale-works/#encrypted-tcp-relays-derp)相当火热，讲述了 tailscale 如何在各种情况下实现打洞，干货满满，全是技术点。本来 QNAP 自带的 myqnapcloud 也够用了，但是被 tailscale 的博文吸引，准备在 NAS 上试用一下。

<!--more-->
## 前期准备
1. NAS 可以 ssh
2. NAS 上安装好 tailscale，QNAP 可以使用第三方Repo [myqnap.org](https://www.myqnap.org) 安装管理。


## 配置 Funnel
###  Choose One Tailnet
这个domain就是你之后可以在公网访问的domian

### Enable MagicDNS
### Configure Access Controls

在 `Access Controls` 里的 `nodeAttrs` 添加如下配置
```json
	"nodeAttrs": [
		{
			// Funnel policy, which lets tailnet members control Funnel
			// for their own devices.
			// Learn more at https://tailscale.com/kb/1223/tailscale-funnel/
			"target": ["autogroup:members"],
			"attr":   ["funnel"],
		},
	],
```

### HTTPS Certificates


### SSH NAS
SSH进入NAS，输入如下命令

```bash
# 暴露您的服务
tailscale serve https / http://127.0.0.1:xxx

# 开启 443 端口
tailscale funnel 443 on

# 查看 funnel 状态
tailscale funnel status
```

## 安全问题
1. 关闭 NAS Admin 账号
2. 关闭 tailscale SSH

