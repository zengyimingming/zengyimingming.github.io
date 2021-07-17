---
layout: post
title: "Hutool简单导出excel文件"
imges: 
date:  2021-07-17 10:00:00 +0800
description: 
tags: java SpringBoot excel
---





之前做excel导出有使用过Apache POI  和 EasyExcel，当然一般都是封装成了工具类

今天主要是记录下笔记，使用Hutool封装好的工具做一个简单的excel导出，个人体验还行哈。

### 步骤1 新建一个springboot项目，导入依赖

```xml
 		<!--web-测试浏览器发起请求-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--导出excel所需糊涂依赖-->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.0.7</version>
        </dependency>
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>4.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml-schemas</artifactId>
            <version>4.1.2</version>
        </dependency>
```



### 步骤2 新建一个User类

```java
public class User {
    //get和set方法请自行添加 或使用lombok
    private String userName;
    private String age;
    private Date birthDay;
	
    public User(String userName, String age, Date birthDay) {
        this.userName = userName;
        this.age = age;
        this.birthDay = birthDay;
    }
}
```

### 步骤3 书写Controller

```java
package com.zeng.controller;

import cn.hutool.core.io.IoUtil;
import cn.hutool.poi.excel.ExcelUtil;
import cn.hutool.poi.excel.ExcelWriter;
import com.zeng.pojo.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServletResponse;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

/**
 * @Author: ZYM
 * @Date: 2021年07月17日 10:04
 * @Description:
 */
@Controller
@RequestMapping("/excel")
public class ExcelController {
    @GetMapping("/export")
    @ResponseBody
    public void export(HttpServletResponse response) {
        List<User> list = new ArrayList<>();
        list.add(new User("张三", "1231", new Date()));
        list.add(new User("zhangsan1", "1232", new Date()));
        list.add(new User("zhangsan2", "1233", new Date()));
        list.add(new User("zhangsan3", "1234", new Date()));
        list.add(new User("zhangsan4", "1235", new Date()));
        list.add(new User("zhangsan5", "1236", new Date()));
        // 通过工具类创建writer，默认创建xls格式 参数true 表示创建xlsx
        ExcelWriter writer = ExcelUtil.getWriter(true);
        //自定义标题别名
        writer.addHeaderAlias("userName", "姓名");
        writer.addHeaderAlias("age", "年龄");
        writer.addHeaderAlias("birthDay", "生日");
        // 合并单元格后的标题行，使用默认标题样式
        //writer.merge(2, "测试康康");
        // 一次性写出内容，使用默认样式，强制输出标题
        writer.write(list, true);
        //设置第三列的列宽 30
        writer.setColumnWidth(2,30);

        response.setContentType("application/vnd.ms-excel;charset=utf-8");
        ServletOutputStream out = null;
        try {
            //out为OutputStream，需要写出到的目标流
            //response为HttpServletResponse对象
            //test.xls是弹出下载对话框的文件名，不能为中文，中文请自行编码
            String name = "test";
            name = new String(name.getBytes("ISO-8859-1"), "utf-8");
            response.setHeader("Content-Disposition", "attachment;filename=" + name + ".xlsx");
            out = response.getOutputStream();
            writer.flush(out, true);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 关闭writer，释放内存
            writer.close();
        }
        //此处记得关闭输出Servlet流
        IoUtil.close(out);
    }
}

```

### 步骤4 发起get请求 

请求地址:  http://localhost:8080/excel/export 

![](https://gitee.com/zengyimingming/picrepo/raw/master/images/20210717110727.png)

查看下载好的excel

<img src="https://gitee.com/zengyimingming/picrepo/raw/master/images/20210717110850.png" style="zoom:80%;" />