---
layout: post
title: 遍历HttpServletRequest获取请求参数
date: 2017-10-23
categories: blog
tags: [java,HttpServletRequest]
description: 遍历HttpServletRequest获取请求参数。
---

##遍历HttpServletRequest获取请求参数
###以方便我们排查错误
```
public List<Map<String, Object>> list(HttpServletRequest request) {
		Enumeration em = request.getParameterNames();
		 while (em.hasMoreElements()) {
		    String name = (String) em.nextElement();
		    String value = request.getParameter(name);
		    System.out.println(name+"\t"+value);
		}
}
```












