# 路由与自定义出站（面板里配）

在面板里配的分流、屏蔽、出站看这页；写在节点文件里的本机路由、出口组看[分流与出口（节点文件）](../features/routing.md)。

在面板里配置屏蔽、指定 DNS、把部分流量转发到其它服务器。三个面板的设置入口和格式各不相同，本页按面板分别说明，**匹配规则的写法三个面板通用**，统一放在最前面。

## 一览

| 面板 | 设置入口 | 能做什么 |
| --- | --- | --- |
| **Xboard** | 「节点管理 → 路由管理」建路由组，节点编辑里的「路由组」勾选 | 禁止访问、直连、指定 DNS、转发到自定义出站 |
| | 节点编辑 →「高级协议配置 → 路由」里的「自定义 Outbounds」「自定义 Routes」 | 按 Xray 格式定义出站和分流规则，只对这一个节点生效 |
| **V2board / v2node** | 「节点管理 → 路由管理」建路由，节点编辑里选择 | 按域名 / IP / 端口 / 协议禁止访问、指定 DNS、指定出站服务器、自定义默认出站 |
| **PPanel** | 「服务器 → 全局节点默认配置」，单台服务器在「节点配置」里覆盖 | DNS 配置、屏蔽规则、出站规则 |

所有规则从上往下匹配，**先命中的生效**，后面的不再看。完整顺序：

<ol class="kz-chain">
<li>本机白名单</li>
<li>本机屏蔽端口 / 屏蔽列表</li>
<li>本机 DNS 规则</li>
<li>自定义 Routes <small>Xboard</small></li>
<li>面板路由管理</li>
<li>本机路由文件 <small>routes_file</small></li>
<li>默认出口</li>
</ol>

本机的几项见 [审计与屏蔽](../features/audit.md)、[DNS 解析](../features/dns.md)、[分流与出口](../features/routing.md)。节点配置里的 `forbidden_bit_torrent`、`forbidden_private_ip` 在所有规则之前检查，白名单也不能放行它们。

## 匹配规则写法 { #match }

面板里的「匹配规则 / 匹配值」每行一条，一条规则里的多行是「或」的关系，命中任意一行即可。

| 写法 | 含义 | 例子 |
| --- | --- | --- |
| `domain:域名` | 该域名**及其所有子域名** | `domain:google.com` 匹配 `google.com`、`www.google.com` |
| `full:域名` | 只匹配这个域名本身 | `full:www.google.com` |
| `keyword:关键词` | 域名里包含这段文字 | `keyword:netflix` |
| `regexp:正则` | 正则表达式 | `regexp:^ad[0-9]+\.` |
| `geosite:分类` | geosite 域名分类 | `geosite:netflix`、`geosite:category-ads-all` |
| IP 或 CIDR | 目标 IP 段 | `8.8.8.8`、`10.0.0.0/8`、`2001:db8::/32` |
| `geoip:地区` | geoip IP 分类 | `geoip:cn`、`geoip:private` |
| `port:端口` | 目标端口，可写范围，逗号分隔 | `port:25`、`port:6881-6889,51413` |
| `protocol:协议` | 识别出的应用协议 | `protocol:bittorrent`、`protocol:quic` |
| 不带前缀的域名 | **各面板含义不同**，见下 | `google.com` |

!!! warning "不带前缀的域名，三个面板含义不同"
    kaze 按各面板自家节点的习惯解释，和在原节点上的效果一致：

    | 面板 | `google.com` 匹配 |
    | --- | --- |
    | Xboard | `google.com` 及其子域名（同 `domain:`）;`*.google.com` 也是同样效果 |
    | V2board / v2node | 域名里**包含** `google.com` 的都算（同 `keyword:`） |
    | PPanel | 只匹配 `google.com` 本身（同 `full:`） |

    不想记差别，就**统一带前缀写**，三个面板效果完全一样。

使用 `geosite:`、`geoip:` 前，在节点上执行一次 `kaze geo` 下载规则数据。没下载时日志会报警告：面板路由里只跳过那一行，规则的其它行照常生效；自定义 Routes 里则整条规则跳过。

## Xboard

### 路由管理 { #xboard-routes }

在「节点管理 → 路由管理」点「添加路由」，然后在节点编辑页的「路由组」里勾选要生效的路由。

| 动作 | 动作值 | 效果 | 例子 |
| --- | --- | --- | --- |
| 禁止访问 | 不填 | 命中的连接直接拒绝 | 匹配 `geosite:category-ads-all` |
| 直连 | 不填 | 从节点本机直接出去，后面的规则不再看 | 匹配 `domain:apple.com` |
| 指定DNS服务器进行解析 | DNS 服务器 | 用这个 DNS 解析命中的域名，然后直连 | `1.1.1.1`、`https://dns.google/dns-query` |
| 转发 | 出站标签（tag） | 交给这个节点「自定义 Outbounds」里同名的出站 | `warp` |

DNS 服务器的写法：

| 写法 | 类型 |
| --- | --- |
| `8.8.8.8` 或 `8.8.8.8:53` | 普通 DNS(UDP) |
| `tcp://8.8.8.8` | DNS over TCP |
| `tls://1.1.1.1` | DNS over TLS（默认 853 端口） |
| `https://dns.google/dns-query` | DNS over HTTPS |
| `tls://1.1.1.1#cloudflare-dns.com` | 用 IP 连接，按 `#` 后的名称验证证书（PPanel 的「TLS 服务器名称」就是这个） |
| `1.1.1.1,8.8.8.8` | 多个用英文逗号隔开，前一个失败再问下一个 |

!!! note "「转发」要和自定义 Outbounds 配合"
    路由组是所有节点共用的，出站却是每个节点各自定义的。「转发」到 `warp`，只有在「自定义 Outbounds」里定义了 `"tag": "warp"` 的节点上才会生效；其它节点上这条规则跳过，流量直连，日志里有一行警告说明原因。

!!! info "Xray 写法和 sing-box 写法都认，推荐 Xray 写法"
    「自定义 Outbounds」和「自定义 Routes」两个框里，Xray 的 JSON（面板提示的格式）和 sing-box 的 JSON 都能识别，可以混用：出站按 `protocol` / `type` 判断类型，字段名两套都接受（`address` 或 `server`、`port` 或 `server_port`、`outboundTag` 或 `outbound`……）。推荐按 **Xray 写法** 填，和面板提示、网上教程一致。

    kaze 不认识或不支持的字段（比如出站的 `ws` / `grpc` / `reality` 传输、规则里的 `balancerTag`、`user`、`source`、sing-box 的 `rule_set`、`process_name`、`clash_mode`）会在**节点启动日志里逐条提示**并忽略，其余部分照常生效；`kaze check` 也会列出来。所以写完看一眼日志就知道有没有被忽略的东西。

### 自定义 Outbounds { #xboard-outbounds }

节点编辑 →「高级协议配置」→「路由」→「自定义 Outbounds」，填一个 JSON 数组，每一项是一个出站，格式与 Xray 相同：

```json
[
  {"tag": "出站标签", "protocol": "类型", "settings": {...}, "streamSettings": {...}}
]
```

| 字段 | 说明 |
| --- | --- |
| `tag` | 出站的名字，路由规则用它引用，同一节点内不要重复 |
| `protocol` | 出站类型，见下表 |
| `settings` | 服务器地址、端口、密码等，按类型填写 |
| `streamSettings` | 可选，只用于开启 TLS；不填就是不加密的 TCP |

支持的类型：

| `protocol` | 用途 | UDP |
| --- | --- | --- |
| `freedom` / `direct` | 直连（不是代理，引用它等于「直连」） | — |
| `blackhole` / `block` / `reject` | 拒绝（引用它等于「禁止访问」） | — |
| `wireguard` | WireGuard，包括 Cloudflare WARP | ✅ |
| `socks` | SOCKS5 代理 | ✅ |
| `http` | HTTP 代理 | ❌ |
| `shadowsocks` | 转发给另一台 Shadowsocks 服务器 | ✅ |
| `trojan` | 转发给另一台 Trojan 服务器（总是 TLS） | ✅ |
| `vless` | 转发给另一台 VLESS 服务器（不支持 flow） | ✅ |

出站之间只能用 TCP 传输，TLS 可选。写了 `ws`、`grpc`、`reality` 等传输的出站不会被启用，日志里会说明，引用它的流量直连。除 HTTP 外的出站都转发 UDP；指向 HTTP 出站的 UDP 流量会被丢弃，**不会**从节点直连泄露 IP。

各类型 `settings` 的写法：

=== "WireGuard / WARP"

    ```json
    {"tag": "warp", "protocol": "wireguard",
     "settings": {"secretKey": "YO2b...你的私钥..."}}
    ```

    只填私钥时，其余参数自动按 Cloudflare WARP 默认值补齐。连接自建 WireGuard 时写全：

    ```json
    {"tag": "wg", "protocol": "wireguard",
     "settings": {
       "secretKey": "本机私钥",
       "address": ["172.16.0.2/32"],
       "mtu": 1280,
       "peers": [{"publicKey": "对端公钥", "endpoint": "wg.example.com:51820"}]
     }}
    ```

    WARP 私钥的获取方法见 [走 WARP 出口](../features/routing.md#warp)。

=== "SOCKS5"

    ```json
    {"tag": "us", "protocol": "socks",
     "settings": {"servers": [{
       "address": "1.2.3.4", "port": 1080,
       "users": [{"user": "用户名", "pass": "密码"}]
     }]}}
    ```

    没有认证就去掉 `users`。

=== "HTTP"

    ```json
    {"tag": "corp", "protocol": "http",
     "settings": {"servers": [{
       "address": "proxy.example.com", "port": 8080,
       "users": [{"user": "用户名", "pass": "密码"}]
     }]}}
    ```

=== "Shadowsocks"

    ```json
    {"tag": "hk", "protocol": "shadowsocks",
     "settings": {"servers": [{
       "address": "hk.example.com", "port": 8388,
       "method": "2022-blake3-aes-128-gcm",
       "password": "落地机的密钥"
     }]}}
    ```

    `method` 支持 `aes-128-gcm`、`aes-192-gcm`、`aes-256-gcm`、`chacha20-ietf-poly1305`、`2022-blake3-aes-128-gcm`、`2022-blake3-aes-256-gcm`。

=== "Trojan"

    ```json
    {"tag": "jp", "protocol": "trojan",
     "settings": {"servers": [{
       "address": "jp.example.com", "port": 443,
       "password": "落地机的密码"
     }]},
     "streamSettings": {"security": "tls",
       "tlsSettings": {"serverName": "jp.example.com"}}}
    ```

    `serverName` 不填时用 `address`。落地机用自签证书时在 `tlsSettings` 里加 `"allowInsecure": true`。

=== "VLESS"

    ```json
    {"tag": "sg", "protocol": "vless",
     "settings": {"vnext": [{
       "address": "sg.example.com", "port": 443,
       "users": [{"id": "落地机的 UUID", "encryption": "none"}]
     }]},
     "streamSettings": {"security": "tls",
       "tlsSettings": {"serverName": "sg.example.com"}}}
    ```

    落地机必须是 VLESS + TCP + TLS（或不加密），不能开 Vision 流控和 REALITY。

!!! tip
    sing-box 风格的扁平写法也能识别，比如 `{"tag": "hk", "protocol": "shadowsocks", "settings": {"server": "hk.example.com", "server_port": 8388, "method": "...", "password": "..."}}`。

### 自定义 Routes { #xboard-custom-routes }

同一页的「自定义 Routes」，填一个 JSON 数组，每一项是一条规则，格式与 Xray 的路由规则相同，sing-box 的写法也能识别：

```json
[
  {"domain": ["geosite:netflix", "domain:disneyplus.com"], "outboundTag": "warp"},
  {"network": "udp", "port": "443", "outboundTag": "block"}
]
```

匹配条件：

| 字段 | 说明 | 例子 |
| --- | --- | --- |
| `domain` | 域名，写法同上文 [匹配规则写法](#match);**不带前缀的是包含匹配**（Xray 的规矩） | `["domain:google.com", "geosite:openai"]` |
| `domain_suffix` | 域名及子域名（sing-box 写法） | `["google.com"]` |
| `domain_keyword` | 包含这段文字 | `["netflix"]` |
| `domain_regex` | 正则 | `["^ad[0-9]+\\."]` |
| `ip` / `ip_cidr` | 目标 IP 段，可用 `geoip:` | `["geoip:cn", "10.0.0.0/8"]` |
| `port` / `port_range` | 目标端口，`443`、`"80,443"`、`"6881-6889"` 都行 | `"443"` |
| `network` | `tcp`、`udp` 或 `tcp,udp` | `"udp"` |
| `protocol` | `bittorrent`、`quic`、`tls`、`http` | `["bittorrent"]` |

去向（三选一）：

| 字段 | 说明 |
| --- | --- |
| `outboundTag`(或 sing-box 的 `outbound`) | 自定义 Outbounds 里的 tag；也可以直接写 `direct`、`block` |
| `"action": "reject"` | 拒绝（sing-box 写法） |
| 都不写 | 直连 |

**组合逻辑**：同一个字段里的多个值是「或」，不同字段之间是「且」。上面第二条的意思是「UDP **且** 端口 443」，也就是 QUIC，不会误伤 TCP 443 的 HTTPS。

`inboundTag`、`ruleTag`、`type: field` 这类字段可以照写，单入站的节点上不影响匹配。其余 kaze 不认识的字段会在日志里提示，规则按剩下的条件生效。

自定义 Routes 排在面板路由管理**之前**，两边冲突时以这里为准。

## V2board / v2node { #v2board-v2node }

「节点管理 → 路由管理」添加路由，在节点编辑里选择要生效的路由。V2board 自带节点和 v2node 两种对接方式的格式相同。

| 动作 | 匹配值 | 动作值 | 效果 |
| --- | --- | --- | --- |
| 禁止访问（域名目标） | 域名 | 不填 | 拒绝访问这些域名 |
| 禁止访问（IP目标） | IP / CIDR / `geoip:` | 不填 | 拒绝访问这些 IP |
| 禁止访问（端口目标） | 端口，如 `25`、`6881-6889` | 不填 | 拒绝访问这些端口 |
| 禁止访问（协议） | `bittorrent` 等 | 不填 | 拒绝这类协议 |
| 指定DNS服务器进行解析 | 域名 | DNS 服务器，写法同 [Xboard](#xboard-routes) | 用它解析，然后直连 |
| 指定出站服务器（域名目标） | 域名 | 出站 JSON | 命中的域名走这个出站 |
| 指定出站服务器（IP目标） | IP / CIDR | 出站 JSON | 命中的 IP 走这个出站 |
| 自定义默认出站 | 不填 | 出站 JSON | 没有任何规则命中的流量都走它 |

「出站 JSON」就是 **一个** Xray 出站对象，格式和 Xboard 的 [自定义 Outbounds](#xboard-outbounds) 里的单项完全一样，`tag` 可以省略。例如把 ChatGPT 转给 WARP:

```text
动作:  指定出站服务器(域名目标)
匹配值: geosite:openai
动作值: {"protocol": "wireguard", "settings": {"secretKey": "YO2b...你的私钥..."}}
```

!!! note
    「自定义默认出站」只能指向真正的代理或 WireGuard；填 `blackhole` 会切断全部流量，kaze 不会执行，并在日志里提示。

## PPanel

「服务器」页右上角「全局节点默认配置」里设置，所有服务器共用；某台服务器要单独设置时，点它的「节点配置」，关掉对应的「使用全局…」开关再填。

### DNS 配置

每一项填类型、地址和域名：

| 字段 | 例子 | 说明 |
| --- | --- | --- |
| 类型 | `UDP` / `TCP` / `TLS` / `HTTPS` | DNS over QUIC 不支持，这一项会被跳过并在日志提示 |
| 地址 | `8.8.8.8:53`、`1.1.1.1`、`dns.google/dns-query` | HTTPS 类型只写主机时自动补 `/dns-query` |
| TLS 服务器名称 | `cloudflare-dns.com` | 可选。TLS / HTTPS 类型的地址填 IP 时，用这个名称验证证书 |
| 域名 | `geosite:netflix`、`suffix:openai.com` | 这些域名用该 DNS 解析，然后直连 |

### 屏蔽规则

每行一条，命中的连接直接拒绝：

```text
geosite:category-ads-all
suffix:doubleclick.net
keyword:tracker
port:25
10.0.0.0/8
```

### 出站规则

每个出站填名称、协议、地址、端口和认证，「规则」里每行一条，命中的流量走这个出站。

| 协议 | kaze 是否支持 |
| --- | --- |
| SOCKS、HTTP、Shadowsocks、VLESS、Trojan | ✅（TCP,TLS 可选） |
| Direct | ✅ 规则命中的直连 |
| Reject | ✅ 规则命中的拒绝 |
| VMess、Hysteria2、TUIC、AnyTLS | ❌ 该出站跳过，它的规则直连，日志提示 |

传输只支持 TCP,`ws`、`grpc`、`xhttp` 以及 REALITY 的出站不会被启用。

PPanel 的规则写法：不带前缀是**完整域名**,`suffix:` 是域名及子域名，`keyword:` 是包含，`regex:` 是正则；上文 [匹配规则写法](#match) 里的 `domain:`、`geosite:`、`port:` 等也都能用。

## 常用配方

### 流媒体和 AI 走 WARP

=== "Xboard"

    自定义 Outbounds:

    ```json
    [{"tag": "warp", "protocol": "wireguard", "settings": {"secretKey": "YO2b...你的私钥..."}}]
    ```

    自定义 Routes:

    ```json
    [{"domain": ["geosite:netflix", "geosite:disney", "geosite:openai"], "outboundTag": "warp"}]
    ```

=== "V2board / v2node"

    ```text
    动作:  指定出站服务器(域名目标)
    匹配值: geosite:netflix
           geosite:disney
           geosite:openai
    动作值: {"protocol": "wireguard", "settings": {"secretKey": "YO2b...你的私钥..."}}
    ```

=== "PPanel"

    PPanel 的出站不支持 WireGuard。在节点上用 [本机路由文件](../features/routing.md#warp) 配置 WARP。

### 屏蔽 QUIC，让客户端改走 TCP

部分审计和测速依赖 TCP，屏蔽 UDP 443 后浏览器和 App 会自动改用 TCP。

=== "Xboard"

    自定义 Routes:

    ```json
    [{"network": "udp", "port": "443", "outboundTag": "block"}]
    ```

=== "V2board / v2node"

    ```text
    动作:  禁止访问(协议)
    匹配值: quic
    ```

=== "PPanel"

    屏蔽规则：

    ```text
    protocol:quic
    ```

### 中转机转发到落地机

入口（中转）机对用户提供服务，把全部流量交给落地机，用户看到的出口 IP 是落地机的。

=== "Xboard"

    自定义 Outbounds:

    ```json
    [{"tag": "landing", "protocol": "shadowsocks",
      "settings": {"servers": [{"address": "落地机IP", "port": 8388,
        "method": "2022-blake3-aes-128-gcm", "password": "落地机的密钥"}]}}]
    ```

    自定义 Routes(`port` 覆盖全部端口，等于「全部流量」):

    ```json
    [{"port": "1-65535", "outboundTag": "landing"}]
    ```

=== "V2board / v2node"

    ```text
    动作:  自定义默认出站
    动作值: {"protocol": "shadowsocks", "settings": {"servers": [{"address": "落地机IP", "port": 8388, "method": "2022-blake3-aes-128-gcm", "password": "落地机的密钥"}]}}
    ```

=== "PPanel"

    出站规则添加一项：协议 Shadowsocks，地址、端口、加密方式、密码填落地机的；规则写 `port:1-65535`。

!!! tip
    UDP 也会一起转发给落地机，游戏、语音照常可用。落地机是 Shadowsocks 时，需在落地机上开启 UDP（大多数面板和后端默认开启）。

### 屏蔽广告和 BT

=== "Xboard"

    路由管理添加：动作「禁止访问」，匹配规则：

    ```text
    geosite:category-ads-all
    protocol:bittorrent
    ```

=== "V2board / v2node"

    添加两条：「禁止访问（域名目标）」匹配 `geosite:category-ads-all`;「禁止访问（协议）」匹配 `bittorrent`。

=== "PPanel"

    屏蔽规则：

    ```text
    geosite:category-ads-all
    protocol:bittorrent
    ```

只想全局禁 BT 时，也可以直接在节点配置里写 `forbidden_bit_torrent=true`，见 [审计与屏蔽](../features/audit.md)。

## 规则没生效怎么查

kaze 每次从面板拉到配置都会检查路由，写错或不支持的地方会在日志里逐条说明，不会悄悄忽略：

```bash
kaze log 200 | grep -iE "custom|panel#|outbound"
```

| 日志里出现 | 原因 |
| --- | --- |
| `outbound "warp" is not defined or not supported` | 规则引用的 tag 在这个节点的自定义 Outbounds 里没有，或那个出站没建成 |
| `transport "ws" is not supported for outbounds` | 出站用了 TCP 以外的传输 |
| `needs geosite_file to be set` | 用了 `geosite:` 但还没执行 `kaze geo` |
| `condition(s) xxx are not supported` | 自定义 Routes 里有 kaze 不认识的字段，已按其余条件生效 |
| `the outbound in the rule's value is not valid JSON` | V2board 动作值的 JSON 写错了，检查引号和逗号 |
