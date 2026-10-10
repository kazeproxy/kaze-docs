# 分流与出口（节点文件）

写在节点 `routes_file` 里的本机路由和出口看这页；面板里配的分流、屏蔽、出站看[路由与自定义出站（面板里配）](../panel/routing.md)。

决定用户流量从哪里出去。常见场景：入口机在国内或中转，出口机在海外，入口把流量转发给出口。

## 两层规则

1. **面板下发的路由规则**（在面板里配），优先匹配。
2. **本机路由文件**(`routes_file`)，面板没命中时再匹配。

面板支持的规则：按域名 / IP / 端口 / 协议屏蔽、直连放行、指定 DNS、走指定出口。三个面板各自的格式和例子见 [面板路由与自定义出站](../panel/routing.md)。

两边都设了时：

- 规则按顺序**第一个命中的生效**：节点本机的屏蔽 BT / 屏蔽内网 → 白名单 → 屏蔽列表 → 本机 DNS 规则 → 面板规则（Xboard 的自定义 Routes 排在路由管理之前）→ 本机 `[route]`。面板只处理它命中的流量，其余交给本机文件，两边是叠加。
- 兜底出口：V2board 的「自定义默认出站」设了就用它，没设才用本机 `[default]`；Xboard 的自定义 Routes 没有兜底项。
- 出口名不要重复：面板自定义出站和本机 `[outbound 名字]` 同名时，本机文件会覆盖面板那个。

## 本机路由文件

落地机最省事的做法是也装 kaze，用 `type=local`（不接面板、不占授权人数）开一个 Shadowsocks 2022 或 VLESS，见[不接面板，直接搭节点](../quickstart/local.md)；入口机的路由文件把流量交给它。

在节点上定义出口、出口组（负载均衡）和分流规则：

```ini
routes_file=/etc/kaze/routes.conf
```

文件格式：

```ini
[outbound us1]
type=trojan
server=us1.example.com
port=443
password=密码

[outbound us2]
type=ss
server=us2.example.com
port=8388
method=2022-blake3-aes-128-gcm
password=密钥

[group us]
outbounds=us1,us2
strategy=round_robin        # 轮流;也可以 random(随机)、failover(主备)

[route]
match=geosite:netflix, domain:example.com
outbound=us                 # 出口名、组名、direct 或 block

[route]
match=domain:only-node-9.test
outbound=us1
nodes=1,2                   # 可选:只对这些节点 ID 生效

[default]
outbound=us                 # 没有规则匹配的流量走这里
```

`[outbound 名字]` 里可用的键：

| 键 | 用于 | 说明 |
| --- | --- | --- |
| `type` | 全部 | `socks` / `http` / `wireguard` / `trojan` / `vless` / `ss` |
| `server`、`port` | 除 wireguard 外 | 落地机地址和端口 |
| `username`、`password` | socks / http | 代理的账号密码，没有就不填 |
| `password` | trojan / ss | 密码。落地机是 Shadowsocks 2022 多用户（比如 kaze 的 `type=local`）时填 `服务端密钥:用户密钥`，和客户端的密码一样 |
| `method` | ss | 加密算法 |
| `uuid` | vless | 用户 UUID；不支持 flow |
| `tls`、`sni`、`insecure` | trojan / vless / socks / http | `tls=true` 开 TLS（trojan 总是开），`sni` 证书域名，`insecure=true` 跳过校验 |
| `private_key`、`public_key`、`endpoint`、`address`、`reserved`、`mtu` | wireguard | 自建 WireGuard 全填；WARP 只填 `private_key`，其余自动（`reserved` 也不用） |

`[route]` 的 `match` 除了域名、`geosite:`、`geoip:`、IP 段，还可以写 `port:27015-27030`、`network:udp`（只匹配 UDP，和其它条件是「且」）。游戏类流量按 IP 段 + 端口分流：

```ini
[route]
match=geoip:jp, network:udp
outbound=jp
```

## 验证出口

客户端连上节点后访问 `https://ifconfig.me`，返回的 IP 应是落地机或 WARP 的；访问 `https://www.cloudflare.com/cdn-cgi/trace` 看 `warp=on`。命令行可以直接测：`curl --proxy socks5h://用户:密码@节点IP:端口 https://ifconfig.me`（SOCKS5 节点）。`log_level=debug` 时日志会写每条连接走了哪个出口。

## 支持的出口类型

| 类型 | UDP | 说明 |
| --- | --- | --- |
| `socks` | ✅ | SOCKS5 代理 |
| `http` | ❌ | HTTP 代理 |
| `wireguard` | ✅ | WireGuard，含 Cloudflare WARP |
| `trojan` | ✅ | 转发给另一个 Trojan 节点 |
| `vless` | ✅ | 转发给另一个 VLESS 节点（每个 UDP 目标一条连接，单个会话最多 64 个目标） |
| `ss` | ✅ | 转发给另一个 Shadowsocks 节点（经典 AEAD 与 2022，单用户或多用户） |

!!! note
    除 HTTP 代理外，所有出口都转发 UDP，中转到落地机时游戏、语音照常可用。指向 HTTP 代理的 UDP 流量会被**丢弃，不会直连**，以免暴露节点 IP。出口暂时故障时 UDP 同样丢弃，5 秒后重试。

## 改规则文件不用重启

本机路由文件、DNS 规则、屏蔽列表、白名单和 geo 数据文件，保存后 10 秒内自动生效，日志里会有一行 `routing reloaded`。只换规则，不动监听，用户的连接不会断。

多台节点共用一份路由时，把文件放到网上，节点用 `routes_url` 定时拉取（代替 `routes_file`）：

```ini
routes_url=https://example.com/kaze/routes.conf
routes_update_interval=30      # 分钟，默认 60
```

## 走 WARP 出口 { #warp }

让部分网站（比如流媒体、ChatGPT）从 Cloudflare WARP 出去。只需要一个 WARP 账号的**私钥**，其余（对端公钥、地址、端点、`reserved`）kaze 自动补上 WARP 的默认值。

**获取私钥**：用开源工具 [wgcf](https://github.com/ViRb3/wgcf) 注册一个免费 WARP 账号：

```bash
wgcf register && wgcf generate
grep PrivateKey wgcf-profile.conf
# PrivateKey = YO2b...(就是它)
```

=== "写在本机路由文件"

    ```ini
    [outbound warp]
    type=wireguard
    private_key=YO2b...你的私钥...

    [route]
    match=geosite:openai, geosite:netflix
    outbound=warp
    ```

=== "写在 Xboard 节点的自定义出站"

    自定义 Outbounds:

    ```json
    [{"tag": "warp", "protocol": "wireguard", "settings": {"secretKey": "YO2b...你的私钥..."}}]
    ```

    自定义 Routes:

    ```json
    [{"domain": ["geosite:openai", "geosite:netflix"], "outboundTag": "warp"}]
    ```

用 `geosite:` 规则前，先执行一次 `kaze geo` 下载规则数据。

## 源进源出

多 IP 服务器上，用户从哪个本机 IP 连进来，流量就从哪个 IP 出去。

```ini
auto_out_ip=true          # 默认 false，走系统默认出口
```

!!! note
    对基于 TCP 的协议有效；Hysteria2 和 TUIC 走系统默认出口。
