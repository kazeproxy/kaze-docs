# 命令行

## 菜单

一键安装后，在服务器上输入 `kaze` 就会出现菜单，输入数字执行：

```text
  kaze v0.5.0   状态：运行中   自动升级：关   开机自启：开

  —— 服务 ——
   1. 查看运行状态
   2. 启动
   3. 停止
   4. 重启

  —— 日志与检查 ——
   5. 查看最近日志
   6. 实时查看日志（Ctrl+C 返回）
   7. 对着面板检查配置

  —— 配置 ——
   8. 查看当前配置
   9. 修改配置
  10. 查看授权状态
  11. 更新 geo 数据

  —— 版本 ——
  12. 升级到最新版
  13. 回滚到上一个版本
  14. 自动升级：开 / 关
  15. 查看版本

  —— 其他 ——
  16. 开机自启：开 / 关
  17. 卸载 kaze
  18. 再加一个面板（第二个实例）
   0. 退出

  请输入数字：
```

## 命令

菜单里的每一项也可以直接用命令执行，适合写进脚本。`kz` 是 `kaze` 的简写，两个都能用；v0.4.2 及更早的版本只有 `kz`。

| 命令 | 作用 |
| --- | --- |
| `kaze status` | 查看运行状态 |
| `kaze start` / `kaze stop` / `kaze restart` | 启动 / 停止 / 重启 |
| `kaze enable` / `kaze disable` | 开启 / 关闭开机自启 |
| `kaze log [行数]` | 查看最近日志，默认 100 行 |
| `kaze follow` | 持续滚动查看日志 |
| `kaze config` | 查看当前配置（密钥自动打码） |
| `kaze config key=value ...` | 修改设置，改完 `kaze restart` 生效 |
| `kaze check` | 对着面板检查配置，不启动 |
| `kaze -reality-keypair` | 生成一对 REALITY 密钥（面板没有生成按钮时用） |
| `kaze license` | 查看授权状态 |
| `kaze instances` | 列出这台机器上的实例（[多个面板](../panel/multi-panel.md)） |
| `kaze add` | 再加一个面板（第二个实例） |
| `kaze -n 名字 ...` | 对第二个实例执行以上任一命令 |
| `kaze geo` | 下载 / 更新 geosite、geoip 数据文件 |
| `kaze version` | 查看版本 |
| `kaze update` | 升级到最新版。旧版本会保留；新版本启动后没跑起来会自动退回旧版 |
| `kaze rollback` | 退回上一个版本（再执行一次回到新版） |
| `kaze auto-update on` / `off` | 开启 / 关闭每日自动升级。各节点在 6 小时内随机时间升级，不会同时升级 |

!!! example "改设置的完整流程"
    ```bash
    kaze config user_speed_limit=100 forbidden_bit_torrent=true
    kaze check
    kaze restart
    ```

## kaze 程序参数

手动运行或 Docker 下直接调用程序时使用。

| 参数 | 作用 |
| --- | --- |
| `-c 文件` | 指定配置文件，默认 `/etc/kaze/kaze.conf` |
| `-check` | 对着面板检查配置后退出 |
| `-license` | 打印授权状态后退出 |
| `-reality-keypair` | 生成一对 REALITY 密钥后退出 |
| `-update-geo` | 下载 / 更新 geo 数据文件后退出 |
| `-v` | 打印版本 |

## Docker 与手动安装

`kaze xxx` 这些命令来自一键脚本装的菜单。Docker 的对应命令见 [Docker 部署 → 日常操作](../quickstart/docker.md#日常操作)，手动安装的见[手动安装 → 日常管理](../quickstart/manual.md#日常管理)。

## 文件位置

一键安装的布局；手动安装时程序直接放在 `/usr/local/bin/kaze`，见[手动安装](../quickstart/manual.md)。

| 路径 | 内容 |
| --- | --- |
| `/usr/local/kaze/kaze` | 程序（`kaze.prev` 是上一个版本，供回滚） |
| `/usr/local/bin/kaze`、`/usr/local/bin/kz` | 菜单和管理命令 |
| `/etc/kaze/kaze.conf` | 配置文件 |
| `/etc/kaze/certs/` | 自动申请的证书 |
| `/etc/kaze/geosite.dat`、`geoip.dat` | geo 数据文件 |
| `/etc/systemd/system/kaze.service` | 系统服务 |
