# 对接 V2board（v2node 节点）

wyx2685 版 V2board 里有两类节点：

- **传统节点**:VMess、VLESS、Trojan 等各自一个页面。按 [对接 V2board](v2board.md) 配置。
- **v2node 节点**：一个统一页面，在里面选协议，支持 XHTTP、ECH、证书模式等更多选项。按本页配置。

!!! tip "从 v2node 后端换成 kaze"
    面板里的节点**不用重建**，只要节点机器上把后端换成 kaze,`type` 写 `v2node` 即可。

## 用面板的一键安装命令 { #one-click }

V2board 的 v2node 节点编辑页底部有「一键安装指令」，形如：

```bash
wget -N https://raw.githubusercontent.com/.../v2node/master/script/install.sh && bash install.sh --api-host 'https://api.example.com' --node-id 12 --api-key 'abcd1234'
```

kaze 的安装脚本认得同样的参数。**把命令里脚本的地址换成 kaze 的**，后面的参数原样保留：

```bash
wget -N https://github.com/kazeproxy/kaze-release/raw/main/install.sh && bash install.sh --api-host 'https://api.example.com' --node-id 12 --api-key 'abcd1234'
```

脚本看到 `--api-host` / `--api-key` 就按 v2node 对接（写入 `type=v2node`）。如果你的节点其实是传统节点，改用[对接 V2board](v2board.md)里带 `--type v2board --server-type` 的命令。

| 参数 | 含义 | 写入配置 |
| --- | --- | --- |
| `--api-host` | 面板「系统配置 → 节点 → 节点对接API地址」，没填时是站点网址 | `webapi_url` |
| `--node-id` | 节点 ID | `node_id` |
| `--api-key` | 面板的通讯密钥 | `webapi_key` |

需要填授权码或其它设置时，接在命令末尾，例如 `--license KZ-XXXX-XXXX-XXXX` 或 `listen=0.0.0.0`。同一台机器跑多个节点时，装好后用 `kaze config node_id=12,13` 改成多个 ID 再 `kaze restart`。

## 节点怎么配

v2node 节点会告诉后端自己是什么协议，**不用填 `server_type`**，不同协议的节点可以写在同一个 `node_id` 里。

=== "一键安装"

    ```bash
    bash <(curl -fsSL https://github.com/kazeproxy/kaze-release/raw/main/install.sh) install \
      --type v2node \
      --node-id 1,2,3 \
      --panel-url https://你的面板 \
      --panel-key 通讯密钥
    ```

=== "配置文件"

    ```ini
    type=v2node
    node_id=1,2,3
    webapi_url=https://你的面板
    webapi_key=通讯密钥
    ```

=== "Docker Compose"

    ```yaml
    environment:
      type: v2node
      node_id: "1,2,3"
      panel_url: https://你的面板
      panel_key: 通讯密钥
    ```

## 面板里的设置

| v2node 设置 | 在 kaze 上 |
| --- | --- |
| 协议:shadowsocks / vmess / vless / trojan / tuic / hysteria2 / anytls | :material-check: |
| 传输:tcp / ws / grpc / http / httpupgrade / xhttp | :material-check: Shadowsocks 选 http 时是 HTTP 混淆（simple-obfs），其它协议是 HTTP/2 |
| TLS、REALITY、Vision 流控 | :material-check: |
| 监听 IP、CDN 真实 IP 请求头（trusted_x_forwarded_for） | :material-check: |
| 接收 PROXY protocol | :material-check: |
| 证书模式 self / http | :material-check: 「证书文件 / 密钥文件」路径上已有文件时直接使用 |
| 证书模式 none（无证书，关闭 TLS） | :material-check: 节点不加密监听，由前面的 Nginx / CDN 处理 TLS；节点本机配了 `cert_file` 时仍用本机证书。TUIC、Hysteria2 必须有 TLS，会改用自签证书并在日志提示 |
| 证书模式 remote（面板生成证书） | :material-check: 直接使用面板下发的证书 |
| Hysteria2 混淆、带宽 | :material-check: 「上行带宽」是节点发给用户的速度，「下行带宽」是用户上传的上限，与 V2board 订阅给客户端的一致 |
| 路由规则 | :material-check: 格式见 [面板路由与自定义出站](routing.md#v2board-v2node) |
| 证书模式 dns | :material-check: Cloudflare、阿里云、腾讯云 DNSPod |
| ECH、VLESS 加密（mlkem） | :material-alert-circle: 暂不支持，日志会提示 |
