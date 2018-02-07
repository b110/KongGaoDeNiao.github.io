---
layout: post
title: Dubbo启动报javassist.ClassPath org.jboss.netty.channel.ChannelFactory异常
date: 2018-01-19
categories: blog
tags: [dubbo]
description: Dubbo启动报javassist.ClassPath org.jboss.netty.channel.ChannelFactory异常。
---

#### Dubbo 启动报javassist.ClassPath org.jboss.netty.channel.ChannelFactory异常   

# 一. javassist/ClassPath

在使用Dubbo的时候，启动报错如下：
```
aused by: java.lang.IllegalStateException: Failed to load extension class(interface: interface com.alibaba.dubbo.common.compiler.Compiler, class line: com.alibaba.dubbo.common.compiler.support.JavassistCompiler) in jar:file:/F:/My_WorkSpace/repository/com/alibaba/dubbo/2.5.3/dubbo-2.5.3.jar!/META-INF/dubbo/internal/com.alibaba.dubbo.common.compiler.Compiler, cause: javassist/ClassPath
    at com.alibaba.dubbo.common.extension.ExtensionLoader.loadFile(ExtensionLoader.java:685)
    at com.alibaba.dubbo.common.extension.ExtensionLoader.loadExtensionClasses(ExtensionLoader.java:591)
    at com.alibaba.dubbo.common.extension.ExtensionLoader.getExtensionClasses(ExtensionLoader.java:567)
    at com.alibaba.dubbo.common.extension.ExtensionLoader.getAdaptiveExtensionClass(ExtensionLoader.java:728)
    at com.alibaba.dubbo.common.extension.ExtensionLoader.createAdaptiveExtension(ExtensionLoader.java:721)
    at com.alibaba.dubbo.common.extension.ExtensionLoader.getAdaptiveExtension(ExtensionLoader.java:455)
    at com.alibaba.dubbo.common.extension.ExtensionLoader.createAdaptiveExtensionClass(ExtensionLoader.java:738)
    at com.alibaba.dubbo.common.extension.ExtensionLoader.getAdaptiveExtensionClass(ExtensionLoader.java:732)
    at com.alibaba.dubbo.common.extension.ExtensionLoader.createAdaptiveExtension(ExtensionLoader.java:721)
    ... 27 more
Caused by: java.lang.NoClassDefFoundError: javassist/ClassPath
    at java.lang.Class.forName0(Native Method)
    at java.lang.Class.forName(Class.java:274)
    at com.alibaba.dubbo.common.extension.ExtensionLoader.loadFile(ExtensionLoader.java:627)
    ... 35 more
```
解决，缺失jar：

```
<!--javassist jar-->
        <dependency>
            <groupId>org.javassist</groupId>
            <artifactId>javassist</artifactId>
            <version>3.18.1-GA</version>
        </dependency>
```
# 二. org/jboss/netty/channel/ChannelFactory
在使用Dubbo的时候，启动报错如下：


```
Exception in thread "main" java.lang.NoClassDefFoundError: org/jboss/netty/channel/ChannelFactory
    at com.alibaba.dubbo.remoting.transport.netty.NettyTransporter.bind(NettyTransporter.java:33)
    at com.alibaba.dubbo.remoting.Transporter$Adpative.bind(Transporter$Adpative.java)
    at com.alibaba.dubbo.remoting.Transporters.bind(Transporters.java:48)
    at com.alibaba.dubbo.remoting.exchange.support.header.HeaderExchanger.bind(HeaderExchanger.java:41)
    at com.alibaba.dubbo.remoting.exchange.Exchangers.bind(Exchangers.java:63)
    at com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtocol.createServer(DubboProtocol.java:287)
    at com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtocol.openServer(DubboProtocol.java:266)
    at com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtocol.export(DubboProtocol.java:253)
    at com.alibaba.dubbo.rpc.protocol.ProtocolListenerWrapper.export(ProtocolListenerWrapper.java:56)
    at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper.export(ProtocolFilterWrapper.java:55)
    at com.alibaba.dubbo.rpc.Protocol$Adpative.export(Protocol$Adpative.java)
    at com.alibaba.dubbo.registry.integration.RegistryProtocol.doLocalExport(RegistryProtocol.java:153)
    at com.alibaba.dubbo.registry.integration.RegistryProtocol.export(RegistryProtocol.java:107)
    at com.alibaba.dubbo.rpc.protocol.ProtocolListenerWrapper.export(ProtocolListenerWrapper.java:54)
    at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper.export(ProtocolFilterWrapper.java:53)
    at com.alibaba.dubbo.rpc.Protocol$Adpative.export(Protocol$Adpative.java)
    at com.alibaba.dubbo.config.ServiceConfig.doExportUrlsFor1Protocol(ServiceConfig.java:485)
    at com.alibaba.dubbo.config.ServiceConfig.doExportUrls(ServiceConfig.java:281)
    at com.alibaba.dubbo.config.ServiceConfig.doExport(ServiceConfig.java:242)
    at com.alibaba.dubbo.config.ServiceConfig.export(ServiceConfig.java:143)
    at com.alibaba.dubbo.config.spring.ServiceBean.onApplicationEvent(ServiceBean.java:109)
    at org.springframework.context.event.SimpleApplicationEventMulticaster.multicastEvent(SimpleApplicationEventMulticaster.java:96)
    at org.springframework.context.support.AbstractApplicationContext.publishEvent(AbstractApplicationContext.java:334)
    at org.springframework.context.support.AbstractApplicationContext.finishRefresh(AbstractApplicationContext.java:948)
    at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:482)
    at org.springframework.context.support.ClassPathXmlApplicationContext.<init>(ClassPathXmlApplicationContext.java:139)
    at org.springframework.context.support.ClassPathXmlApplicationContext.<init>(ClassPathXmlApplicationContext.java:93)
    at com.dufy.service.ProviderServiceTest.main(ProviderServiceTest.java:21)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:606)
    at com.intellij.rt.execution.application.AppMain.main(AppMain.java:147)
Caused by: java.lang.ClassNotFoundException: org.jboss.netty.channel.ChannelFactory
    at java.net.URLClassLoader$1.run(URLClassLoader.java:366)
    at java.net.URLClassLoader$1.run(URLClassLoader.java:355)
    at java.security.AccessController.doPrivileged(Native Method)
    at java.net.URLClassLoader.findClass(URLClassLoader.java:354)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:425)
    at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:308)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:358)
    ... 33 more
```

解决，Dubbo依赖Netty，添加jar：


```
<!--netty jar-->
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty</artifactId>
            <version>3.6.10.Final</version>
        </dependency>
```


[转载](http://blog.csdn.net/u010648555/article/details/78015518)













