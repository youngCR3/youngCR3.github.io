---
title: Linux
date: 2022-04-22 22:07:41
tags: Linux
categories: tools
---

Linux常用指令汇总

<!--more-->

- 查询使用某个端口的进程

```bash
ss -tnlp|grep ":1234"
```

- 查看某个进程

```bash
ps -ef|grep 1234
```

