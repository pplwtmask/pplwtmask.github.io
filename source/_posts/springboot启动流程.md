---
title: springboot启动流程
date: 2021-09-29 15:38:06
---


### springboot启动流程
观察者设计模式：定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。
springboot启动是以发布订阅的方式，即观察者设计模式启动，application的状态发生改变所有的监听器都会更新

```
public interface SpringApplicationRunListener {

	void starting();

	void environmentPrepared(ConfigurableEnvironment environment);

	void contextPrepared(ConfigurableApplicationContext context);

	void contextLoaded(ConfigurableApplicationContext context);

	void started(ConfigurableApplicationContext context);

	void running(ConfigurableApplicationContext context);

	void failed(ConfigurableApplicationContext context, Throwable exception);

}
```


run()
|
|---->创建SpringApplication---实例化ApplicationContextInitializer、ApplicationListener
|
发布事件：ApplicationStartingEvent
|
|---->创建Environment
|
发布事件：ApplicationEnvironmentPreparedEvent
|
|---->1.打印banner2.创建spring上下文（root）3.调用ApplicationContextInitializer（初始化上下文）
|
发布事件：ApplicationContextInitializedEvent
|
|---->1.注册bean定义2.给ApplicationListener中实现ApplicationContextAware接口的类设置上下文
|
发布事件：ApplicationPreparedEvent
|
|---->1.注册ShutDownHook2.刷新上下文 |afterRefresh()为空实现，可以复写
|
发布事件：ApplicationStartedEvent
|
|---->1.运行ApplicationRunner2.运行CommandLineRunner
|
发布事件：ApplicationReadyEvent
|
|
|
结束

