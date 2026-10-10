# 协议与传输

协议、传输、安全层都在**面板节点编辑页**设置，kaze 自动同步。这里说几个要点。

## 支持的协议

| 协议 | UDP | 说明 |
| --- | --- | --- |
| Trojan | ✅ | |
| SOCKS5 | ✅ | 可开 TLS；只核对密码（面板订阅里用户名和密码都是 UUID，本地模式用户名可以是名字） |
| HTTP 代理 | ❌ | 可开 TLS；不转发 UDP；认证同 SOCKS5 |
| Shadowsocks | ✅ | 经典 AEAD 和 2022 都支持 |
| VLESS | ✅ | 支持 Vision 流控 |
| VMess | ✅ | |
| Hysteria2 | ✅ | 基于 QUIC，必须配 TLS 证书 |
| TUIC v5 | ✅ | 基于 QUIC，必须配 TLS 证书 |
| AnyTLS | ✅ | |
| Mieru | ✅ | 仅 TCP 传输 |

Shadowsocks 支持的加密：`aes-128-gcm`、`aes-192-gcm`、`aes-256-gcm`、`chacha20-ietf-poly1305`、`2022-blake3-aes-128-gcm`、`2022-blake3-aes-256-gcm`。

## 支持的传输

`tcp`、`ws`(WebSocket)、`httpupgrade`、`grpc`、`h2`、`xhttp`(SplitHTTP)，以及 QUIC（Hysteria2 / TUIC 专用）。

**多路复用**：面板里给节点开了多路复用的，客户端用 smux / yamux / h2mux 都能连。

## REALITY

节点安全选 **REALITY**，不需要域名和证书。可用于 VLESS 和 Trojan。面板里要填：

| 面板字段 | 示例 | 说明 |
| --- | --- | --- |
| 伪装域名 | `www.microsoft.com` | 一个支持 TLS 1.3 的大网站。不是你的用户的访问会被原样转给它，探测者只会看到这个网站 |
| 伪装站点端口 | `443` | 伪装网站的端口，一般 443 |
| 私钥 / 公钥 | 点生成按钮 | Xboard 点私钥输入框右边的钥匙按钮，私钥和公钥一起生成。私钥给节点，公钥给客户端 |
| Short ID | `6af819e1` | 0~16 位十六进制，点刷新按钮生成或随便填一个 |

这些值由面板下发给节点，节点上不用再配。

面板没有生成按钮时，可以在服务器上运行 `kaze -reality-keypair` 生成一对，PrivateKey 填私钥、PublicKey 填公钥。

!!! note
    REALITY 能与官方客户端互通。目前握手回包没有模仿伪装站点的形态，针对性的流量分析有可能区分，普通使用无影响。

## Vision 流控

VLESS 节点的流控选 `xtls-rprx-vision`，传输方式选 `tcp`，安全性选 TLS 或 REALITY。这是最常用、最快的 VLESS 配置：

| 面板字段 | 填 |
| --- | --- |
| 传输方式 | `tcp` |
| 安全性 | `REALITY`(或 `TLS`) |
| 流控 | `xtls-rprx-vision` |

## Shadow TLS（内置，无需额外部署）

Shadowsocks 节点的插件选 **Shadow TLS**，插件选项填：

```
host=伪装域名;password=密码;version=3
```

面板里「连接端口」和「服务端口」填**同一个**端口。kaze 内置 Shadow TLS v3，**不需要**另外运行 shadow-tls 程序，也不用多占一个端口——其它后端通常要在 Shadowsocks 前面再挂一个 shadow-tls 进程并打通两个端口。

## TLS 证书

Trojan、AnyTLS、Hysteria2、TUIC，以及开了 TLS 的 VLESS / VMess 都需要证书。最简单的做法：

1. 把域名（如 `node1.example.com`）解析到服务器 IP;
2. 面板节点的「服务器名称 SNI」填这个域名，证书模式选 **http**（Xboard：高级协议配置 → TLS）；
3. 保证服务器 **80 端口**能从公网访问，kaze 会自动申请证书并续期。不选证书模式时用的是自签证书。

80 端口用不了、或者想用泛域名证书，用 DNS 验证；已经有证书文件也可以直接用。详见 [TLS 证书](../features/cert.md)。
