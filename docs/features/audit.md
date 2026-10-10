# 审计与屏蔽

## 屏蔽 BT

默认开启。通过嗅探连接特征拦截 BitTorrent 的握手、DHT、tracker 汇报。

```ini
forbidden_bit_torrent=true
```

## 屏蔽内网

防止用户通过节点探测服务器所在的内网。默认开启。

```ini
forbidden_private_ip=true
```

## 端口黑名单

禁止代理某些目标端口，逗号分隔，可写范围。常用来屏蔽发信端口防垃圾邮件：

```ini
forbidden_ports=25,465,587
```

## 审计规则（黑名单）

拦截指定的域名、IP、端口。规则文件一行一条：

```ini
block_list_file=/etc/kaze/block.txt
```

规则格式：

```
example.com            域名里包含这段文字就屏蔽
domain:example.com     这个域名和它的子域名
full:example.com       只匹配这一个域名
regexp:.*\.example\.com 正则
ip:1.2.3.0/24          目标 IP 段
port:25,6881-6889      目标端口
geosite:category-ads   数据文件里的一组域名(见下)
geoip:cn               数据文件里一个国家 / 地区的 IP 段
```

## 远程审计规则

规则放在一个地址上，定时下载、自动更新，更新后立即生效不用重启。下载失败时沿用上一次的规则。

```ini
block_list_url=https://example.com/blockList
block_list_update_interval=60    # 更新间隔,分钟
```

## 白名单

白名单里的目标不会被审计规则和面板路由屏蔽（屏蔽 BT、屏蔽内网两项除外），格式同审计规则。

```ini
white_list_file=/etc/kaze/white.txt
```

## geosite / geoip 数据文件

`geosite:` 和 `geoip:` 规则用通用的 `geosite.dat` 和 `geoip.dat`。下载 / 更新：

```bash
kaze geo
# 或手动运行:
kaze -c /etc/kaze/kaze.conf -update-geo
```

默认存在配置文件同目录，也可以指定路径：

```ini
geosite_file=/etc/kaze/geosite.dat
geoip_file=/etc/kaze/geoip.dat
```
