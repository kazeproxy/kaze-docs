# 对接 PPanel

PPanel 的节点模型和 Xboard / V2board 不同，先弄清这点，配置就好懂了：

| | Xboard / V2board | PPanel |
| --- | --- | --- |
| 一个节点 ID 对应 | **一种协议**（VLESS 和 Trojan 是两个节点、两个 ID） | **一台服务器**（在它里面勾选要开的协议，每种协议各设一个端口） |
| `node_id` 填 | 节点 ID | 服务器 ID |
| `server_type` 填 | 不用填（Xboard）或填一种 | 要跑这台服务器上的**哪几种协议**，逗号隔开 |

所以 `node_id=5` 加 `server_type=vless,hysteria` 的意思是：PPanel 里 ID 为 5 的服务器开了 VLESS 和 Hysteria2，kaze 两个都跑，各自监听面板里给它们设的端口，**一个进程就能全部带起来**。协议名按 kaze 的写法：PPanel 里叫 hysteria2 的这里写 `hysteria`，Shadowsocks 写 `ss` 或 `shadowsocks`。

## 节点怎么配

=== "只跑一种协议"

    ```ini
    type=ppanel
    node_id=5                     # PPanel 里的服务器 ID
    server_type=vless
    webapi_url=https://你的面板
    webapi_key=通讯密钥             # PPanel 的 secret_key
    ```

=== "一个服务器跑多种协议"

    ```ini
    type=ppanel
    node_id=5
    server_type=vless,trojan,hysteria,tuic
    webapi_url=https://你的面板
    webapi_key=通讯密钥
    ```

    列出的每种协议都会按 PPanel 里设置的端口各自启动。只列出一部分时，其余协议不跑；PPanel 里没启用的协议写了会报错并列出该服务器实际开了哪些。

=== "Docker Compose"

    ```yaml
    cap_add:
      - NET_ADMIN                 # 用端口跳跃时需要
    environment:
      type: ppanel
      node_id: "5"
      server_type: vless,trojan,hysteria,tuic
      panel_url: https://你的面板
      panel_key: 通讯密钥
    ```

    完整文件见 [Docker 部署](../quickstart/docker.md)。

=== "多个服务器"

    一台机器对接 PPanel 里的两台「服务器」（比如一台只给 VIP 套餐用，一台给普通套餐）：

    ```ini
    type=ppanel
    node_id=5,6
    webapi_url=https://你的面板
    webapi_key=通讯密钥

    [node 5]
    server_type=vless,trojan

    [node 6]
    server_type=shadowsocks
    ```

!!! info "名称对照"
    | PPanel 里叫 | kaze 配置里写 |
    | --- | --- |
    | Server ID | `node_id` |
    | Secret Key | `webapi_key`(也可写 `panel_key`) |
    | hysteria2 | `hysteria` |
    | shadowsocks | `ss` 或 `shadowsocks` |

!!! warning "协议必须在 PPanel 里启用"
    `server_type` 写的协议在该服务器上没有启用时，节点会报错并列出这台服务器实际开了哪些协议，照着改即可。

## 面板里的这些设置会自动生效

| PPanel 设置 | 在 kaze 上 |
| --- | --- |
| 端口、加密、传输（TCP / WebSocket / gRPC / HTTPUpgrade / XHTTP / H2）、路径 | :material-check: 自动生效 |
| TLS、REALITY、Vision 流控 | :material-check: 自动生效 |
| Shadowsocks 2022 服务端密钥 | :material-check: 自动生效 |
| Hysteria2 混淆、带宽 | :material-check: 自动生效 |
| Hysteria2 端口跳跃 | :material-check: 自动生效：面板节点设置里的跳跃端口范围会下发给节点，节点配置不用改（需要 `CAP_NET_ADMIN`，一键安装已给），见[端口跳跃](../features/port-hopping.md) |
| AnyTLS 填充方案 | :material-check: 自动生效 |
| 接收 PROXY protocol | :material-check: 自动生效 |
| Shadowsocks 插件 simple-obfs(http)、Shadow TLS v3 | :material-check: 自动生效；Shadow TLS 的版本要选 3 |
| 证书模式 http / dns | :material-check: 自动生效 |
| 证书模式 self | :material-check: 自签证书保存在节点上，指纹自动上报，PPanel 订阅会锁定它，客户端不用开「允许不安全」 |
| 证书模式 file | :material-check: PPanel 不下发路径，在节点配置填 `cert_file`、`key_file` |
| 流量上报阈值 | :material-check: 自动生效 |
| 用户限速、设备数 | :material-check: 自动生效 |
| 屏蔽规则、DNS 规则、出站规则 | :material-check: 转换为 kaze 路由，规则写法与 PPanel 一致，见 [面板路由与自定义出站](routing.md#ppanel) |
| 流量倍率（ratio） | :material-scale-balance: 面板计费时处理，节点只上报真实流量 |

## 暂不支持的设置

开了以下设置时，kaze 会在启动日志里**明确写出**哪一项没有生效，不会悄悄忽略。

| 设置 | 说明 / 替代办法 |
| --- | --- |
| ECH | 关闭 |
| 出站的 WebSocket / gRPC / REALITY | 出站只支持 TCP（可带 TLS） |
| ip_strategy | 在节点上设置，如 `dns_strategy=ipv4_only`(可选 `ipv4_first` / `ipv4_only` / `ipv6_first` / `ipv6_only`) |
| 出站 VMess / Hysteria2 / TUIC / AnyTLS | 改用 SOCKS、Shadowsocks、Trojan、VLESS 出站 |
| DNS over QUIC | 改用 UDP / TLS / HTTPS |
| Shadowsocks 插件 v2ray-plugin / gost-plugin / restls / kcptun、obfs 的 tls 模式 | 改用 simple-obfs http 或 Shadow TLS v3 |
| Mieru 流量模式、强制用户提示 | 关闭 |
| XHTTP 选 h3 | 改用 h2 / http/1.1 |
