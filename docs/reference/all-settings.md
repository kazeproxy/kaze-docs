# 配置项速查表

所有设置都写在 `kaze.conf` 里，每行一个，格式是 `配置项=值`；也可以用**同名环境变量**传入（Docker 常用，写成 `配置项: 值`）。面板下发的协议、端口、传输、用户等不在这里，由面板控制。

说明里的「例」可以直接照抄，把值换成你自己的。没写进配置的项就用「默认」。

=== "kaze.conf（一键安装）"

    ```ini
    type=xboard
    node_id=1
    webapi_url=https://panel.example.com
    webapi_key=你的通讯密钥
    ```

    改完执行 `kaze restart` 生效；也可以用 `kaze config 配置项=值` 修改。

=== "环境变量（Docker）"

    ```yaml
    environment:
      type: xboard
      node_id: "1"
      panel_url: https://panel.example.com
      panel_key: 你的通讯密钥
    ```

    改完执行 `docker compose up -d` 生效。

## 对接（必填）

| 配置项 | 默认 | 说明 |
| --- | --- | --- |
| `type` | xboard | 面板类型：`xboard`、`v2board`、`v2node`（wyx2685 的 v2node 节点）、`ppanel`；`local` = 不接面板，节点写在本文件里（见[不接面板，直接搭节点](../quickstart/local.md)）<br>例：`type=xboard` |
| `server_type` | — | 节点协议。Xboard、v2node 可不填；V2board、PPanel 必填。PPanel 可写多个：`server_type=vless,trojan,hysteria`<br>例：`server_type=vless`。可选值：`trojan` `ss`（或 `shadowsocks`） `vless` `vmess` `hysteria` `tuic` `anytls` `mieru` `socks` `http`，见[对接 V2board](../panel/v2board.md) |
| `node_id` | — | 面板里的节点 ID（PPanel 是服务器 ID），多个用逗号分隔<br>例：`node_id=1` 或 `node_id=1,2,3` |
| `webapi_url` | — | 面板地址，带 `https://`(也可写 `panel_url`)<br>例：`webapi_url=https://panel.example.com` |
| `webapi_key` | — | 通讯密钥。Xboard 在后台「系统配置 → 节点配置 → 通讯密钥」，V2board 在「系统配置 → 节点 → 通讯密钥」（也可写 `panel_key`）<br>例：`webapi_key=xxxxxxxxxxxx`；PPanel 在「系统设置 → 节点配置 → Secret Key」 |
| `machine_id` | — | Xboard 机器模式的机器 ID，填了就不需要 `node_id`，见[机器模式](../panel/xboard-machine.md)<br>例：`machine_id=3` |
| `port` `users` `cipher` `server_key` `tls` `server_name` `network` `path` `service_name` `plugin` `plugin_opts` `obfs_password` | — | 仅 `type=local`（不接面板）时用，见[不接面板，直接搭节点](../quickstart/local.md) |
| `machine_token` | — | 机器令牌，在面板添加机器时给出<br>例：`machine_token=xxxxxxxx` |

## 监听与日志

日志的用法见[日志](../features/log.md)。

| 配置项 | 默认 | 说明 |
| --- | --- | --- |
| `listen` | 面板下发 | 监听的 IP。`0.0.0.0` 同时监听 IPv4 和 IPv6;只想监听某个 IP 就写那个 IP<br>例：`listen=0.0.0.0` |
| `log_level` | info | 日志级别：`debug`（排错用，很多）/ `info` / `warn` / `error`（只记错误）<br>例：`log_level=warn` |
| `memory_limit` | 0 | 内存软上限，单位 MB。0 = 自动（机器内存或容器限制的 70%）；`off` = 不限制<br>例：`memory_limit=512` |
| `log_file_dir` | — | 把日志另存到这个目录。留空只输出到系统日志（`kaze log` 查看）<br>例：`log_file_dir=/var/log/kaze` |
| `log_file_retention_days` | 7 | 日志文件保留天数,0 = 永久保留<br>例：`log_file_retention_days=30` |

## 同步间隔

| 配置项 | 默认 | 说明 |
| --- | --- | --- |
| `check_interval` | 0 | 从面板拉取配置和用户的间隔，单位秒。0 = 用面板设置的值，Xboard 面板默认 60 秒（Xboard：系统配置 → 节点配置 → 节点拉取动作轮询间隔；V2board：系统配置 → 节点 → 拉取间隔；PPanel：系统设置 → 节点配置）<br>例：`check_interval=60` |
| `submit_interval` | 0 | 向面板上报流量的间隔，单位秒。0 = 用面板设置的值（面板里的「推送间隔」，位置同上）<br>例：`submit_interval=60` |
| `submit_traffic_min_traffic` | 0 | 用户本次流量少于这么多 KB 就攒到下次一起上报，减少面板压力。0 = 用面板设置的值<br>例：`submit_traffic_min_traffic=10` |
| `submit_alive_ip_min_traffic` | 0 | 用户本次流量少于这么多 KB 就不算在线（不占设备数）。0 = 用面板设置的值<br>例：`submit_alive_ip_min_traffic=10` |

## 超时

| 配置项 | 默认 | 说明 |
| --- | --- | --- |
| `tcp_timeout` | 5 | TCP 连接空闲多久断开，单位分钟<br>例：`tcp_timeout=10` |
| `udp_timeout` | 2 | UDP 会话空闲多久结束，单位分钟<br>例：`udp_timeout=3` |

## 用户限制

面板套餐里设置的限速、设备数优先；这里的设置用于面板没设置的情况，或给所有用户加一个上限。详见[用户限制与风控](../features/limits.md)。

| 配置项 | 默认 | 说明 |
| --- | --- | --- |
| `user_speed_limit` | 0 | 每个用户限速上限，单位 Mbps。面板值更低时用面板值。0 = 只用面板值<br>例：`user_speed_limit=100` |
| `node_speed_limit` | 0 | 整个节点的总带宽上限，单位 Mbps。0 = 不限<br>例：`node_speed_limit=1000` |
| `user_device_limit` | 0 | 面板没设设备数的用户，最多同时几个 IP 在线。0 = 不限（也可写 `user_conn_limit`）<br>例：`user_device_limit=3` |
| `device_ipv6_prefix` | 64 | 同一 IPv6 网段算一台设备：手机用 IPv6 时地址会在 /64 内不断变化，按单个地址算会误判超设备数。128 = 按单个地址算；IPv4 始终按单个地址<br>例：`device_ipv6_prefix=56` |
| `user_tcp_limit` | 0 | 每个用户最多同时多少个连接。0 = 不限<br>例：`user_tcp_limit=256` |

## 动态限速

说明见[用户限制与风控](../features/limits.md)。

一段时间内持续高速下载的用户，自动临时降速一段时间，防止少数人占满带宽。

| 配置项 | 默认 | 说明 |
| --- | --- | --- |
| `dy_limit_enable` | false | 总开关<br>例：`dy_limit_enable=true` |
| `dy_limit_duration` | 全天 | 只在这些时间段生效，可以写多段、可以跨零点。留空 = 全天<br>例：`dy_limit_duration=20:00-02:00,10:00-14:00` |
| `dy_limit_trigger_time` | 60 | 统计多少秒内的平均速度<br>例：`dy_limit_trigger_time=60` |
| `dy_limit_trigger_speed` | 100 | 平均速度超过这么多 Mbps 就触发<br>例：`dy_limit_trigger_speed=100` |
| `dy_limit_speed` | 30 | 触发后限速到这么多 Mbps<br>例：`dy_limit_speed=30` |
| `dy_limit_time` | 600 | 限速持续多少秒<br>例：`dy_limit_time=600` |
| `dy_limit_white_user_id` | — | 不受动态限速的用户 ID，逗号分隔<br>例：`dy_limit_white_user_id=1,25,108` |

## 审计与屏蔽

规则文件的写法见[审计与屏蔽](../features/audit.md)。

| 配置项 | 默认 | 说明 |
| --- | --- | --- |
| `forbidden_bit_torrent` | true | 屏蔽 BT 下载。`false` = 允许<br>例：`forbidden_bit_torrent=false` |
| `forbidden_private_ip` | true | 禁止访问内网地址（防止用户访问节点所在的内网）<br>例：`forbidden_private_ip=true` |
| `forbidden_ports` | — | 禁止访问的目标端口，可写区间。屏蔽 25/465/587 可防止被用来发垃圾邮件<br>例：`forbidden_ports=25,465,587,1000-2000` |
| `block_list_file` | — | 本地审计规则文件<br>例：`block_list_file=/etc/kaze/block.txt` |
| `block_list_url` | — | 远程审计规则地址，定时自动更新<br>例：`block_list_url=https://example.com/block.txt` |
| `block_list_update_interval` | 60 | 远程规则的更新间隔，单位分钟<br>例：`block_list_update_interval=120` |
| `white_list_file` | — | 白名单文件，命中的不受审计规则限制<br>例：`white_list_file=/etc/kaze/white.txt` |
| `geosite_file` | 同目录 geosite.dat | geosite 数据文件，规则里用 `geosite:` 时需要。`kaze geo` 可一键下载<br>例：`geosite_file=/etc/kaze/geosite.dat` |
| `geo_update_interval` | 7 | geo 数据文件超过这么多天就自动重新下载，只更新节点上已有的文件。0 = 不自动更新<br>例：`geo_update_interval=3` |
| `geoip_file` | 同目录 geoip.dat | geoip 数据文件，规则里用 `geoip:` 时需要<br>例：`geoip_file=/etc/kaze/geoip.dat` |

## 路由与出口

| 配置项 | 默认 | 说明 |
| --- | --- | --- |
| `routes_file` | — | 本机路由文件（出口、出口组、规则），写法见[分流与出口](../features/routing.md)。改了保存即生效，不用重启<br>例：`routes_file=/etc/kaze/routes.conf` |
| `routes_url` | — | 从网上定时拉取路由文件，代替 `routes_file`，多台节点共用一份<br>例：`routes_url=https://example.com/kaze/routes.conf` |
| `routes_update_interval` | 60 | `routes_url` 的更新间隔，单位分钟<br>例：`routes_update_interval=30` |
| `auto_out_ip` | false | 源进源出：多 IP 机器上，从哪个 IP 进来就从哪个 IP 出去<br>例：`auto_out_ip=true` |

## DNS

详见 [DNS 解析](../features/dns.md)。

| 配置项 | 默认 | 说明 |
| --- | --- | --- |
| `default_dns` | 系统 DNS | DNS 服务器，多个用逗号分隔，前一个失败用下一个。写法：`8.8.8.8`（普通 UDP）、`tcp://8.8.8.8`、`tls://1.1.1.1`(DoT)、`https://1.1.1.1/dns-query`(DoH)<br>例：`default_dns=https://1.1.1.1/dns-query,8.8.8.8` |
| `dns_rules_file` | — | 按域名分流的 DNS 规则文件<br>例：`dns_rules_file=/etc/kaze/dns.conf` |
| `dns_cache_time` | 10 | DNS 结果缓存时间，单位分钟<br>例：`dns_cache_time=30` |
| `dns_strategy` | ipv4_first | `ipv4_first`（优先 IPv4）/ `ipv4_only`（只用 IPv4,机器没有 IPv6 时用）/ `ipv6_first` / `ipv6_only`<br>例：`dns_strategy=ipv4_only` |

## 端口跳跃

| 配置项 | 默认 | 说明 |
| --- | --- | --- |
| `hop_ports` | — | Hysteria2 / TUIC 端口跳跃的端口段，和面板「连接端口」一致。多段：`20000-25000,30000-35000`。见[端口跳跃](../features/port-hopping.md)<br>例：`hop_ports=20000-30000` |

## 证书

一般不用在节点上设置：证书模式、域名在面板里填即可（面板里的 `none` / `content` / `remote` 模式只在面板选，节点配置不接受）（Xboard：节点管理 → 编辑节点 → 高级协议配置 → TLS；其它面板见[面板设置对照](../panel/compat.md)）。这里的设置用于面板没有提供的情况，详见 [TLS 证书](../features/cert.md)。

| 配置项 | 默认 | 说明 |
| --- | --- | --- |
| `cert_mode` | — | `http` = 自动申请（需开放 80 端口）/ `dns` = DNS 验证自动申请 / `file` = 用自己的证书文件 / `self` = 自签<br>例：`cert_mode=http` |
| `cert_domain` | — | 证书域名，要解析到这台机器<br>例：`cert_domain=node1.example.com` |
| `cert_email` | — | 自动申请时登记的邮箱，用于证书到期提醒<br>例：`cert_email=you@example.com` |
| `cert_dns_provider` | — | DNS 验证的服务商：`cloudflare` / `alidns`（阿里云）/ `tencentcloud`（腾讯云 DNSPod）<br>例：`cert_dns_provider=cloudflare` |
| `cert_dns_env` | — | DNS 服务商的密钥。多个用逗号分隔，如 `ALICLOUD_ACCESS_KEY=xxx,ALICLOUD_SECRET_KEY=yyy`<br>例：`cert_dns_env=CLOUDFLARE_DNS_API_TOKEN=你的令牌` |
| `cert_file` | — | 证书文件路径（`cert_mode=file` 时），文件更新后自动重新加载<br>例：`cert_file=/etc/kaze/cert.pem` |
| `key_file` | — | 证书私钥文件路径<br>例：`key_file=/etc/kaze/key.pem` |
| `force_close_ssl` | false | 前面有 nginx 等负责 TLS 时打开，kaze 不再处理 TLS<br>例：`force_close_ssl=true` |
| `trojan_fallback` | — | 不是 Trojan / AnyTLS 的连接转给这个地址，让端口看起来像普通网站<br>例：`trojan_fallback=127.0.0.1:80` |

## 中转与真实 IP

详见[真实 IP](../features/realip.md)。

| 配置项 | 默认 | 说明 |
| --- | --- | --- |
| `trusted_x_forwarded_for` | — | 套了 CDN 或反代时，从这个请求头读用户真实 IP。Cloudflare 用 `CF-Connecting-IP`，其他通常是 `X-Forwarded-For`，多个用逗号分隔<br>例：`trusted_x_forwarded_for=CF-Connecting-IP` |
| `proxy_protocol` | false | 前面有中转（HAProxy、nginx stream、gost 等）并发送 PROXY 头时打开，读取用户真实 IP<br>例：`proxy_protocol=true` |
| `force_proxy_protocol` | false | 只接受带 PROXY 头的连接，其余拒绝（防止绕过中转直连）<br>例：`force_proxy_protocol=true` |

## 防爆破

说明见[用户限制与风控](../features/limits.md)。

有人反复用错误的密码连接时，临时封禁他的 IP。

| 配置项 | 默认 | 说明 |
| --- | --- | --- |
| `invalid_access_enable` | false | 总开关<br>例：`invalid_access_enable=true` |
| `invalid_access_count` | 30 | 统计窗口内失败多少次就封禁<br>例：`invalid_access_count=30` |
| `invalid_access_duration` | 60 | 统计窗口，单位秒<br>例：`invalid_access_duration=60` |
| `invalid_access_forbidden_time` | 600 | 封禁多久，单位秒<br>例：`invalid_access_forbidden_time=3600` |

## 授权

详见[授权](../license.md)。不填就是免费版（每节点 100 个有效用户，节点收到的名单超过 300 人则拒绝服务）；授权在 Telegram 机器人购买，见[首页](../index.md#购买授权)。

| 配置项 | 默认 | 说明 |
| --- | --- | --- |
| `license_key` | — | 授权码<br>例：`license_key=KZ-XXXX-XXXX-XXXX` |
| `license_file` | — | 或者把授权码放在文件里<br>例：`license_file=/etc/kaze/license.txt` |
| `license_server` | `https://auth.kazecore.dev` | 授权签到地址，一般不用改<br>例：`license_server=https://auth.kazecore.dev` |

## 单节点独立设置

一个进程跑多个节点时，用 `[node 节点ID]` 段落给某个节点单独设置，只对它生效：

```ini
type=xboard
node_id=1
webapi_url=https://panel.example.com
webapi_key=你的通讯密钥

[node 5]
cert_mode=http
cert_domain=node5.example.com
forbidden_bit_torrent=false
```
