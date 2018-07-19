---
title: hadoop common warn and error
date: 2018-07-19 20:49:37
tags:
---

# 常规报错

不影响运行的错误

### Container preempted by scheduler

如果有fair scheduler 并且有很多不同的 queue ，高优的应用程序会把jobs杀掉。每个queue都有相应的优先级 Preemption，

https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.3.2/bk_yarn_resource_mgt/content/preemption.html

解决方案：取决于对性能和效果的预期，好的方法是检查job和队列的优先级。