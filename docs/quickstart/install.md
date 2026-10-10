# 一键安装

适用于带 systemd 的 Linux，也就是绝大多数 VPS。脚本会下载程序、写好配置、注册系统服务，并在启动前**先对着面板检查一遍配置**。

!!! info "开始前"
    请先完成 [准备工作](prepare.md)，手里要有 **节点 ID**、**面板地址**、**通讯密钥**。

## 安装

=== ":material-alpha-x-box: 对接 Xboard"

    Xboard 会告诉节点每个 ID 是什么协议，**不用填协议**。

    ```bash
    bash <(curl -fsSL https://github.com/kazeproxy/kaze-release/raw/main/install.sh) install \
      --type xboard \
      --node-id 1 \
      --panel-url https://你的面板 \
      --panel-key 通讯密钥
    ```

=== ":material-alpha-v-box: 对接 V2board"

    V2board 不会告诉节点协议，**必须用 `--server-type` 指定**。

    ```bash
    bash <(curl -fsSL https://github.com/kazeproxy/kaze-release/raw/main/install.sh) install \
      --type v2board \
      --server-type vless \
      --node-id 1 \
      --panel-url https://你的面板 \
      --panel-key 通讯密钥
    ```

=== ":material-alpha-p-box: 对接 PPanel"

    `--node-id` 填 PPanel 的服务器 ID,`--server-type` 写要跑的协议，多个用逗号分隔。

    ```bash
    bash <(curl -fsSL https://github.com/kazeproxy/kaze-release/raw/main/install.sh) install \
      --type ppanel \
      --server-type vless,trojan \
      --node-id 5 \
      --panel-url https://你的面板 \
      --panel-key 通讯密钥
    ```

看到 `done.` 和 `active (running)` 就是装好了。填了授权码时末尾还会打印一行授权判定（`LICENSE_OK ...` 或拒绝原因）。

## 确认在服务

```bash
kaze log
```

正常会看到这三行（顺序可能不同）：

```text
listening node=1 protocol=vless addr=[::]:443 transport=tcp+reality   ← 节点已监听
users synced node=1 via=pull total=12 added=12 removed=0              ← 已从面板拉到用户
licence node=1 status="free edition, up to 100 users (no licence)"    ← 授权状态（或 LICENSE_OK）
```

然后到面板「节点管理」看这个节点变成在线（第一次上报最长等一个拉取周期，默认 60 秒）。客户端连不上先确认安全组放行了连接端口（TCP；Hysteria2 / TUIC 是 UDP）。排查见[常见问题](../faq.md)。

## 参数说明

| 参数 | 必填 | 说明 |
| --- | :---: | --- |
| `--type` | | 面板类型：`xboard`（默认）、`v2board`、`v2node` 或 `ppanel`；`local` 为不接面板（见[不接面板，直接搭节点](local.md)） |
| `--node-id` | :material-check: | 节点 ID。一台机器多个节点用逗号分隔：`1,2,3` |
| `--panel-url` | :material-check: | 面板地址 |
| `--panel-key` | :material-check: | 通讯密钥 |
| `--server-type` | V2board 传统节点 / PPanel 必填；Xboard、v2node 不用 | 节点协议：`trojan` `ss` `vless` `vmess` `hysteria` `tuic` `anytls` `mieru` `socks` `http`；PPanel 可写多个 |
| `--license` | | 授权码 `KZ-XXXX-XXXX-XXXX`，不填是免费版 |
| `key=value` | | 任意其它设置，追加在命令末尾，例如 `user_speed_limit=100` |
| `--name` | | 同一台机器的第二个实例（第二个面板），见[一台机器多个面板](../panel/multi-panel.md) |
| `--mode machine --panel --token --machine-id` | | Xboard 机器模式，照抄面板给的命令即可，见[机器模式](../panel/xboard-machine.md) |
| `--api-host --api-key` | | V2board v2node 节点的一键命令，见[v2node](../panel/v2node.md) |

`install` 这个词可以省略（面板给出的命令就没有）；`bash <(curl -fsSL 地址)`、`curl -fsSL 地址 \| sudo bash -s --`、先 `wget` 再 `bash install.sh` 三种跑法等价，`github.com/kazeproxy/kaze-release/raw/main/install.sh` 和 `raw.githubusercontent.com/kazeproxy/kaze-release/main/install.sh` 是同一个文件。

!!! example "一台机器跑多个节点，并顺带设置限速"
    ```bash
    bash <(curl -fsSL https://github.com/kazeproxy/kaze-release/raw/main/install.sh) install \
      --type xboard --node-id 1,2,3 \
      --panel-url https://你的面板 --panel-key 通讯密钥 \
      user_speed_limit=200 forbidden_ports=25,465,587
    ```

## 检查没通过怎么办

安装脚本会在启动前运行一次配置检查。检查失败时**服务不会启动**，屏幕上会直接说明是哪一项有问题。改好后：

```bash
kaze config key=value   # 修改出错的那一项
kaze check              # 再检查一次
kaze restart            # 通过后启动
```

## 日常管理

装好后用 `kaze` 命令管理，常用的几个：

```bash
kaze status    # 运行状态
kaze log       # 最近日志
kaze restart   # 重启
```

装好后输入 `kaze` 打开菜单，完整命令见 [命令行](../reference/cli.md)。

!!! success "下一步"
    回到面板，用客户端订阅连一下这个节点试试。功能细节见 [功能配置](../features/limits.md)。
