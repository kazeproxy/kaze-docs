# 端口跳跃

Hysteria2 和 TUIC 客户端可以在一段端口里不断换端口连接，躲开针对单个 UDP 端口的限速和封锁。节点只监听一个端口；kaze 在内核里把整段端口转发到它，不经过 kaze 进程，没有额外开销。

转发规则写在 kaze 自己的 nftables 表 `kaze_hop` 里（没有 nftables 时用 iptables 的 `KAZE_HOP` 链），节点停止时自动删除，不会动你已有的防火墙规则。

## 设置

=== ":material-alpha-x-box: Xboard / V2board / v2node"

    1. 面板「节点管理 → 编辑节点」里编辑 Hysteria2 / TUIC 节点：**连接端口** 填端口段，例如 `20000-30000`;**服务端口** 填节点实际监听的端口，例如 `443`。面板会把端口段下发进用户订阅。
    2. 节点配置里填同一段端口：

        ```ini
        hop_ports=20000-30000
        ```

    面板不会把连接端口下发给节点，所以第 2 步不能省。

=== ":material-alpha-p-box: PPanel"

    PPanel 后台编辑 Hysteria2 节点时填端口跳跃范围即可，面板会把范围下发给节点，节点配置不用改（Docker 需要给容器 `NET_ADMIN`，见 [Docker 部署](../quickstart/docker.md)）。

端口段可写 `20000-30000` 或 `20000:30000`，多段用逗号分隔：`20000-25000,30000-35000`。节点配置里的 `hop_ports` 优先于面板。

启动后日志出现 `port hopping ports=20000-30000 to=443` 即生效。

## 权限

写内核转发规则需要 `CAP_NET_ADMIN`。

- **一键脚本安装**：已自带，不用处理。
- **Docker**：加上这一项能力：

    ```yaml
    services:
      kaze-hy2:
        image: ghcr.io/kazeproxy/kaze:latest
        network_mode: host
        cap_add:
          - NET_ADMIN
    ```

- **手动运行**：用 root 运行，或给 systemd 服务加 `AmbientCapabilities=CAP_NET_ADMIN`。

权限不够时节点照常运行，日志提示 `port hopping not set up`:客户端仍能连节点本身的端口，只是跳跃的那些端口不通。

!!! warning "云服务器安全组"
    记得在云厂商的安全组 / 防火墙里放行整段 **UDP** 端口。
