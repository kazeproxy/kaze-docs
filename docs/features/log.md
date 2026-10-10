# 日志

## 日志级别

```ini
log_level=info     # debug / info / warn / error
```

默认输出到系统日志（journald）。用 `kaze log` 或 `kaze follow` 查看。

## 日志文件

同时写到文件，每天一个，超过保留天数自动删除。

```ini
log_file_dir=/var/log/kaze
log_file_retention_days=7      # 0 = 永久保留
```

!!! note
    kaze 只会删除自己按日期命名的日志文件，不碰目录里别的文件。
