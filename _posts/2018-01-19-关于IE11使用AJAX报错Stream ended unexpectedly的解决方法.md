---
layout: post
title: 关于IE11使用AJAX报错Stream ended unexpectedly的解决方法
date: 2018-01-19
categories: blog
tags: [疑难杂症]
description: 关于IE11使用AJAX报错Stream ended unexpectedly的解决方法。
---

## 关于IE 使用AJAX报错：“Stream ended unexpectedly”的解决方法

```
天津吃饭大学交费报考页面有一个表单提交页面有checkbox、radio、select、file等 使用AJAX的FormData将form里面的东西转换序列化传送至后台。

以上的操作在firefox和Chrome都能正常通过，但是在IE上面却是一闪而过没有正常登录。后端就报错了。

估计是Ajax的FormData和IE不兼容的问题

```
界面：
![image](https://note.youdao.com/yws/api/personal/file/59CCE2525D4E4799AA67D1C10A96D496?method=download&shareKey=0b9f15d54779b966e245d0db03c10fd0)
ie11 选第二个报考方向正常报考，选第一个报考方向，点保存没反应也没提示
小波 2018/1/18 14:33:13
其他浏览器正常

JavaScript:

```
	 var formData = new FormData($("#addbkForm")[0]);
		 $.ajax({
				url : 'student_saveBkzy.htm',
				type : 'POST',
				data : formData,
				async : false,
				cache : false,
				contentType : false,
				processData : false,
				success : function(data) {
					var result = data.result;
					if (result == 1) {
						showAlertCallback('报考成功',alertCallback);
						function alertCallback(){
							window.location.reload(true);
						}
					} else {
						showErrorAlert2(data.message);
					}
				},
				complete : function() {
					closeLoading();
				}
			}); 
```

报错的内容：


```
10:36:16,282 DEBUG DispatcherServlet:989 - Could not complete request  
org.springframework.web.multipart.MultipartException: Could not parse multipart servlet request; nested exception is org.apache.commons.fileupload.FileUploadException: Stream ended unexpectedly  
    at org.springframework.web.multipart.commons.CommonsMultipartResolver.parseRequest(CommonsMultipartResolver.java:165)  
    at org.springframework.web.multipart.commons.CommonsMultipartResolver.resolveMultipart(CommonsMultipartResolver.java:142)  
    at org.springframework.web.servlet.DispatcherServlet.checkMultipart(DispatcherServlet.java:1089)  
    at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:928)  
    at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:893)  
    at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:966)  
    at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:868)  
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:648)  
    at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:842)  
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:729)  
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:291)  
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:206)  
    at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)  
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:239)  
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:206)  
    at org.apache.shiro.web.servlet.ProxiedFilterChain.doFilter(ProxiedFilterChain.java:61)  
    at org.apache.shiro.web.servlet.AdviceFilter.executeChain(AdviceFilter.java:108)  
    at org.apache.shiro.web.servlet.AdviceFilter.doFilterInternal(AdviceFilter.java:137)  
    at org.apache.shiro.web.servlet.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:125)  
    at org.apache.shiro.web.servlet.ProxiedFilterChain.doFilter(ProxiedFilterChain.java:66)  
    at org.apache.shiro.web.servlet.AbstractShiroFilter.executeChain(AbstractShiroFilter.java:449)  
    at org.apache.shiro.web.servlet.AbstractShiroFilter$1.call(AbstractShiroFilter.java:365)  
    at org.apache.shiro.subject.support.SubjectCallable.doCall(SubjectCallable.java:90)  
    at org.apache.shiro.subject.support.SubjectCallable.call(SubjectCallable.java:83)  
    at org.apache.shiro.subject.support.DelegatingSubject.execute(DelegatingSubject.java:383)  
    at org.apache.shiro.web.servlet.AbstractShiroFilter.doFilterInternal(AbstractShiroFilter.java:362)  
    at org.apache.shiro.web.servlet.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:125)  
    at org.springframework.web.filter.DelegatingFilterProxy.invokeDelegate(DelegatingFilterProxy.java:344)  
    at org.springframework.web.filter.DelegatingFilterProxy.doFilter(DelegatingFilterProxy.java:261)  
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:239)  
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:206)  
    at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:85)  
    at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)  
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:239)  
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:206)  
    at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:212)  
    at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:106)  
    at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:502)  
    at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:141)  
    at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:79)  
    at org.apache.catalina.valves.AbstractAccessLogValve.invoke(AbstractAccessLogValve.java:616)  
    at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:88)  
    at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:521)  
    at org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1096)  
    at org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:674)  
    at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1500)  
    at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.run(NioEndpoint.java:1456)  
    at java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)  
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)  
    at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)  
    at java.lang.Thread.run(Unknown Source)  
Caused by: org.apache.commons.fileupload.FileUploadException: Stream ended unexpectedly  
    at org.apache.commons.fileupload.FileUploadBase.parseRequest(FileUploadBase.java:369)  
    at org.apache.commons.fileupload.servlet.ServletFileUpload.parseRequest(ServletFileUpload.java:126)  
    at org.springframework.web.multipart.commons.CommonsMultipartResolver.parseRequest(CommonsMultipartResolver.java:158)  
    ... 50 more  
```

别人分析BUG思路：

```
Bug问题定位:

一开始看到这个错误我就一脸懵逼了，我这个什么时候有上传文件的东西呢？首先是各种场景测试一遍，checkbox勾选和不勾选上都不能通过，还是报错。然后认真看看前端的代码，发现checkBox那边没有name的属性，手贱补充上去。再测试下各种场景传送至后端的数据。

没有勾选checkbox的状态


一开始看还是挺正常的啊，没有问题啊但还是继续报错。然后试试checkbox打钩上。


奇迹发生了，竟然可以正常登录。多次测试之后发觉只有checkBox勾选上了才能正常登录。

在网上搜索了一遍各种中文资料，描述我这样子的问题比较少。然后再stackoverflow上面看到一个问题的描述和我的差不多，也是IE所遇到的问题。然后按照下面的说法在末端增加一个隐藏的属性<input>这样子就可以正常通过了。

原因分析：

参照国外的网站资料了解到这个锅不是jQuery的锅，我觉得应该是IE的锅啦。其他两个浏览器都没有这个问题，怎么就你IE这么矫情出这个错误呢。

只要你所使用的form最后一个元素是checkBox、Radio之类的没有勾选上的话，使用FormData进行转换就会发生这个错误。
```


我的分析思路：


```
IE兼容性问题  没了￣□￣｜｜

百度 百度  发现国外一个网站有这个问题的解决如下图，奈何看不懂我草！！

继续百度

```
![image](https://note.youdao.com/yws/api/personal/file/1CA0736DC17E4B818E7A34C601AD0B87?method=download&shareKey=61b715224f6c835c5c810b176bfb2378)

别人的解决方案：

```
参照国外的网站资料了解到这个锅不是jQuery的锅，我觉得应该是IE的锅啦。其他两个浏览器都没有这个问题，怎么就你IE这么矫情出这个错误呢。

只要你所使用的form最后一个元素是checkBox、Radio之类的没有勾选上的话，使用FormData进行转换就会发生这个错误。

解决方法：

只需要在form的最后添加一个隐藏的属性用来做最后的元素就可以了。所以我就再记住我的下面添加了一个hidden的input了。

至此问题顺利解决了。
PS：我所使用的IE版本为IE11，其他较老的版本是否这样子解决办法暂时没有试过。
```
我的解决方案：
```
添加一个隐藏域 好使了！！好使了！！  66666
<input type="hidden" name="test">  
```



参照资料：
- http://blog.csdn.net/hanchao_h/article/details/54171267
- http://stackoverflow.com/questions/27903414/ie-11-error-while-sending-multipart-form-data-request-stream-ended-unexpectedl
- https://blog.yorkxin.org/2014/02/06/ajax-with-formdata-is-broken-on-ie10-ie11











