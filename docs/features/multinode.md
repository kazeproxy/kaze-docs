# 一台机器多节点、多协议

一台落地机通常不止跑一个节点：同一个面板的几个节点、几种协议，甚至两个面板。kaze 的做法是**一个进程带一个面板的所有节点**，第二个面板再开一个实例。

| 场景 | 做法 | 进程数 |
| --- | --- | --- |
| 同一面板的多个节点 | `node_id=1,2,3` | 1 |
| 同一面板、不同协议 | 见下表，按面板不同略有差别 | 1 |
| 某个节点要单独设置 | `[node N]` 段落 | 1 |
| 两个面板 | 两个实例：`--name b` / `kaze -n b` | 每个面板 1 个 |

## 一个进程多个节点

同一台机器的多个节点写在一个 `node_id` 里，一个进程全带起来，各自监听面板里给它们设的端口：

```ini
node_id=1,2,3
```

## 一个进程多种协议

每种面板告诉后端「这个节点是什么协议」的方式不同，写法也略有差别，但都**不需要第二个进程**：

| 面板 | 写法 | 说明 |
| --- | --- | --- |
| Xboard | `node_id=1,2,3`，不填 `server_type` | 面板下发每个节点的协议，随便混 |
| V2board（v2node 节点） | `node_id=1,2,3`，不填 `server_type` | v2node 节点自带协议，随便混 |
| V2board（传统节点） | 每个协议的节点写一个 `[node N]` 段，段里填 `server_type` | 公共区的 `server_type` 给没有单独段落的节点用 |
| PPanel | `node_id=5` 加 `server_type=vless,trojan,hysteria` | 一个服务器 ID 开了几种协议就列几种 |

=== "Xboard"

    ```ini
    type=xboard
    node_id=1,2,3,4,5
    webapi_url=https://你的面板
    webapi_key=通讯密钥
    ```

=== "V2board 传统节点"

    ```ini
    type=v2board
    webapi_url=https://你的面板
    webapi_key=通讯密钥
    server_type=vless
    node_id=1,2          # 两个 VLESS 节点

    [node 7]
    server_type=trojan   # 节点 7 是 Trojan

    [node 9]
    server_type=hysteria # 节点 9 是 Hysteria2
    ```

=== "PPanel"

    ```ini
    type=ppanel
    node_id=5
    server_type=vless,trojan,hysteria
    webapi_url=https://你的面板
    webapi_key=通讯密钥
    ```

    几个服务器 ID 各开不同协议时，用 `[node N]` 段分别写 `server_type`，见[对接 PPanel](../panel/ppanel.md)。

## 单个节点的独立设置

写在 `[node 节点ID]` 下面的设置只对这个节点生效，其余节点仍用公共设置。有独立设置的节点不用再写进 `node_id`。

```ini
type=xboard
node_id=1,2
webapi_url=https://你的面板
webapi_key=通讯密钥
user_speed_limit=100

[node 5]
cert_mode=http
cert_domain=node5.example.com

[node 6]
user_speed_limit=50
forbidden_bit_torrent=false
```

!!! note
    除 `node_id`、授权、`log_level` 这几项外，其它设置都能在 `[node N]` 段里单独指定。

## 两个面板

一个进程只对接一个面板。同一台机器要给两个面板跑节点，再装一个实例（`--name b`），各有自己的配置、服务和授权码，升级时一起升级。写法见[一台机器多个面板](../panel/multi-panel.md)。

## 注意

- 同一台机器上所有节点（不管哪个面板）的端口不能重复，面板不会替你检查；撞了的节点会在日志里报错，改端口后自动恢复。
- 授权按面板算，节点数不限：同一面板的所有节点共用一个授权码。
