# 面板设置对照

面板里每一项设置在 kaze 上的效果。图例：

- :material-check-circle:{ style="color:#2e7d32" } **生效**：面板改了，节点自动跟着变
- :material-scale-balance:{ style="color:#1565c0" } **面板负责**：这项由面板自己处理（计费、订阅），和节点无关，也不会冲突
- :material-alert-circle:{ style="color:#ef6c00" } **暂不支持**：节点启动日志会明确提示，不会悄悄失效

!!! tip "限速和倍率分别由谁负责"
    - **限速、设备数**：由面板决定数值（写在套餐里，购买时复制到用户），**节点负责执行**。
    - **倍率、动态倍率**：面板在扣流量时按「真实流量 × 倍率」计算，**节点只上报真实流量**，不参与倍率计算。

    所以两者分工明确，不会冲突。节点上额外设置的 `user_speed_limit` 只会和面板值**取较小者**，只收紧、不放宽。

## 用户与计费

| 设置 | Xboard | V2board | PPanel |
| --- | :---: | :---: | :---: |
| 套餐 / 用户限速 | :material-check-circle: | :material-check-circle: | :material-check-circle: |
| 设备数限制 | :material-check-circle: | :material-check-circle: | :material-check-circle: |
| 跨节点设备数汇总 | :material-check-circle: | :material-check-circle: | :material-alert-circle: 面板未提供，按本节点计算 |
| 节点倍率 / 动态倍率 | :material-scale-balance: | :material-scale-balance: | :material-scale-balance: |
| 流量上报、在线 IP 上报 | :material-check-circle: | :material-check-circle: | :material-check-circle: |
| 机器负载上报 | :material-check-circle: | — | :material-check-circle: |
| 节点运行指标（连接数、网速等） | :material-check-circle: | — | — |
| WebSocket 实时推送 | :material-check-circle: | — | — |
| 推送 / 拉取间隔 | :material-check-circle: | :material-check-circle: | :material-check-circle: |

## 协议与传输

| 设置 | 状态 |
| --- | :---: |
| 端口、协议、加密 | :material-check-circle: |
| 传输:TCP / WebSocket / HTTPUpgrade / gRPC / H2 / XHTTP | :material-check-circle: |
| gRPC 多路模式（multiMode）、Xray 的自定义路径服务名（以 `/` 开头、竖线分隔两个路径的写法） | :material-check-circle: |
| TLS、REALITY、Vision 流控 | :material-check-circle: |
| 多路复用（smux / yamux / h2mux，含填充） | :material-check-circle: |
| Shadowsocks 2022、simple-obfs(http)、Shadow TLS v3 | :material-check-circle: Shadow TLS 要在插件选项里写 `version=3`；不写时面板给客户端的是 v2,节点会明确报错 |
| Hysteria2 混淆与带宽、TUIC、AnyTLS 填充、Mieru(TCP) | :material-check-circle: 带宽方向按各面板自己的订阅理解，用户上传上限会告知客户端 |
| TUIC 拥塞控制 BBR / Cubic | :material-check-circle: |
| TUIC 拥塞控制 NewReno | :material-alert-circle: 按 Cubic 运行（两者很接近），日志提示 |
| TUIC / XHTTP 的 ALPN | :material-check-circle: TUIC 接受 h3 / h2 / http/1.1;XHTTP 走 h2 / http/1.1（PPanel 里选 h3 会提示） |
| SOCKS5（含 UDP）、HTTP 代理，均可开 TLS | :material-check-circle: |
| 接收中转的 PROXY protocol | :material-check-circle: |
| CDN 真实 IP 请求头（trusted_x_forwarded_for） | :material-check-circle: |
| WebSocket / gRPC / XHTTP 的 Host 伪装 | :material-check-circle: 节点不校验 Host，客户端填什么都能连 |
| XHTTP 额外设置（extra） | :material-check-circle: 只影响客户端的项无需处理；改变上传方式或超过节点上限的项会提示 |
| Naive、ShadowsocksR、Snell 节点 | :material-alert-circle: |
| Hysteria 1、TUIC v4 | :material-alert-circle: 请改用 Hysteria2、TUIC v5 |
| VLESS 流控 xtls-rprx-direct / splice、mKCP 传输 | :material-alert-circle: 已被 Xray 淘汰，请改用 Vision、TCP |
| ECH | :material-alert-circle: |
| 多路复用 Brutal | :material-alert-circle: |
| TCP 的 HTTP 伪装头 | :material-alert-circle: |
| Mieru UDP 传输、流量模式、强制用户提示 | :material-alert-circle: |
| Shadowsocks UDP over TCP（uot，v1 / v2） | :material-check-circle: |
| VLESS 加密（mlkem） | :material-alert-circle: |
| Shadowsocks `2022-blake3-chacha20-poly1305`、`xchacha20-ietf-poly1305` | :material-alert-circle: |
| Shadowsocks 插件 v2ray-plugin / gost / kcptun / restls / obfs-tls | :material-alert-circle: |
| 端口跳跃（Hysteria2 / TUIC） | :material-check-circle: Xboard / V2board 不会把端口范围下发给节点，需在节点配置填 `hop_ports=20000-30000`（和面板连接端口一致）；PPanel 会下发，节点不用填。见[端口跳跃](../features/port-hopping.md)。Mieru 等 TCP 协议不适用 |

## 证书

| 证书模式 | 状态 |
| --- | :---: |
| self（自签） | :material-check-circle: 证书保存在节点上，重启不变；PPanel 会自动在订阅里锁定这张证书 |
| none | :material-check-circle: 节点自带证书时用节点的，否则自签；v2node 的「无证书（关闭 TLS）」表示前面有 Nginx / CDN 处理 TLS，节点不加密监听 |
| http（自动申请续期） | :material-check-circle: |
| content（面板直接下发证书内容） | :material-check-circle: |
| file（节点本地证书文件） | :material-check-circle: 面板不下发路径时，在节点配置填 `cert_file`、`key_file`;v2node 填的证书路径上已有文件时直接使用 |
| dns（DNS 验证） | :material-check-circle: Cloudflare、阿里云、腾讯云 DNSPod；其他服务商用 acme.sh 签发后填 `cert_file=/etc/kaze/cert.pem` 和 `key_file=/etc/kaze/key.pem` |

## 路由管理

| 面板动作 | 状态 |
| --- | :---: |
| 禁止访问（block） | :material-check-circle: |
| 直连（direct） | :material-check-circle: |
| 指定 DNS 服务器解析（dns） | :material-check-circle: |
| 转发（proxy，指向自定义出站） | :material-check-circle: |
| V2board 的 block_ip / block_port / protocol / route / route_ip / default_out | :material-check-circle: 协议可识别 http、tls、quic、bittorrent |
| 匹配：域名、子域名、`*.x`、`regexp:`、`geosite:`、`geoip:`、IP / CIDR、端口 | :material-check-circle: |

## 自定义出站与自定义路由（Xboard 高级设置）

| 设置 | 状态 |
| --- | :---: |
| 出站 socks / http / wireguard（含 WARP） | :material-check-circle: |
| 出站 freedom（直连）/ blackhole（屏蔽） | :material-check-circle: |
| 出站 trojan / vless / shadowsocks（TCP，可带 TLS） | :material-check-circle: |
| 出站的 WebSocket / gRPC / REALITY 传输 | :material-alert-circle: 会提示并直连 |
| 出站 vmess / hysteria2 / tuic / anytls | :material-alert-circle: 会提示并直连 |
| 路由条件:domain、domain_suffix / keyword / regex、ip、port、protocol、network(tcp/udp) | :material-check-circle: |
| 条件组合：同一字段任一满足，不同字段同时满足（与 Xray 一致） | :material-check-circle: |
| 路由条件:inboundTag、ruleTag | :material-check-circle: 单入站节点上不影响匹配 |
| 路由条件:source、user 等 | :material-alert-circle: 会提示，规则按其余条件生效 |
| 出站的 sendThrough、proxySettings、mux | :material-alert-circle: 会提示并忽略 |

完整格式和例子见 [面板路由与自定义出站](routing.md)。

!!! tip "自定义 Outbounds / Routes 用 Xray 格式"
    Xboard 的这两个框是原样转给节点的 JSON,kaze 按 **Xray 格式**理解，和面板示例一致，不需要改写。常用写法：
    ```json
    [{"type": "field", "network": "udp", "port": "443", "outboundTag": "block"}]
    ```
    上面这条屏蔽 QUIC(UDP 443)，不影响 TCP 443 的 HTTPS。
