---
layout: post
title: mysqldump: unknown option '--no-beep' 错误解决
date: 2018-02-05
categories: blog
tags: [mysql,mysqldump]
description: mysqldump 命令备份数据库时，unknown option '--no-beep' 错误解决！！！
---

#### 今天用 mysqldump 命令备份数据库时，出现了问题，截图如下：
![image](https://note.youdao.com/yws/api/personal/file/C4E765EF35C744A09AD4697E6089ED4E?method=download&shareKey=b15916e0fee6e7455acb2069034ccc97)
##### 估计是版本的问题，我新装的 5.6 的，以前用的是 5.1 的，从来没出过问题。

网上找了一下，说查看 my.ini 发现[clien]下有 no-beep 参数，mysql客户端将会读取此参数(该参数作用暂时不知)。

解决办法是：

- 1.删除my.ini [client]下的 no-beep 参数;
- 2.在 mysqldump 后加--no-defaults参数,即:mysqldump --no-defualts -h主机IP - -u用户名 -p密码 数据库 > xxx.sql 。

经测试，这两种方法有效。



```
@echo off
set "Ymd=%date:~,4%%date:~5,2%%date:~8,2%"
"C:\Program Files\MySQL\MySQL Server 5.6\bin\mysqldump" --no-defaults -u root --password=1qaz4321 api_cms> D:\db_backup\api_cms\api_cms-%Ymd%.sql

"C:\Program Files\MySQL\MySQL Server 5.6\bin\mysqldump" --no-defaults -u root --password=1qaz4321 test> D:\db_backup\test\test-%Ymd%.sql
@echo on
```
