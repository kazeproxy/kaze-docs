# 对接 Xboard

## 节点怎么配

Xboard 对接最省事：节点这边**不用填协议**，面板会告诉每个节点 ID 是什么协议。于是一个进程可以同时跑不同协议的节点。

```ini
type=xboard
node_id=1,2,3
webapi_url=https://你的面板
webapi_key=通讯密钥
```

`node_id` 里可以混着写不同协议的节点，kaze 会各自按面板下发的协议启动。

## 面板里要设什么

端口、协议、传输、TLS、REALITY、流控、用户、路由规则全部在面板「节点管理 → 添加节点」里设置，kaze 自动同步，节点上不用再配。下面两个是最常用的例子，照着填就能用（字段名称以你的面板版本为准，括号里是字段英文名）。

=== "VLESS + REALITY（推荐，不需要域名和证书）"

    | 面板字段 | 填什么 | 示例 |
    | --- | --- | --- |
    | 节点类型 | 新建节点时第一个选项，选 VLESS | `vless` |
    | 节点名称 | 用户在客户端里看到的名字 | `香港 01` |
    | 权限组 | 哪些套餐的用户能用这个节点 | 选你的套餐组 |
    | 节点地址（host） | 用户连接的地址：服务器 IP，或解析到它的域名 | `1.2.3.4` |
    | 连接端口（port） | 用户连接的端口 | `443` |
    | 服务端口（server_port） | kaze 实际监听的端口，一般和连接端口相同 | `443` |
    | 倍率（rate） | 流量计费倍率 | `1` |
    | 传输方式 | | `tcp` |
    | 安全性 | | `REALITY` |
    | 流控（flow） | | `xtls-rprx-vision` |
    | 伪装域名（server_name） | 一个支持 TLS 1.3 的大网站，不是你的用户的访问会被转给它 | `www.microsoft.com` |
    | 伪装站点端口 | 伪装域名的端口 | `443` |
    | 私钥（Private key）/ 公钥（Public key） | 点私钥输入框右边的 :material-key: **钥匙按钮**，私钥和公钥会一起生成；老版本没有按钮时，装好 kaze 后在服务器上跑 `kaze -reality-keypair`，把输出的两串分别填进来 | 一串 43 个字符 |
    | Short ID | 点输入框右边的 :material-refresh: **刷新按钮**生成，或自己填 0~16 位十六进制 | `6af819e1` |

=== "Trojan + TLS（需要域名）"

    先把一个域名（如 `node1.example.com`）解析到服务器 IP。

    | 面板字段 | 填什么 | 示例 |
    | --- | --- | --- |
    | 节点名称 | | `美国 01` |
    | 节点地址（host） | 你的域名 | `node1.example.com` |
    | 连接端口 / 服务端口 | | `443` / `443` |
    | 服务器名称 SNI(server_name) | 和节点地址相同 | `node1.example.com` |
    | 允许不安全（allow_insecure） | 证书正常时不要开 | 关 |

    证书由 kaze 自动申请（要求服务器 80 端口能从公网访问），也可以在面板里选证书模式（节点管理 → 编辑节点 → 高级协议配置 → TLS → 证书模式），详见 [TLS 证书](../features/cert.md)。

=== "Shadowsocks 2022 + Shadow TLS（不需要域名，有证书也行）"

    | 面板字段 | 填什么 | 示例 |
    | --- | --- | --- |
    | 加密算法 | | `2022-blake3-aes-128-gcm` |
    | 连接端口 / 服务端口 | **填同一个** | `12843` / `12843` |
    | 插件 | Shadow TLS | |
    | 插件选项 | 伪装站、自定义密码、固定 `version=3` | `host=cloud.tencent.com;password=随便一串;version=3` |

    kaze 内置 Shadow TLS v3，**不用在服务器上另外跑 shadow-tls 进程，也不用多占一个端口**（其它后端通常要在 Shadowsocks 前面再挂一个 shadow-tls 进程并打通两个端口）。伪装站选一个 TLS 1.3、能正常 443 访问的大站。

    !!! warning "Shadowrocket 用户要走 Clash Meta 格式的订阅"
        Xboard 给 Shadowrocket 的 `ss://` 一行式订阅里，Shadow TLS 插件 Shadowrocket 解析不出来（显示插件 none，连不上）。让 Shadowrocket 拿 Clash Meta 格式即可：`app/Protocols/ClashMeta.php` 的 `$flags` 加上 `'shadowrocket'`，`app/Protocols/Shadowrocket.php` 的 `$flags` 改成 `[]`，然后 `php artisan optimize:clear`。不要改到 `Clash.php`（老 Clash 不支持 2022 加密，会把节点过滤掉）。

!!! tip "连接端口和服务端口有什么区别"
    **连接端口**是写进用户订阅、客户端去连的端口；**服务端口**是 kaze 在服务器上监听的端口。直连时两个填一样的。前面有中转（比如中转机 8443 转发到落地机 443）时，连接端口填中转的 `8443`，服务端口填 `443`。Hysteria2 / TUIC 的端口跳跃也用到这个区别，见[端口跳跃](../features/port-hopping.md)。用 kaze 自己做中转（入口机跑节点、把流量交给落地机）见[中转机转发到落地机](routing.md#中转机转发到落地机)。

## 新旧两套接口

Xboard 有新旧两版节点接口。kaze 优先用新版（`/api/v2/server/`)，老面板会自动退回旧版（`/api/v1/server/UniProxy/`），不用你操心。

## 面板里开了但 kaze 不支持的功能

如果面板给节点配了 kaze 还不支持的东西（例如某些自定义出站类型），**启动日志里会明确写出来是哪一条、为什么没生效**，不会静默失效。看日志即可发现。

## 机器模式

想在面板里给机器分配节点、自动部署，看 [Xboard 机器模式](xboard-machine.md)。

## 实时推送（WebSocket）

Xboard 后台开启 WebSocket（系统配置 → 节点配置 → **启用 WebSocket 通信**，并在面板服务器上运行 `php artisan ws-server`）后，kaze 会自动连上推送通道，无需任何配置：

- 改节点配置、增删用户、用户到期或封禁，**几秒内生效**，不用等下一个拉取周期；被删用户的连接立即断开；
- 各节点的在线设备实时汇总，跨节点设备数限制更准确。

没开推送时一切照旧，kaze 每 5 分钟检查一次面板是否开启。即使连上推送，定时拉取也继续保留，作为兜底。

## 面板里看到的节点状态

新版 Xboard 的节点列表会显示 kaze 上报的负载（CPU、内存）和运行指标：运行时长、活跃连接数、实时网速、活跃用户数、面板请求成功/失败次数、推送通道状态等。新版面板上，流量、在线、负载、指标合并成一次请求上报，老版面板自动退回分开上报。
