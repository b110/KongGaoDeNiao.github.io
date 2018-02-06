---
layout: post
title: Mysql查询显示行号,实现类似Oracle数据库的ROWNUM()
date: 2018-01-31
categories: blog
tags: [mysql]
description: Mysql查询显示行号,实现类似Oracle数据库的ROWNUM()。
---

### Mysql查询显示行号,实现类似Oracle数据库的ROWNUM()

在mysql数据库中没有ROWNUM()这个函数,只能通过变量的方式来实现.

说明：表heyf_10中字段，empid(员工工号)、deptid(部门编号)、salary(薪资)；
rownum是自定义变量，表示行号。

![image](https://note.youdao.com/yws/api/personal/file/75623BF9248F4697A82116C248D8A9D8?method=download&shareKey=dcf4117ebafad6d5e26e43a8fda7642d)

```
SELECT
	(@rownum :=@rownum + 1) AS rownum,
	t.*
FROM
	menu t,
	(SELECT @rownum := 0) b
ORDER BY
	t. CODE ASC
```
