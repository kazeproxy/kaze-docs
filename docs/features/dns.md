# DNS 解析

节点解析目标域名用的 DNS。

## 默认 DNS

逗号分隔，按顺序尝试，前一个失败用下一个。留空用系统 DNS。

```ini
default_dns=tcp-tls://1.1.1.1,8.8.8.8
```

支持的地址格式：

| 写法 | 协议 |
| --- | --- |
| `8.8.8.8` 或 `8.8.8.8:5353` | 普通 DNS(UDP) |
| `tcp://8.8.8.8` | 普通 DNS(TCP) |
| `tcp-tls://dns.google` 或 `tls://dns.google` | DNS over TLS（两种写法等价） |
| `https://dns.google/dns-query` | DNS over HTTPS |
| `tcp-tls://1.1.1.1#cloudflare-dns.com` | 用 IP 连接，按 `#` 后面的名称验证证书。用于证书里不含 IP 的服务器；`https://` 同样可用 |

```ini
dns_cache_time=10              # 缓存时间,分钟
dns_strategy=ipv4_first        # ipv4_first / ipv4_only / ipv6_first / ipv6_only
```

## DNS 规则文件（按域名分流）

指定的域名用指定的 DNS 解析，常用于流媒体解锁。

```ini
dns_rules_file=/etc/kaze/dns.conf
```

文件一行一条，`DNS服务器[,备用] = 规则, 规则 ...`:

```
https://dns.google/dns-query = geosite:netflix, domain:hulu.com
1.1.1.1, tcp-tls://dns.google = domain:disney.com
```

命中规则的域名用该行的 DNS 解析，其余域名仍走 `default_dns` 或系统 DNS。
