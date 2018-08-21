---
title: 时间处理总结 shell python
date: 2018-08-10 17:30:26
categories: 珍惜时间
tags:
    - time
    - datetime
---

# shell 篇

date命令

linux 和 mac 不一样！
环境centos 6

``` bash 
date -d "1 days ago" +%F  # 指定显示日期的格式 %F | full date; same as %Y-%m-%d 
date +"%Y%m%d" -d "-2 day"  # -d 指定哪一天
2017-03-04

```

## 指定显示日期的格式

| 格式 | 含义                                  |
| ---- | ------------------------------------- |
| %s   | seconds since 1970-01-01 00:00:00 UTC |
| %F   | full date; same as %Y-%m-%d           |
| %Y   | year                                  |
| %m   | month                                 |
| %d   | day                                   |

时间戳 seconds since 1970-01-01 00:00:00 UTC