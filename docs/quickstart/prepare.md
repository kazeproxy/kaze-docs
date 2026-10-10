# 准备工作

开始前确认以下几样东西都已就绪。

## 服务器

- [x] 一台 Linux 服务器（VPS），**64 位**，架构为 `x86_64`（amd64）或 `aarch64`(arm64)
- [x] 推荐系统:Debian 11+ / Ubuntu 20.04+ / CentOS Stream 9 / Rocky / AlmaLinux
- [x] 有 `root` 权限
- [x] 防火墙或安全组**放行节点的连接端口**（TCP 协议放 TCP，Hysteria2 / TUIC 放 UDP，端口跳跃放整段）

??? question "怎么看服务器架构?"
    ```bash
    uname -m
    ```
    输出 `x86_64` 用 amd64 版本，输出 `aarch64` 用 arm64 版本。一键安装脚本会自动识别，不用手动选。

## 面板里的节点

在 Xboard、V2board 或 PPanel 后台**新建一个节点**，设置好协议、端口、传输、TLS 等。然后记下三样东西：

| 要记下的 | 在哪里找 | 示例 |
| --- | --- | --- |
| :material-identifier: **节点 ID** | 节点列表里的数字（PPanel 填服务器 ID） | `1` |
| :material-web: **面板地址** | 面板的**后台 API 地址**。前后端一起部署时就是浏览器访问面板的地址；用户前端单独部署（主题 `config.js` 里有 `server_url`）时填 `server_url` 那个域名。不要带 `/admin` 等路径 | `https://panel.example.com` |
| :material-key-variant: **通讯密钥** | Xboard:后台「系统配置 → 节点配置 → 通讯密钥」；V2board:「系统配置 → 节点 → 通讯密钥」；PPanel:「系统设置 → 节点配置 → Secret Key」 | `xxxxxxxxxxxx` |

!!! warning "通讯密钥要保密"
    通讯密钥能读取你所有用户的连接信息。不要截图外传，不要提交到公开仓库。

## 域名与证书（可选）

用到 TLS 的节点（Trojan、AnyTLS、Hysteria2、TUIC，或开了 TLS 的 VLESS / VMess）需要证书：

- **有域名**：在域名服务商那里加一条 **A 记录**，例如 `node1.example.com → 1.2.3.4`（你的服务器 IP），再在面板节点的「服务器名称 SNI」里填这个域名，并把证书模式选成 **http**（Xboard：高级协议配置 → TLS → 证书模式）。kaze 会自动申请证书并续期，要求 **80 端口能从公网访问**；不选模式时用自签证书，客户端要允许不安全连接。
- **用 REALITY**：不需要域名和证书。

详见 [TLS 证书](../features/cert.md)。

## 选择安装方式

<div class="grid cards" markdown>

-   :material-console:{ .lg .middle } **一键安装**（推荐）

    ---

    一条命令装好并注册为系统服务，带 `kaze` 管理命令。

    [:octicons-arrow-right-24: 一键安装](install.md)

-   :material-docker:{ .lg .middle } **Docker 部署**

    ---

    已经在用 Docker，或想把节点和其它服务隔离。

    [:octicons-arrow-right-24: 先安装 Docker](install-docker.md)

-   :material-wrench:{ .lg .middle } **手动安装**

    ---

    自己管理程序和 systemd 服务。

    [:octicons-arrow-right-24: 手动安装](manual.md)

</div>
