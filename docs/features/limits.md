# 用户限制与风控

这些是节点级的默认值，写在配置里；面板下发的限速与本机值取较小者；面板下发的设备数直接生效，本机值只给面板没设的用户用。

## 限速

```ini
# 每个用户的限速上限,Mbps。0 = 只用面板给每个用户的限速
user_speed_limit=0
# 整个节点的总带宽上限,Mbps。0 = 不限
node_speed_limit=0
```

面板给用户下发了限速时，取**面板值和这里的较小值**。

## 设备数 / 连接数

```ini
# 面板没给设备数限制的用户,最多几个 IP 同时在线。0 = 不限
user_device_limit=0
# 每个用户最多同时几条连接。0 = 不限
user_tcp_limit=0
```

!!! note
    面板下发了 `device_limit` 时优先用面板值。跨节点的设备数靠面板汇总，约一分钟延迟（PPanel 不提供汇总，只按本节点计算）。

    节点套了中转或 CDN 时，设备数限制依赖能拿到真实 IP，见[中转与 CDN 真实 IP](realip.md)。

## 动态限速

用户在一段时间内平均速度超过阈值，就临时降速一段时间，之后自动恢复。只会收紧，不会放宽套餐本来的限速。

```ini
dy_limit_enable=false
# 生效时间段(服务器本地时区),可多段,可跨零点。留空=全天。例:20:00-02:00,10:00-14:00
dy_limit_duration=
dy_limit_trigger_time=60       # 统计多少秒的平均速度
dy_limit_trigger_speed=100     # 触发阈值 Mbps
dy_limit_speed=30              # 触发后限到多少 Mbps
dy_limit_time=600              # 限速持续多少秒
dy_limit_white_user_id=        # 永不限速的用户 ID,逗号分隔
```

## 防爆破

同一个 IP 短时间内反复认证失败（密码猜解、探测），自动封一段时间。

```ini
invalid_access_enable=false
invalid_access_count=30          # 多少次失败触发
invalid_access_duration=60       # 统计窗口,秒
invalid_access_forbidden_time=600 # 封禁时长,秒
```

!!! note
    开启前请确保节点能拿到真实 IP，否则可能误伤。
