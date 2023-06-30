---
title: 利用 Clash Rules 绕行 BT 下载的多种简单方法
tags: clash bt openwrt 软路由
type: article
pageview: true
key: clash-bypass-bt

article_header:
  type: cover
  image:
    src: assets\images\blogs\2023-6-29-clash-bypass-bt\clash.png
aside:
  toc: true
---

在使用 Clash 进行透明代理的过程中，无论是为了节省流量使用，还是因为服务商禁止 BT，需要绕行 BT 流量的情况十分常见。本文试着总结了我自己在 PC 和软路由上使用 Clash 时绕行 BT 流量的多种相对简单的方法，它们基本上都是利用 Clash 的规则设置实现的。

<!--more-->

----------

## Process-Based Rule / 进程规则

> [Clash Process-Based Rule 文档](https://lancellc.gitbook.io/clash/clash-config-file/rules/process-based-rule)。

于我这样因学艺不精而没办法完全掌控系统内所有流量的走向之人而言最为简单且有效的方法。对于 Win PC，在任务管理器内找到 BT 下载软件的进程名，通过 PROCESS-NAME 规则导向 DIRECT 即可。对于 Linux PC 及软路由等，进程名依然能匹配上，有时也会考虑 PROCESS-PATH 规则匹配**绝对路径**。

## IP-Based Rule / IP 规则

> [Clash IP-Based Rule 文档](https://lancellc.gitbook.io/clash/clash-config-file/rules/ip-based-rule)

### 针对 Dockers

在 PC 上直接运行的进程用进程规则就可以解决了，但软路由上的 BT 软件如果是在 Docker 内运行，则进程规则写起来会变得麻烦。[V2EX 上的相关讨论](https://www.v2ex.com/t/757230)中[@zer](https://www.v2ex.com/member/zer)提供了一种针对 Docker 的简单思路：

> 我是在 docker 里跑的 transmission 和 qbittorrent，这 2 个容器利用 macvlan 使用独立的 ip 。在路由器上的 openclash 设置个访问控制，让 ip 不走代理就行。前提是 clash 使用 redir 模式。

### BT 软件固定 IP

当然绕行 IP 的思路理论上也可以直接应用于 BT 下载软件，只需要令软件流量的 IP 固定。比如：

 - 软件绑定 IP：以 qBittorrent 为例，在 `设置 > 高级 > qBittorrent相关 > 绑定到的可选IP地址` 处可以设置。指定接口等方法也可以实现。
 - 独立的下载机器：这样甚至可以 MAC 绕行。

总之无论何种方法，只要 IP 固定，Clash 处就很方便设置了。不过我在尝试 IP 思路时总是未能完全成功，会偶尔有 BT 下载流量通过内核。我没有花时间去排查流量来源。

## Port-Based Rule / 端口规则

> [Clash Port-Based Rule 文档](https://lancellc.gitbook.io/clash/clash-config-file/rules/miscellaneous-rule)

### 端口黑名单

我曾看到有 Github Issues 讨论中提及 qBittorrent 可以设置端口范围，但我并没有找到相关设置。如果软件可以实现，那么在此基础上，与 IP 规则类似，也可以通过绕过特定端口范围实现目的。

### 端口白名单

这个思路下还有另一个方法，即令 Clash **仅代理常用端口流量**。OpenClash 已集成这个功能，现版本（v0.45.121-beta）设置路径为 `插件设置 > 流量控制 > 仅允许常用端口流量`。内核或其他 UI 可以手写规则。不过对于真正有软路由代理需求（而不是直接终端运行代理）的人来说这样设置多半是与需求相违了。

## Match Rule / 缺省规则

> [Clash Match Rule 文档](https://lancellc.gitbook.io/clash/clash-config-file/rules/match-rule)

相比直接设置端口白名单，令 Clash **仅代理命中规则流量**要容易接受一些。取决于规则的命中率以及实际应用需求，这个方法甚至可能是最一劳永逸（而不仅限于解决 BT 下载问题）的方法了。OpenClash 同样提供一键覆写设置，位于 `覆写设置 > 规则设置 > 仅代理命中规则流量`。非 OpenClash 用户可以手动追加/修改 Match 规则。

如果非智能设备的代理需求非常简单，路由器可以靠规则白名单轻松满足，同时智能设备自己在终端代理的话，这个方法也算十分行之有效。比如，软路由主要为客厅游戏机提供代理，而需要代理的游戏的相关规则已经写好。

## *其他不依靠 Clash 规则的方法

除了利用 Clash 的规则设置绕过代理，当然还有很多在 Clash 外处理流量的方法可以解决这个问题，不过我太菜了，不会 X( 。很多思路也与“简单解决”这个要求相悖。因为目前我自己的使用环境已经解决了这个问题，所以暂时不打算继续折腾下去了。

----------

## 题外

🎉🎊👏👏OK，这篇就是我博客的第一篇正式博文了！🎉🎊👏👏虽然形式有点随意，内容也相当无趣且不严谨，完全不是自己敢拍胸脯说擅长的领域，但是主要记给自己看的博客就该写些能力边界上的东西才有意义嘛。这次就写到这里罢。希望我将来还有足够的动力写出下一篇博文（Flag）。