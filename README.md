# Clash 分流规则，重点分流 AI 服务、字节海外 AI 服务 、 WEB3 应用、教育类 APP、开发者常用下载节点等分流服务

> 本项目基于 ACL4SSR 项目进行修改，用于个人使用

# 重点个性化分流规则介绍

| 分流规则名称 | 说明                                                                                  |
| ------------ | ------------------------------------------------------------------------------------- |
| 🇺🇲 美区AI    | 需美国节点的 AI 服务，OpenAI、Claude、Grok、Perplexity、Google 服务与 Gemini 合并在此 |
| 🌈 CiciAI    | 建议使用新加坡节点，确保字节海外版的 APP 和 AI 服务可正常使用                         |
| 👨‍💻 Developer | 建议使用流量大或免费节点，各种 docker 镜像、模型文件下载专用                          |
| 🔒 Web3      | 建议使用自建的安全节点，避免出现安全风险                                              |

其他分流规则，包括 哔哩哔哩,Khan,海外媒体,Spotify,Netflix, YouTube, DisneyPlus, 游戏平台等常用规则。

## 重要提醒

部分AI网站和APP优先使用了 `HTTP/3 (QUIC)`协议，导致即使分流和TUN 模式下都可能无法使用服务，在确保IP干净也无法使用的情况下，请添加下面的自定义规则（可能会影响部分游戏服务）并启用。

```
AND,((NETWORK,UDP),(DST-PORT,443)),REJECT
```

特别说明：为什么要加第一行 REJECT？
hysteria2，这是一个基于 UDP 的暴力传输协议。

误区：很多人以为用了 UDP 协议的节点，就必须让所有 UDP 流量都通畅。

真相：

Hysteria2 协议本身确实是用 UDP 发包给服务器的（这是底层的车）。

但是，浏览器里的 HTTP/3 (QUIC) 也是 UDP。

当我们加了 - AND,((NETWORK,UDP),(DST-PORT,443)),REJECT 这一行时，我们并没有掐断 Hysteria2 的连接。我们只是在本地告诉浏览器：“别尝试用 QUIC 直连或者乱发 UDP 包了，老老实实把数据包封装成 TCP 给我”。

一旦浏览器转为 TCP，Clash 就会把它接管过来，完美地通过 Hysteria2 协议封装发出去。

# 使用方式

业务分组只显示订阅提供的实际节点，以及原有的 `DIRECT` / `REJECT` 内置策略。已移除地区组、自动选择组、自定义组及所有组间选项；“🚀 节点选择”保留为分流规则使用的业务组，其他分组不再引用它。

更新订阅后，请在各业务分组直接选择节点，例如为“🇺🇲 美区AI”选择美国节点、为“🌈 字节AI”选择新加坡节点。原先默认直连的分组继续优先 `DIRECT`，原先默认代理的分组优先订阅节点。

`Clash/kclash.ini` 和 `Clash/pref.ini` 仍为订阅转换器的 INI 配置，使用 `.*` 匹配转换时读到的订阅节点，不会额外创建原生 `proxy-providers` 配置。

在订阅的 URL 上增加 config=[替换成下面 ini,或自定义]

[https://raw.githubusercontent.com/szkane/ClashRuleSet/main/Clash/kclash.ini](https://raw.githubusercontent.com/szkane/ClashRuleSet/main/Clash/kclash.ini)

订阅地址后面增加：`&config=https%3A%2F%2Fraw.githubusercontent.com%2Fszkane%2FClashRuleSet%2Frefs%2Fheads%2Fmain%2FClash%2Fkclash.ini`
