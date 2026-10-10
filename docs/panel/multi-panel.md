# 一台机器多个面板

一个 kaze 进程对接一个面板。一台机器要给两个面板跑节点，就跑**两个实例**：程序只装一份，每个实例有自己的配置目录和服务，互不影响。

| | 第一个实例 | 第二个实例（名字 `b`） |
| --- | --- | --- |
| 配置目录 | `/etc/kaze` | `/etc/kaze-b` |
| 服务名 | `kaze` | `kaze-b` |
| 操作命令 | `kaze ...` | `kaze -n b ...` |
| 授权 | 面板 A 的授权码 | 面板 B 的授权码 |

## 添加第二个实例

<span class="kz-step">1</span> 第一个面板照常[安装](../quickstart/install.md)。

<span class="kz-step">2</span> 菜单里选 **18. 再加一个面板**，按提示填第二个面板的信息；或者直接在安装命令上加 `--name b`：

=== "Xboard 机器模式"

    把面板「添加机器」给出的命令换成 kaze 的脚本地址，末尾加 `--name b`：

    ```bash
    bash <(curl -fsSL https://github.com/kazeproxy/kaze-release/raw/main/install.sh) --mode machine --panel 'https://b.example.com' --token '机器令牌' --machine-id 7 --name b
    ```

=== "普通模式"

    ```bash
    bash <(curl -fsSL https://github.com/kazeproxy/kaze-release/raw/main/install.sh) install --name b --type xboard --node-id 12 --panel-url https://b.example.com --panel-key 通讯密钥 --license KZ-XXXX-XXXX-XXXX
    ```

实例名是 1–16 位小写字母或数字。第二个实例直接用已装好的程序，不重复下载。

## 日常操作

```bash
kaze instances          # 列出这台机器上的实例
kaze -n b               # 第二个实例的菜单
kaze -n b status        # 状态、日志、配置、授权……命令和第一个实例一样
kaze -n b config license_key=KZ-XXXX-XXXX-XXXX
kaze -n b restart
```

升级和回滚（`kaze update` / `kaze rollback`、自动升级）对所有实例一起生效：换一次程序，逐个重启，任何一个起不来就整体回滚。

移除第二个实例：`kaze -n b` 菜单里选 17，只停掉这个实例，程序和其他实例不受影响，配置留在 `/etc/kaze-b`。

## 注意

- **授权按面板算**：两个面板各需要一个授权码，同一个码给第二个面板用会被拒绝（`LICENSE_REFUSED mismatch`）。没有授权码的实例按免费版运行。
- **端口不能撞**：两个面板互相看不到对方分配到这台机器的节点，在两边建节点时要错开端口。撞了的那个节点会在日志里报错，换端口后自动恢复。
- 端口跳跃的转发规则各实例各自维护，范围也不要重叠。
- Docker 用户不需要这些：在 Compose 里写两个 service、各挂一个目录即可：

    ```yaml
    services:
      kaze-a:
        image: ghcr.io/kazeproxy/kaze:latest
        network_mode: host
        restart: always
        environment: {type: xboard, node_id: "1", panel_url: https://a.example.com, panel_key: 密钥A, license_key: KZ-AAAA-AAAA-AAAA}
        volumes: [./a:/etc/kaze]
      kaze-b:
        image: ghcr.io/kazeproxy/kaze:latest
        network_mode: host
        restart: always
        environment: {type: xboard, node_id: "7", panel_url: https://b.example.com, panel_key: 密钥B, license_key: KZ-BBBB-BBBB-BBBB}
        volumes: [./b:/etc/kaze]
    ```
