---
categories:
- 建站技能
date: '2025-11-05 14:40:24+08:00'
description: ''
draft: false
image: ''
slug: check-domain
cover: /archives/check-domain/ic3kgv.png
tags:
- 域名
title: 入手域名怎么确认是否被墙
---

前不久搭了一个小破站，然后在 cf 上入手了一个看起来还行的域名，服务端配置了 https 证书访问，然后对 http 进行转发到 https 了

```shell
server {
    listen 80;
    server_name abc.com www.abc.com;
    return 301 https://www.abc.com$request_uri;
}
```

由于本地 pc 是长期带魔法棒的，所以在 pc 端访问 http 以及 https 的时候，一切看起来都没有任何问题

然后手机端进行访问的时候，就出现了问题，手机端是没有配置魔法，在访问 https 的时候一切正常，但是在访问 http 的时候就嘎了

开始还以为是 nginx 配置的问题，左右想不明白，为什么没有转发到 https 呢

最终发现，是伟大的墙在搞鬼，拦截了域名的 http 请求，但是对于 https 的请求又是正常的，顿时心痛如绞痛，痛失一笔巨款，- -||

如下，是在本地没有魔法请求 http 的时候，得到的响应，表明 `Connection was reset`

![](/archives/check-domain/ic3kgv.png)

在浏览器中进行 http 访问的时候，得到的响应如下，表明 `ERR_CONNECTION_RESET`

![](/archives/check-domain/0u0wkk.png)

登录了几个外网节点进行 curl 测试，都是显示请求正常，在聚名上查询，显示域名已经被墙了，本以为捡漏的域名，结果发现是掉坑里了

![](/archives/check-domain/a2gc6k.png)

如下是 `abc.com` 域名查询的情况，可以参考下，查询地址 `https://www.juming.com/hao/abc_com`

![](/archives/check-domain/p4wgbd.png)

