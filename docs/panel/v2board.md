# 对接 V2board

支持 wyx2685 维护的 V2board 的**传统节点**。如果你在面板里建的是 v2node 节点，看 [对接 V2board（v2node 节点）](v2node.md)。

## 节点怎么配

V2board 的接口不会告诉节点协议，所以**必须填 `server_type`**：

```ini
type=v2board
server_type=vless
node_id=1
webapi_url=https://你的面板
webapi_key=通讯密钥
```

同一个 `server_type` 的多个节点可以写在一个 `node_id` 里：`node_id=1,2,3`。不同协议的节点各写一个 `[node N]` 段、段里填自己的 `server_type`，仍是一个进程，见[一台机器多节点、多协议](../features/multinode.md)。

!!! warning "V2board 的节点 ID 按协议各自编号"
    VLESS 的 1 号和 Trojan 的 1 号是两个节点。一台机器要同时跑 ID 相同、协议不同的两个节点时，`[node N]` 段分不开它们——第二个请用第二个实例：`--name b --type v2board --server-type trojan --node-id 1`，见[一台机器多个面板](multi-panel.md)（同样适用于同一面板的第二个实例）。

## server_type 可选值

`trojan`、`ss`、`vless`、`vmess`、`hysteria`、`tuic`、`anytls`、`mieru`、`socks`、`http`。其中 `v2ray` 等同 `vmess`,`hysteria` 指 Hysteria2。

## Shadowsocks 混淆

V2board 的 Shadowsocks 节点如果开了 simple-obfs 的 HTTP 混淆，kaze 会自动识别并提供，不用额外配置。
