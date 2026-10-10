# TLS 证书

Trojan、AnyTLS、Hysteria2、TUIC，以及开了 TLS 的 VLESS / VMess 都需要证书。

证书来源的优先顺序：**节点配置 > 面板下发的证书 > 自动申请 > 自签**。

## 自动申请（推荐）

向 Let's Encrypt 自动申请并自动续期（到期前 30 天续）。要求域名已解析到本机，且本机 **80 端口能从公网访问**。

!!! note "什么都不选时是自签证书"
    面板里没选证书模式、节点配置也没写 `cert_mode` 时，kaze 用**自签证书**，客户端必须允许不安全连接。想让客户端正常校验，就按下面任一种方式申请真证书。

=== ":material-alpha-x-box: 在面板里设置（一般这样就够）"

    Xboard：节点管理 → 编辑节点 → **高级协议配置** → **TLS** 页签 → 证书模式选 **http**，填证书域名。V2board / v2node / PPanel 的入口见[面板设置对照 → 证书](../panel/compat.md)。节点配置不用改。

=== ":material-file-cog: 在节点配置里设置"

    面板没有证书设置、或想覆盖面板时，写在 `kaze.conf`：

    ```ini
    cert_mode=http
    cert_domain=node.example.com
    cert_email=you@example.com
    ```

申请到的证书存在配置文件同目录的 `certs` 文件夹。续期后自动重新加载，不用重启。

## DNS 验证自动申请

不方便开放 80 端口、用泛域名，或者域名套了 CDN 时，用 DNS 记录证明域名归属。支持三家 DNS 服务商：

| 服务商 | `cert_dns_provider` 填 | 密钥（`cert_dns_env`，多个用逗号分隔） |
| --- | --- | --- |
| Cloudflare | `cloudflare` | `CLOUDFLARE_DNS_API_TOKEN=令牌`（API Token，权限 Zone.DNS:Edit；不支持旧版 Global API Key） |
| 阿里云 DNS | `alidns` | `ALICLOUD_ACCESS_KEY=AccessKeyId` 和 `ALICLOUD_SECRET_KEY=AccessKeySecret` |
| 腾讯云 DNSPod | `tencentcloud` | `TENCENTCLOUD_SECRET_ID=SecretId` 和 `TENCENTCLOUD_SECRET_KEY=SecretKey` |

=== ":material-alpha-x-box: 在 Xboard 面板里设置（推荐）"

    节点管理 → 编辑节点 → **高级协议配置** → **TLS** 页签 → 证书模式选 **dns-01 (ACME)**，填证书域名、通知邮箱、DNS 提供商和密钥。保存后节点自动申请，不用改节点配置。

    acme.sh 和 lego 两种写法的变量名都能识别（例如 `CF_Token`、`Ali_Key`/`Ali_Secret`、`Tencent_SecretId`）。

=== ":material-file-cog: 在节点配置里设置"

    ```ini
    cert_mode=dns
    cert_domain=node.example.com
    cert_email=you@example.com
    cert_dns_provider=cloudflare
    cert_dns_env=CLOUDFLARE_DNS_API_TOKEN=你的令牌
    ```

    多个变量用逗号分隔：`cert_dns_env=ALICLOUD_ACCESS_KEY=xxx,ALICLOUD_SECRET_KEY=yyy`。

首次申请要等 DNS 记录生效，通常一两分钟，期间日志会提示；之后续期和重启都用缓存，秒开。证书存在配置文件同目录的 `certs` 文件夹，到期前 30 天自动续期并热加载。

!!! note "其他 DNS 服务商"
    用 acme.sh 等工具签发后，按下面「证书文件」的方式填路径即可。

## 证书文件

已经有证书时，直接指定路径。证书续期后自动重新加载。

```ini
cert_mode=file
cert_file=/etc/kaze/cert.pem
key_file=/etc/kaze/key.pem
```


## 自签证书

客户端需要打开「允许不安全」。部分客户端已不再提供这个选项。

```ini
cert_mode=self
cert_domain=node.example.com
```

## 前面有 nginx 卸载 TLS

TLS 由前面的 nginx / caddy 处理时，让 kaze 不再做 TLS:

```ini
force_close_ssl=true
```

!!! note
    对 Hysteria2 / TUIC（QUIC 内置 TLS）无效。
