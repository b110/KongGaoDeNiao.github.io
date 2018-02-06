---
layout: post
title: Mysql的FIND_IN_SET的使用
date: 2018-01-08
categories: blog
tags: [mysql]
description: Mysql的FIND_IN_SET的使用。
---

#### Mysql的FIND_IN_SET的使用
##### 背景：查询参展展会中所有的产品的名称，参展展会表存储的产品id product以","分割；

```
参展展会表
id
name
product（存储id","分割 例如：1,2,3）

测试数据：
1 张三 1,2,3
2 李四 2,3
```
```
产品表
id
name

测试数据：
1 液态食品包装、灌装设备及技术
2 自动化、控制系统及检测设备
3 后段包装设备及技术
4 PET设备及技术
```

```
查询思路：
1.现将产品按照id在'1,2,4'集合中查询出来

 SELECT p.name FROM dict_product p WHERE FIND_IN_SET(p.id,'1,2,4')
```
![这里写图片描述](http://img.blog.csdn.net/20171129141102993?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjI5MDYzNDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
```

2.查询返回的是多个值使用GROUP_CONCAT合成一条

 SELECT
	e.companyname,
	(
		SELECT
			GROUP_CONCAT(p. NAME)
		FROM
			dict_product p
		WHERE
			FIND_IN_SET(p.id, e.product)
	) as product
FROM
	join_exhibition e
```
![这里写图片描述](http://img.blog.csdn.net/20171129143351336?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjI5MDYzNDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


####  FIND_IN_SET(str,strlist) 
MySQL手册中find_in_set函数的语法：
FIND_IN_SET(str,strlist)

str 要查询的字符串
strlist 字段名 参数以”,”分隔 如 (1,2,6,8)
查询字段(strlist)中包含(str)的结果，返回结果为null或记录

假如字符串str在由N个子链组成的字符串列表strlist 中，则返回值的范围在 1 到 N 之间。 一个字符串列表就是一个由一些被 ‘,’ 符号分开的子链组成的字符串。如果第一个参数是一个常数字符串，而第二个是type SET列，则FIND_IN_SET() 函数被优化，使用比特计算。 如果str不在strlist 或strlist 为空字符串，则返回值为 0 。如任意一个参数为NULL，则返回值为 NULL。这个函数在第一个参数包含一个逗号(‘,’)时将无法正常运行。

#### 补充：有点类似in。。。。。。。












