# 常见问题与排查

## 先看这三条

```bash
kaze check      # 对着面板检查配置,会直接指出问题
kaze status     # 看是否在运行
kaze log        # 看最近日志
```

## 面板里节点显示离线

在线状态靠节点定时上报，刚装好要等一个拉取周期（Xboard：系统配置 → 节点配置 → 节点拉取动作轮询间隔，默认 60 秒）。超过 2 分钟还离线，按顺序查：

1. `kaze status` 是不是 running；不是就看 `kaze log` 最后几行。
2. 日志有 `the address answered a web page, not the panel API` 或 `405`：`webapi_url` 填成了用户前端站或带了多余路径。要填面板**后台 API** 的域名（前端单独部署时看主题 `config.js` 里的 `server_url`），不要带 `/admin`。
3. 日志有 `401` / `403` / `invalid token`：`webapi_key` 和「系统配置 → 节点配置 → 通讯密钥」不一致，或 `node_id` 在面板里不存在。
4. 日志有 `x509` / `certificate`：面板本身用了自签或过期证书，先修面板。
5. 面板里改过通讯密钥、没 `kaze restart`。

`kaze check` 会一次把以上都查出来。机器模式下面板显示「节点数 0」：节点没分配给这台机器（编辑节点 → 选机器）、`machine_id` 填错、配置里多写了 `node_id`（机器模式不要写），或还没到下一个拉取周期。

## 启动不了

- **配置检查没过**：`kaze check` 的输出会说明哪一项不对。
- **对接信息错**：确认 `type`、`node_id`、`webapi_url`、`webapi_key` 和面板一致（`panel_url` / `panel_key` 是同一项的别名，一键安装和 Docker 写的就是它们）。V2board 还要 `server_type`。正确的样子：

    ```ini
    type=xboard
    node_id=1                              # 面板节点列表里的 ID
    webapi_url=https://panel.example.com   # 面板后台 API 地址（前端单独部署时是 config.js 里的 server_url）
    webapi_key=xxxxxxxxxxxx                # 系统配置 → 节点配置 → 通讯密钥
    ```

- **端口被占用**：日志里会写 `address already in use`。`ss -lntup | grep :443` 找到占用的进程，多半是没停掉的 XrayR / V2bX / nginx；停掉它或换端口。
- **证书申请失败**：日志搜 `acme` / `challenge`。常见原因：域名没解析到本机；80 端口被 nginx 占用（改用 [DNS 验证](features/cert.md)）；Cloudflare 小云朵开着（关掉或用 DNS 验证）；同一域名一周内申请超过 5 次被 Let's Encrypt 限流（等一周或换子域名）。

## 节点在线，用户连不上

1. 云厂商安全组放行了节点的**连接端口**吗？TCP 协议放 TCP，Hysteria2 / TUIC 放 **UDP**，端口跳跃放整段。
2. 用户在面板里是不是有效（套餐内、未到期、没超流量）？`kaze log` 里搜 `users synced` 看拉到了几个用户。
3. 免费版每节点只服务 100 人，第 101 个起排队；`kaze license` 看状态。
4. 客户端报 TLS 错误：`kaze log | grep -i cert`，自签证书要客户端允许不安全，或申请真证书。
5. REALITY 节点：伪装域名要能从服务器访问到（`curl -I https://伪装域名`）。
6. 订阅里 Shadowsocks 的插件显示 none / 连不上（Shadowrocket）：见[对接 Xboard → Shadow TLS](panel/xboard.md)。

## 面板里开的某个功能没生效

启动日志里会**明确写出**哪条规则 / 哪个字段不支持、为什么。`kaze log` 搜一下即可。kaze 不会静默失效。

## 设备数限制不准

节点套了 CDN 或中转时，要先让 kaze 拿到真实 IP，见[中转与 CDN 真实 IP](features/realip.md)。否则所有用户看起来来自同一个 IP。

## 一台机器怎么跑多个节点 / 多种协议 / 多个面板

同一面板的节点写在一个 `node_id` 里，一个进程全带起来，不同协议也可以混；第二个面板再装一个实例（`--name b`）。见[一台机器多节点、多协议](features/multinode.md)和[一台机器多个面板](panel/multi-panel.md)。

## 授权不生效 / 显示免费版

```bash
kaze license
```

输出会说明原因，每种状态的含义和处理见[授权说明 → 日志里的授权状态](license.md#日志里的授权状态)。最常见的两种：

- `LICENSE_REFUSED (mismatch)`：这个码已经绑在别的面板域名上——先在测试站试过、面板换了域名、或这台机器 `webapi_url` 写得和别的机器不一样。确认 `kaze config` 里的 `webapi_url` 域名就是你要绑的那个；是的话在机器人里换绑。
- `LICENSE_REFUSED (expired)`：到期了，在机器人里续费，6 小时内自动恢复（`kaze restart` 立即）。

## Hysteria2 / TUIC 弱网速度慢

Hysteria2 客户端填了带宽时按该速率发送（Brutal）；没填时用 BBR 自动探测线路速度。TUIC 在面板编辑节点时把「拥塞控制」选为 `bbr` 即可（Xboard：节点管理 → 编辑节点 → 高级协议配置 → TUIC；V2board / PPanel 在节点的协议设置里）。

弱网下仍然慢，可以让用户在客户端填上真实带宽。

## 游戏、语音不通

屏蔽 BT、屏蔽内网不影响这些。如果分流规则把这类流量指向 HTTP 代理出口：HTTP 代理不转发 UDP，UDP 会被丢弃（不会直连，以免暴露节点 IP），纯 UDP 的游戏和语音就会不通，改用其它类型的出口即可。落地机是 Shadowsocks 时，确认落地机开了 UDP。见[分流与出口](features/routing.md)。

## 更新 geo 数据

```bash
kaze geo
```

## 还是解决不了

带上 `kaze log` 的输出和 `kaze check` 的结果，用机器人主菜单的「联系客服」或到频道 [@kaze_core](https://t.me/kaze_core) 评论区反馈。日志里不含密钥明文（会打码），可以直接贴。
