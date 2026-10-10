# 安装 Docker

用 Docker 部署 kaze 之前，服务器上要先装好 Docker。已经装过的可以直接跳到 [部署 kaze](docker.md)。

??? question "怎么判断装没装?"
    ```bash
    docker --version && docker compose version
    ```
    两条都能输出版本号，就说明 Docker 和 Compose 插件都装好了，可以跳过本页。

## 安装

=== ":material-flash: 官方一键脚本（推荐）"

    适用于 Debian、Ubuntu、CentOS、Rocky、AlmaLinux、Fedora 等主流系统，会同时装好 Docker 和 Compose 插件。

    ```bash
    curl -fsSL https://get.docker.com | bash
    ```

=== ":material-map-marker: 中国大陆服务器"

    官方源在国内可能很慢，可以让脚本改用阿里云镜像：

    ```bash
    curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
    ```

=== ":material-debian: Debian / Ubuntu（手动）"

    ```bash
    apt-get update
    apt-get install -y ca-certificates curl
    install -m 0755 -d /etc/apt/keyrings
    . /etc/os-release
    curl -fsSL https://download.docker.com/linux/$ID/gpg -o /etc/apt/keyrings/docker.asc
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
      https://download.docker.com/linux/$ID $VERSION_CODENAME stable" \
      > /etc/apt/sources.list.d/docker.list
    apt-get update
    apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
    ```

=== ":material-redhat: CentOS / Rocky / Alma（手动）"

    ```bash
    dnf install -y dnf-plugins-core
    dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    dnf install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
    ```

=== ":material-linux: Alpine"

    ```bash
    apk add docker docker-cli-compose
    rc-update add docker default
    service docker start
    ```

## 启动并设为开机自启

```bash
systemctl enable --now docker
```

!!! note "Alpine 用户"
    Alpine 使用 OpenRC，上一步的 `rc-update` 和 `service` 已经完成了这一步，跳过即可。

## 验证

```bash
docker --version
docker compose version
docker run --rm hello-world
```

最后一条输出 `Hello from Docker!` 就说明安装成功。

!!! success "Docker 装好了"
    下一步:[:material-arrow-right: 部署 kaze](docker.md)

## 常见问题

??? failure "`curl: command not found`"
    先装 curl:Debian / Ubuntu 用 `apt-get install -y curl`,CentOS 系用 `dnf install -y curl`。

??? failure "`docker compose` 提示不是命令，但 `docker-compose` 可以用"
    你装的是旧版独立 Compose。本文档的命令用新版插件写法 `docker compose`（中间是空格）。按上面的方法重装，或把命令里的 `docker compose` 换成 `docker-compose`。

??? failure "脚本卡住不动或下载很慢"
    多半是网络问题。国内服务器用上面「中国大陆服务器」页签里的镜像命令。
