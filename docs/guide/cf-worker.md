---
home: false
heroText: Cloudflare Worker
tagline: 部署到 Cloudflare Worker
footer: Licensed under GPL-2.0 license
---

# 部署到 Cloudflare Worker

**本方式不推荐**，我不保证一直工作。除非你有特殊需求，并且你知道自己在做什么，否则请采用 [Cloudflare Pages](./cf-pages.md)

## Cloudflare Worker 代码

把下面代码 copy 到 Cloudflare Worker 在线编辑器内。

https://raw.githubusercontent.com/zizifn/edgetunnel/main/libs/cf-worker-vless/cf-worker-vless-dev.js

## 客户端配置
采用Vless+Websocket+TLS方式，服务器地址与主机名都是那个.workers.dev域名的，需要去掉 "https://" 前缀。路径不用填，留空。端口443, UUID为设定的即可。

## 修改 UUID
推荐设置环境变量，名为全大写UUID，值为生成的UUID。

::: warning
这是 VLESS over WS 配置。只有 UUID 是强制，其他参数无所谓。WS PATH 随便填写。
:::

![cf-worker-code](../public/cf-worker-code.png)
