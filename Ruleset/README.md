## 资源

[官方网站](https://nssurge.com/) / [手册](http://manual.nssurge.com/) / [常见问题](https://nssurge.zendesk.com/) / [社区](https://community.nssurge.com/)

## 说明

文件夹内规则适用于 **Surge** 、**Quantumult X** (需打开资源解析器) 和 **Loon** 等代理软件，下面是以 **Surge** 为例。

### 关于 IPv6

默认并不开启 IPv6，如需要可在「更多设置 >」里打开「IPv6 支持」，或在文本配置中修改 `ipv6 = false` 为 `ipv6 = true`。

### 关于 DNS

如果所使用的网络没有 DNS 劫持问题，则配置为使用系统 DNS 并追加公共 DNS，如果所使用的网络存在 DNS 劫持问题，则配置为仅使用公共 DNS；

> 如部分运营商存在劫持海外正常网站至反诈页面的（据目前反馈它们没有抢答公共 DNS，所以）可以在「DNS 设置」中选择「使用自定义 DNS 服务器」或文本配置中将 `dns-server =` 中的 `system` 移除。

不建议使用海外 DNS（包括 119.28.28.28），如 `1.1.1.1` 解析哔哩哔哩返回的是香港的 CDN，这时候再指定规则直连没什么意义；

> 如果非要用海外 DNS 或不知道所用 DNS 是否为海外，遇到应走直连的国内域名走了代理或不知道为什么走了代理，可自行排查：在「工具」的「DNS 结果」中查找走了代理的相关域名，就可以看到是哪个 DNS 返回了什么结果（IP）。

> 注意，有代理规则的域名并不会有本地 DNS 结果（远程解析），如果出现已有代理规则的域名出现在「DNS 结果」说明有不带「no-resolve」参数的 IP 规则位于 DOMAIN 规则之前。

非必要不建议使用 DoH；

> 必要指的是如中国移动这种抢答公共 DNS 的运营商

### 关于 Apple 分流

默认 Apple 分流为直连（除了被动或主动屏蔽的那些，所以 Apple.list 放在 Global.list 之后），所以如果想完全走代理可以将 `RULE-SET,Apple.list` 修改为代理策略。

但若想 Apple 只要国内全走直连只要国外全走代理可将 `RULE-SET,Apple.list` 注释或移除，**前提是 Apple 相关域名仅使用国内 DNS**。

### 服务器不支持 UDP 转发时的策略行为

当服务器不支持 UDP 转发时会阻止相关 UDP 请求，如果你没有支持 UDP 转发的服务器，可以注释掉 `udp-policy-not-supported-behaviour = reject` 或将其值修改为 `direct` 以开启允许相关 UDP 请求直连。

### 关于规则及策略组

使用排序要求如下：

1. [必须] Unbreak.list - 用于修正后续规则行为
2. [可选] Advertising.list - 广告（macOS 不建议开启）
3. [可选] Privacy.list - 隐私（行为分析、隐私追踪）
4. [可选] Hijacking.list - 劫持（运营商、恶意网址）
5. [必须] Streaming.list - 流媒体服务
6. [可选] StreamingSE.list - （大陆面向国际的）流媒体服务
7. [必须] Global.list - 国际网络分流
8. [可选] Apple.list - 苹果服务分流
9. [可选] Adobe.list - 跳过 Adobe 验证
10. [必须] China.list - 国内网络分流

#### Unbreak

主要用于修正后续规则中 REJECT 及 PROXY 策略的一些不正确情况，如常见的暴力去广告造成的某些推送服务无法使用、使用 Google 的一些可直连服务。

#### Streaming

主要为国际流媒体服务，`Music` 和 `Video` 目录里的独立分流文件全是从 `Streaming.list` 中剥离出来的。

Streaming 策略组最初的设想使用方式是独立出来给有观看流媒体服务的用户一个方便的使用方式。如默认节点使用的是美国但有日本和英国的流媒体服务，在观看 AbemaTV 时在 Streaming 策略组选择日本节点，在观看 BBC 时选择英国节点。

而对于一些流媒体爱好者而言，他们会进一步按照区域建立策略组如 HK、JP 等，然后将适用于相应区域的流媒体服务独立分流文件的策略，指定为相应的策略组，如此就不用在 Streaming 策略组来回切换。

一些需要注意的事项：

1. 一些流媒体服务需要「原生（或称「本土」）」的 IP，此类称呼存在争议只需要明白如开通的是港区 Netflix 不是随便找个香港服务器就能够使用的。
2. tv 位于 `Apple` 目录内；
3. `Video` 下的 `bilibili.list` 和 `iQiyi.list` 与国内版不是一个 App；
4. 当不需要「Streaming 策略组」时，`Streaming.list` 策略应该调整为 PROXY 而不是移除；
5. 如需细化流媒体如「Youtube.list」等，需要加在「Streaming.list」之前。

#### StreamingSE

一般为中国大陆的流媒体面向港澳台或海外的版本，不同于上述的独立版本，下列流媒体如果直接代理会影响中国大陆版内容的播放。所以以策略组的形式，在需要观看面向港澳台或海外的版本时切换代理，日常可选直连。
目前支持：

- 哔哩哔哩（僅限港澳台地區）；
- 愛奇藝海外站；
- 芒果TV国际；

#### 应用类

如需细分应用类如「Telegram.list、Google.list、PayPal.list」需要加在「Global.list」之前。

### 从 Surge 迁移到 Quantumult X 可能遇上的问题

注意，Quantumult X 并没有官方文档，所以以下内容可能有误。

规则类型优先级，在 Quantumult X 中并不是完全按顺序匹配，各类型的优先级应该为： host > host-suffix > host-keyword > user-agnet > geoip & ip-cidr。

> 举个例子：如某远程分流文件中含有 `host-suffix, instagram.com, proxy` 时，添加 `host-keyword, instagram, direct` 并不能改变 instagram.com 的策略；

顺序上来说 Quantumult X 将本地规则放在规则列表的顶部，所以当本地规则存在 `geoip,cn,direct` 时，远程分流文件中对于 CN 地区的 IP 规则并不会生效，解决办法是将相关规则也写在本地或将 `geoip,cn,direct` 放到远程分流文件中。

另外，在 Surge 中如 `DOMAIN,1.1.1.1` 这样的规则可以用于匹配目标主机为 IP 地址 1.1.1.1 的连接，但并不适用于 Quantumult X。

### 关于去广告

#### 为什么有些应用还是无法使用？

1. 开启「HTTPS 解密(MitM)」功能
2. 安装证书
3. 到系统「设置 > 通用 > 关于本机」中底部的「证书信任设置」中信任所安装的证书！

#### 为什么某应用还是有广告

**1.缓存**

有些应用会将广告缓存，如果在使用规则前应用就已经缓存了广告，所以你需要：
- 应用内设置里清除缓存。
- 但有的应用并不会清除广告的缓存，所以需要将应用删除重装。

**2.功能**

广告阻止不仅于使用 [Rule] 规则，有的广告需要 [URL Rewrite] 和 [MITM]，这就意味着：
- Surge 虽然支持 [URL Rewrite] 和 [MITM]，但无法处理 TUN 的广告。
- Quantumult 虽然支持 [URL Rewrite] 和 [MITM]，但需要在「更多 > 附加功能」中开启「激进阻止」以更全面的支持（X 版本默认带此功能）否则同 Surge 效果一样，但 Quantumult 对于 IP 不会进行 MitM，所以有的广告不能如 Surge 一般去除。

**3.规则不是万能的**

不是所有广告都能简单的依靠规则阻止。
