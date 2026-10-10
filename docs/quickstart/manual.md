# 手动安装

想自己掌控每一步时用这种方式。

## 下载程序

=== "amd64 (x86_64)"

    ```bash
    curl -fsSL -o /usr/local/bin/kaze \
      https://github.com/kazeproxy/kaze-release/releases/latest/download/kaze-linux-amd64
    chmod +x /usr/local/bin/kaze
    ```

=== "arm64 (aarch64)"

    ```bash
    curl -fsSL -o /usr/local/bin/kaze \
      https://github.com/kazeproxy/kaze-release/releases/latest/download/kaze-linux-arm64
    chmod +x /usr/local/bin/kaze
    ```

??? tip "校验下载的文件（推荐）"
    每个版本都附带 `SHA256SUMS`，核对一下能确认拿到的是官方构建：
    ```bash
    cd /tmp
    curl -fsSL -O https://github.com/kazeproxy/kaze-release/releases/latest/download/SHA256SUMS
    sha256sum /usr/local/bin/kaze
    grep kaze-linux SHA256SUMS
    ```
    两边的校验值一致即可。

## 写配置

```bash
mkdir -p /etc/kaze
cat > /etc/kaze/kaze.conf <<'EOF'
type=xboard
node_id=1
webapi_url=https://你的面板
webapi_key=通讯密钥
EOF
```

!!! note "对接 V2board"
    再加一行 `server_type=vless`（换成你的协议）。

## 检查配置

```bash
kaze -c /etc/kaze/kaze.conf -check
```

它会连面板拉一次配置和用户，告诉你这个节点会怎么跑，但不监听端口。有问题会直接指出来。

## 注册系统服务

```ini title="/etc/systemd/system/kaze.service"
[Unit]
Description=kaze node
After=network-online.target
Wants=network-online.target

[Service]
User=kaze
Group=kaze
ExecStart=/usr/local/bin/kaze -c /etc/kaze/kaze.conf
Restart=always
RestartSec=3
LimitNOFILE=1048576
AmbientCapabilities=CAP_NET_BIND_SERVICE
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
```

```bash
useradd --system --no-create-home --shell /usr/sbin/nologin kaze
chown -R kaze:kaze /etc/kaze && chmod 600 /etc/kaze/kaze.conf
systemctl daemon-reload
systemctl enable --now kaze
systemctl status kaze
```

!!! info "为什么用单独的 kaze 用户"
    节点不需要 root 权限。`CAP_NET_BIND_SERVICE` 让它能监听 443 这类低端口，其它权限一律不给，即使出问题影响也有限。

## 日常管理

手动安装没有一键脚本带的 `kaze` 菜单命令（那个菜单脚本装在 `/usr/local/bin/kaze`，程序本身在 `/usr/local/kaze/kaze`；手动安装时 `/usr/local/bin/kaze` 就是程序本身）。文档里的 `kaze xxx` 对应：

| 文档里写的 | 手动安装时 |
| --- | --- |
| `kaze restart` / `kaze status` | `systemctl restart kaze` / `systemctl status kaze` |
| `kaze log` | `journalctl -u kaze -n 100 --no-pager` |
| `kaze check` | `kaze -c /etc/kaze/kaze.conf -check` |
| `kaze license` | `kaze -c /etc/kaze/kaze.conf -license` |
| `kaze geo` | `kaze -c /etc/kaze/kaze.conf -update-geo` |
| `kaze config 配置项=值` | 编辑 `/etc/kaze/kaze.conf` 后 `systemctl restart kaze` |
| `kaze update` | 重新下载二进制覆盖后 `systemctl restart kaze` |
| `kaze -reality-keypair` | 相同（这是程序参数，不经过菜单） |
