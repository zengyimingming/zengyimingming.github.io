---
layout: post
title:  "Spring Boot启动加载数据"
imges: 
date:   2021-01-20 10:00:00 +0800
tags: java
description: 学习笔记
---

我们在使用SpringBoot搭建项目的时候，如果希望在项目启动完成之前，能够初始化一些操作，针对这种需求，可以考虑实现如下两个接口（任一个都可以）

```
org.springframework.boot.CommandLineRunner
org.springframework.boot.ApplicationRunner
```

例子：

### CommandLineRunner

```java
@Component
public class TestCommandLineRunner implements CommandLineRunner {
 
    @Override
    // 实现该接口之后，实现其run方法，可以在run方法中自定义实现任务
    public void run(String... args) throws Exception {
        System.out.println("服务启动，TestCommandLineRunner执行启动加载任务...");
    }
}
```

### ApplicationRunner

```java
@Component
public class TestApplicationRunner implements ApplicationRunner {
 
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("服务启动，TestApplicationRunner执行启动加载任务...");
    }
}
```