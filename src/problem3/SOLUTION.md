Provide your solution here:

## Steps
1. Check disk usage
``` shell
df -h
```
2. Identify largest directories
``` shell
du -h --max-depth=1 / | sort -hr
du -h --max-depth=1 /var | sort -hr
```
3. Inspect NGINX logs
``` shell
ls -lh /var/log/nginx
du -sh /var/log/nginx/*
```

## Root Causes and Recovery
1. Log Rotation Missing

**Impact**
- Disk full
- Service instability

**Recovery**

``` shell
gzip /var/log/nginx/*.log
truncate -s 0 /var/log/nginx/access.log
truncate -s 0 /var/log/nginx/error.log
```

2. High Traffic

**Impact**
- Rapid disk consumption
- Performance degradation

**Recovery**
- Scale up NginX server

3. Debug Logging Enabled

**Impact**
- Extremely fast log growth

**Recovery**
- Update Nginx logging configuration and restart nginx service

/etc/nginx/conf.d/nginx.conf
```
...
error_log /var/log/nginx/error.log warn;
...
```

```shell
systemctl reload nginx
```
