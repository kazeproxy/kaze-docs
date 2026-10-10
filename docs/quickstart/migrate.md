# 从 XrayR / V2bX / Xray 迁移

面板里的节点**不用重建**，节点 ID 不变，用户订阅不用动：kaze 读的是同一份面板节点配置。整个过程每台机器几分钟。

## 顺序

1. 先停掉旧后端，否则端口冲突：`systemctl disable --now XrayR`（或 `V2bX`、`xray`）。日志出现 `address already in use` 就是旧进程还在。
2. 按[一键安装](install.md)装 kaze，`--node-id` 只填**这台机器**的节点 ID（填了别台的 ID 会在本机也监听那个端口）。
3. `kaze log` 看到 `listening` 和 `users synced`，面板里节点变在线。
4. 全部机器换完后再删旧后端的配置。

## 配置对照

| XrayR / V2bX `config.yml` | kaze `kaze.conf` |
| --- | --- |
| `ApiHost` | `webapi_url` |
| `ApiKey` | `webapi_key` |
| `NodeID` | `node_id` |
| `NodeType`（V2board 必填） | `server_type` |
| `CertMode` / `CertDomain` / `CertFile` / `KeyFile` | `cert_mode` / `cert_domain` / `cert_file` / `key_file` |
| `DNSEnv`（DNS 验证密钥） | `cert_dns_provider` + `cert_dns_env` |
| `SpeedLimit` | `user_speed_limit` |
| `DeviceLimit` | `user_device_limit` |
| `EnableProxyProtocol` | `proxy_protocol` |
| `UpdatePeriodic` | `check_interval` |
| 屏蔽规则 / `rulelist` | `block_list_file`（格式见[审计与屏蔽](../features/audit.md)） |

Xboard 节点什么都不用填，面板下发。

## 证书

已经申请好的证书直接复制过来用：把 XrayR 的 `/etc/XrayR/cert/` 里的两个文件放到 `/etc/kaze/`，填 `cert_mode=file`、`cert_file=/etc/kaze/cert.pem`、`key_file=/etc/kaze/key.pem`。续期由 kaze 接管（`cert_mode=http` 或 `dns`）。

## 面板里的自定义出站 / 路由

Xray 格式的 JSON 原样可用，sing-box 格式也认；不支持的字段启动日志逐条提示并忽略。见[路由与自定义出站](../panel/routing.md)。

## 需要先改的节点

下面这些 kaze 不支持，迁移前要在面板改协议或传输，否则该节点在 kaze 上不启动（日志有提示）：ShadowsocksR、Snell、NaiveProxy、mKCP、`xtls-rprx-direct`、simple-obfs 的 `tls` 模式、v2ray-plugin、Shadowsocks 2022 的 chacha20 算法。完整对照见[面板设置对照](../panel/compat.md)。
