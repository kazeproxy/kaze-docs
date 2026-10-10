# 升级与卸载

## 升级

=== ":material-console: 一键安装的"

    ```bash
    kaze update
    ```
    下载最新版并重启，配置保持不变。旧版本会保留下来：新版本启动后没跑起来，会自动退回旧版；跑起来后发现问题，执行 `kaze rollback` 退回。

    想让节点自己保持最新：`kaze auto-update on`。每天检查一次，有新版才升级，各节点在 6 小时内的随机时间升级，不会同时升级。

    !!! note "v0.4.2 及更早的版本"
        这些版本的 `kaze` 还没有 `update` 命令，先用一次：
        ```bash
        bash <(curl -fsSL https://github.com/kazeproxy/kaze-release/raw/main/install.sh) update
        ```

=== ":material-docker: Docker Compose"

    ```bash
    cd /opt/kaze
    docker compose pull
    docker compose up -d
    ```

=== ":material-docker: docker run"

    ```bash
    docker pull ghcr.io/kazeproxy/kaze:latest
    docker rm -f kaze
    # 再执行一次当初的 docker run 命令
    ```

=== ":material-wrench: 手动安装的"

    重新下载程序覆盖 `/usr/local/bin/kaze`，然后 `systemctl restart kaze`。

!!! tip "查看当前版本"
    `kaze version`，或 Docker 下 `docker exec kaze kaze -v`。所有版本见 [发布页](https://github.com/kazeproxy/kaze-release/releases)。

## 卸载

=== ":material-console: 一键安装的"

    ```bash
    bash <(curl -fsSL https://github.com/kazeproxy/kaze-release/raw/main/install.sh) uninstall
    ```

    会删除程序和服务，**保留配置**在 `/etc/kaze`。彻底清除：

    ```bash
    rm -rf /etc/kaze && userdel kaze
    ```

=== ":material-docker: Docker"

    ```bash
    cd /opt/kaze && docker compose down
    rm -rf /opt/kaze            # 连同证书等数据一起删除
    ```
