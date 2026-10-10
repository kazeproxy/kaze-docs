# 部署 kaze

!!! warning "先装 Docker"
    还没装 Docker 的，先看 [安装 Docker](install-docker.md)。

镜像地址：`ghcr.io/kazeproxy/kaze:latest`，同时提供 amd64 和 arm64,Docker 会自动拉取对应架构。

所有设置都通过**环境变量**传入，变量名就是配置项的名字，不需要配置文件。需要文件的设置（`routes_file`、`block_list_file`、`dns_rules_file`、`cert_file`、`license_file` 等）把文件放进挂载到 `/etc/kaze` 的目录（下面例子里的 `./data`），值写容器内路径，例如 `routes_file: /etc/kaze/routes.conf`。

## 方式一:Docker Compose（推荐）

用 Compose 管理，改配置、升级、看日志都更方便。

<span class="kz-step">1</span> 建一个目录并进入：

```bash
mkdir -p /opt/kaze && cd /opt/kaze
```

<span class="kz-step">2</span> 新建 `docker-compose.yml`:

=== ":material-alpha-x-box: 对接 Xboard"

    ```yaml title="docker-compose.yml"
    services:
      kaze:
        image: ghcr.io/kazeproxy/kaze:latest
        container_name: kaze
        restart: always
        network_mode: host          # (1)!
        environment:
          type: xboard
          node_id: "1"              # (2)!
          panel_url: https://你的面板
          panel_key: 通讯密钥
          # license_key: KZ-XXXX-XXXX-XXXX  # (3)!
        volumes:
          - ./data:/etc/kaze         # (4)!
        ulimits:
          nofile: 1048576
    ```

    1. 用 host 网络，省去端口映射，也能正确拿到客户端地址。
    2. 多个节点用逗号分隔：`"1,2,3"`，可以是不同协议。
    3. 不填就是免费版。
    4. 存放自动申请的证书和 geo 数据文件，升级容器不会丢。

=== ":material-alpha-v-box: 对接 V2board"

    ```yaml title="docker-compose.yml"
    services:
      kaze:
        image: ghcr.io/kazeproxy/kaze:latest
        container_name: kaze
        restart: always
        network_mode: host
        environment:
          type: v2board
          server_type: vless        # (1)!
          node_id: "1"
          panel_url: https://你的面板
          panel_key: 通讯密钥
        volumes:
          - ./data:/etc/kaze
        ulimits:
          nofile: 1048576
    ```

    1. V2board 必须指定协议。

=== ":material-alpha-p-box: 对接 PPanel"

    ```yaml title="docker-compose.yml"
    services:
      kaze:
        image: ghcr.io/kazeproxy/kaze:latest
        container_name: kaze
        restart: always
        network_mode: host
        cap_add:
          - NET_ADMIN               # (1)!
        environment:
          type: ppanel
          node_id: "5"              # (2)!
          server_type: vless,trojan,hysteria  # (3)!
          panel_url: https://你的面板
          panel_key: 通讯密钥         # (4)!
        volumes:
          - ./data:/etc/kaze
        ulimits:
          nofile: 1048576
    ```

    1. Hysteria2 在面板里开了端口跳跃时需要；不用可以删掉。
    2. PPanel 里的服务器 ID。
    3. 要跑这台服务器的哪几种协议，逗号分隔，每种按面板设置的端口各自启动。写法见 [对接 PPanel](../panel/ppanel.md)。
    4. PPanel 的 Secret Key。

=== ":material-alpha-v-circle: 对接 V2board（v2node 节点）"

    ```yaml title="docker-compose.yml"
    services:
      kaze:
        image: ghcr.io/kazeproxy/kaze:latest
        container_name: kaze
        restart: always
        network_mode: host
        environment:
          type: v2node              # (1)!
          node_id: "1"
          panel_url: https://你的面板
          panel_key: 通讯密钥
        volumes:
          - ./data:/etc/kaze
        ulimits:
          nofile: 1048576
    ```

    1. 面板里节点类型为 wyx2685 的 v2node 时用，见 [对接 V2board（v2node 节点）](../panel/v2node.md)。

<span class="kz-step">3</span> 启动：

```bash
docker compose up -d
```

<span class="kz-step">4</span> 看日志确认正常：

```bash
docker compose logs -f
```

看到 `listening` 就说明节点已经在服务了。按 ++ctrl+c++ 退出日志查看，不影响运行。

## 方式二:docker run

不想写文件的话，一条命令也行：

```bash
docker run -d --name kaze --restart always --network host \
  --ulimit nofile=1048576:1048576 \
  -e type=xboard \
  -e node_id=1 \
  -e panel_url=https://你的面板 \
  -e panel_key=通讯密钥 \
  -v /opt/kaze/data:/etc/kaze \
  ghcr.io/kazeproxy/kaze:latest
```

## 日常操作

| 操作 | Compose | docker run |
| --- | --- | --- |
| 查看日志 | `docker compose logs -f` | `docker logs -f kaze` |
| 重启 | `docker compose restart` | `docker restart kaze` |
| 停止 | `docker compose down` | `docker rm -f kaze` |
| 升级到最新版 | `docker compose pull && docker compose up -d` | 见 [升级与卸载](upgrade.md) |
| 检查配置（节点运行中也可以） | `docker compose run --rm kaze -check` | — |
| 生成 REALITY 密钥 | `docker compose exec kaze kaze -reality-keypair` | `docker exec kaze kaze -reality-keypair` |
| 回滚到某个版本 | 把 `image:` 改成 `ghcr.io/kazeproxy/kaze:v0.5.1` 再 `docker compose up -d` | `docker run` 时指定同样的 tag |
| 查看授权状态（`kaze license`） | `docker compose exec kaze kaze -license` | `docker exec kaze kaze -license` |
| 更新 geo 数据（`kaze geo`） | `docker compose exec kaze kaze -update-geo` | `docker exec kaze kaze -update-geo` |

常用的几项放在一起：

```yaml
    environment:
      license_key: KZ-XXXX-XXXX-XXXX     # 或 license_file: /etc/kaze/license.txt
      routes_file: /etc/kaze/routes.conf  # 文件放在 ./data/routes.conf
      cert_mode: file
      cert_file: /etc/kaze/cert.pem       # ./data/cert.pem
      key_file: /etc/kaze/key.pem
      hop_ports: 20000-30000              # Hysteria2 端口跳跃，同时要 cap_add: [NET_ADMIN]
      TZ: Asia/Shanghai                   # 动态限速的时段按容器时区算，默认 UTC
```

!!! tip "改设置"
    Compose:改 `docker-compose.yml` 里的 `environment`，然后 `docker compose up -d` 生效。
    所有可用设置见 [配置项速查表](../reference/all-settings.md)，写法统一为 `配置项: 值`。
