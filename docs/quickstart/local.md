# 不接面板，直接搭节点

没有面板也能用：把协议、端口和用户直接写在 `kaze.conf` 里，`type=local`。适合自用的 SOCKS5 / Shadowsocks 出口、临时中转、给几个人用的小节点。免费版的 100 人上限对这种用法绰绰有余，不需要授权码。

```ini
type=local
server_type=socks            # socks / http / ss（或 shadowsocks）/ trojan / vless / vmess / hysteria / tuic / anytls / mieru
port=1080
users=alice:密码1,bob:密码2
```

一键安装时把面板参数换成这些设置：

```bash
bash <(curl -fsSL https://github.com/kazeproxy/kaze-release/raw/main/install.sh) install --type local --server-type socks port=1080 users=alice:密码1,bob:密码2
```

装好后客户端用 `服务器IP:1080`，用户名 `alice`、密码 `密码1` 连接。

## 每种协议填什么

`users` 一律是 `名字:密钥` 用逗号隔开；密钥对不同协议的含义：

| 协议 | `users` 里的密钥 | 还要填 |
| --- | --- | --- |
| `socks` / `http` | 密码（用户名随意，客户端填名字即可） | 建议 `tls=true` + 证书，否则密码明文；客户端要支持 SOCKS/HTTP over TLS（Shadowrocket 可以），自签证书要开允许不安全 |
| `shadowsocks` 经典 AEAD | 密码 | `cipher=aes-128-gcm` 等 |
| `shadowsocks` 2022 | 用户 UUID（客户端密码见下） | `cipher=2022-blake3-aes-128-gcm`、`server_key=`（`openssl rand -base64 16`，256 位算法用 32） |
| `trojan` | 密码 | 证书（见下） |
| `vless` / `vmess` | UUID | 可选 `network=ws` + `path=/xxx`、`tls=true` |
| `hysteria`（Hysteria2） | 密码 | 证书；可选 `obfs_password=`、`hop_ports=` |
| `tuic` | `uuid:密码` 两段一起写 | 证书 |
| `anytls` / `mieru` | 密码 | 证书 / 无 |

Shadowsocks 2022 的客户端密码是 `服务端密钥:用户密钥`，其中用户密钥是 kaze 从 `users` 里的 UUID 取前 16 字节转成的 base64（和 Xboard 一样），**不是** `server_key:UUID`——直接用 `kaze check` 打印出来的那一串（形如 `MDEy...==:NGY4...==`）。

带 Shadow TLS 的 Shadowsocks 2022 这样写：

```ini
[node 2]
server_type=ss
cipher=2022-blake3-aes-128-gcm
server_key=MDEyMzQ1Njc4OWFiY2RlZg==
port=443
users=carol:4f8a1c2e-0000-4000-8000-000000000001
plugin=shadow-tls
plugin_opts=host=cloud.tencent.com;password=任意一串;version=3
```

Shadowrocket：类型 Shadowsocks，算法 `2022-blake3-aes-128-gcm`，密码填 `kaze check` 打印的那串，插件 shadow-tls，插件选项的 host / password / version 和上面一致。

一键安装命令里的 `users=` 密码只用字母数字；带中文或特殊字符的请改配置文件。

## 证书

需要 TLS 的协议（Trojan、Hysteria2、TUIC、AnyTLS，以及开了 `tls=true` 的其它协议）和面板模式一样用 `cert_mode` / `cert_domain` 自动申请，或 `cert_file` / `key_file` 指定文件，见 [TLS 证书](../features/cert.md)。什么都不填时用自签证书，客户端需要允许不安全连接。

## 一台机器多个节点

每个节点一个 `[node N]` 段，公共部分是 1 号节点：

```ini
type=local
server_type=socks
port=1080
users=alice:密码1

[node 2]
server_type=shadowsocks
cipher=2022-blake3-aes-128-gcm
server_key=MDEyMzQ1Njc4OWFiY2RlZg==
port=8388
users=carol:4f8a1c2e-0000-4000-8000-000000000001

[node 3]
server_type=hysteria
port=443
users=dave:密码3
cert_mode=http
cert_domain=node.example.com
```

路由与出口、屏蔽规则、限速、设备数、DNS 等所有本机设置照常可用，见[功能配置](../features/routing.md)。改了 `kaze.conf` 要 `kaze restart`（本地模式没有面板推送）。

## 和面板模式的区别

- 没有流量统计与计费、没有到期和套餐——用户就是写在文件里的那几个。
- 没有面板下发的路由，只有本机路由文件。
- 不需要授权码：本地模式一律按免费版运行（每节点 100 个用户）。
