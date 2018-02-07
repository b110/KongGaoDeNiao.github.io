---
layout: post
title: Springboot与Quartz用数据库管理集成
date: 2017-12-08
categories: blog
tags: [springboot,quartz]
description: Springboot与Quartz用数据库管理集成。
---

### Springboot与Quartz用数据库管理集成遇到的坑（包你好使）！！Service无法注入 null
#### 近期项目需要加上定时任务，原先使用springboot自带的 schedule。但是这个没法集成表直接在线进行修改cron、启用、禁用任务。
#### 大坑一：集成完Quartz后，发现在注入Service时注入不进去。一直报空指针异常。百度上一大片这样的问题的解决，但是都说的模棱两可。我整整试了一个通宵也没有结果，把百度上所有文章都看了一遍也不好使。妈了个鸡，最后没辙了放弃了。将问题反馈给了技术老大（10年大牛），最后问题完美解决。。。。不服不行，我墙都不服。
#### 首先你要知道：使用spring-boot框架，其理念为零配置文件，所有的配置都是基于注解和暴露bean的方式。Spring容器可以管理Bean，但是**Quartz的job是自己管理**的，如果在Job中注入Spring管理的Bean，需要先把Quartz的Job也让Spring管理起来，因此，我们需要**重写JobFactory**。
```
源码分析可以参考这，具体原理知道就行，因为这并没有什么卵用。
```
http://www.cnblogs.com/daxin/p/3608320.html
http://blog.csdn.net/a67474506/article/details/38402059

```
参考文章，这只是一部分百度上所有的都看遍了。跪谢！！！
```
https://www.cnblogs.com/javanoob/p/springboot_schedule.html
https://www.cnblogs.com/softidea/p/6073495.html
http://blog.csdn.net/magic_best/article/details/50158125
http://blog.csdn.net/iycynna_123/article/details/52993542
http://www.jianshu.com/p/b460171c57ea


# 周末再写吧 忙死了  ！！












