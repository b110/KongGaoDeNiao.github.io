---
layout: post
title: SpringMVC前台日期作为实体类对象参数类型转换错误解决.
date: 2018-02-06
categories: blog
tags: [java,springmvc]
description: 关于SpringMVC前台日期作为实体类对象参数类型转换错误解决。
---

#### 关于SpringMVC前台日期作为实体类对象参数类型转换错误解决

异常信息：

```
Field error in object 'invoiceItem' on field 'invoiceDate': rejected value [2018-01-29]; codes [typeMismatch.invoiceItem.invoiceDate,typeMismatch.invoiceDate,typeMismatch.java.util.Date,typeMismatch]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [invoiceItem.invoiceDate,invoiceDate]; arguments []; default message [invoiceDate]]; default message [Failed to convert property value of type 'java.lang.String' to required type 'java.util.Date' for property 'invoiceDate'; nested exception is org.springframework.core.convert.ConversionFailedException: Failed to convert from type [java.lang.String] to type [@com.baomidou.mybatisplus.annotations.TableField java.util.Date] for value '2018-01-29'; nested exception is java.lang.IllegalArgumentException]

```
异常场景描述:

前端输入框为选择日期的插件，选择日期完成，请求后台添加数据，报错。

解决方式

在实体类中的属性上添加该注解，即可解决该错误

   
```
 @DateTimeFormat(pattern = "yyyy-MM-dd") 
 private Date invoiceDate; 
```

参考文章：
[http://blog.csdn.net/pyfysf/article/details/79124189](http://blog.csdn.net/pyfysf/article/details/79124189)
