---
layout: post
title:  "Spring Boot项目启动读取配置文件"
imges: 
date:   2021-01-18 10:00:00 +0800
tags: java
description: 学习笔记
---

###  PropertiesConfig.java 仅个人学习开发使用

```
package com.test.config;

import org.springframework.beans.factory.config.PropertiesFactoryBean;
import org.springframework.boot.context.event.ApplicationReadyEvent;
import org.springframework.context.ApplicationEvent;
import org.springframework.context.ApplicationListener;
import org.springframework.core.io.Resource;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.stereotype.Component;

import java.util.HashMap;
import java.util.Map;
import java.util.Properties;

@Component
public class PropertiesConfig implements ApplicationListener {
    //2.保存加载配置参数
    private static Map<String, String> resourcePropertiesMap = new HashMap<String, String>();

    //3.获取配置参数值 提供外部程序调用 用于获取配置文件的键值
    public static String getKey(String key) {
        return resourcePropertiesMap.get(key);
    }

    //1.监听启动完成，执行配置加载到resourcePropertiesMap
    @Override
    public void onApplicationEvent(ApplicationEvent event) {
        if(event instanceof ApplicationReadyEvent){
            this.init(resourcePropertiesMap);
        }
    }

    //初始化加载resource.properties
    public void init(Map<String , String> map)  {
        // 获得PathMatchingResourcePatternResolver对象
        PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
        try {
        //加载resource文件(也可以加载resources)
            Resource resources = resolver.getResource("classpath:config/resource.properties");
            PropertiesFactoryBean config = new PropertiesFactoryBean();
            config.setLocation(resources);
            config.afterPropertiesSet();
            Properties prop = config.getObject();
        //循环遍历所有得键值对并且存入集合
            for (String key : prop.stringPropertyNames()) {
                map.put(key, (String) prop.get(key));
            }
        } catch (Exception e) {
            new Exception("resource.properties配置文件加载失败");
        }

    }
}
```


