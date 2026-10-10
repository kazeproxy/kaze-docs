# 中转与 CDN 真实 IP

节点套了中转、负载均衡或 CDN 时，kaze 看到的是中转 / CDN 的地址，而不是用户的。设备数限制、防爆破都依赖真实 IP，这时要让 kaze 还原出用户地址。

## CDN 后的真实 IP（HTTP 请求头）

节点套了 Cloudflare 这类 CDN，或用 nginx 反代时，从请求头里取真实 IP。

```ini
trusted_x_forwarded_for=CF-Connecting-IP,X-Forwarded-For
```

按顺序取第一个有值的。**只对 WebSocket、HTTPUpgrade、gRPC、H2、XHTTP 传输有效。**

!!! warning
    只填前面那一层**确实会设置**的请求头。填了它不会设置的，用户可以自己伪造 IP 绕过设备数限制。Cloudflare 填 `CF-Connecting-IP`,nginx 反代一般填 `X-Forwarded-For`。

## 中转后的真实 IP(PROXY protocol)

节点前面有 TCP 中转（HAProxy、专线等）并发送 PROXY protocol 头时：

```ini
proxy_protocol=true
# 强制要求 PROXY 头,没有的连接一律拒绝(开启后只能经中转访问):
force_proxy_protocol=false
```
