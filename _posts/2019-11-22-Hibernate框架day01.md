---
layout: post
title:  "Hibernate-day01"
imges: 
date:   2019-11-22 11:00:00 +0800
description: 学习笔记
---

# hibernate是什么   
### 1 框架是什么
    1.框架是用来提高开发效率的
    2.封装了好了一些功能.我们需要使用这些功能时,调用即可.不需要再手动实现.
    3.所以框架可以理解成是一个半成品的项目.只要懂得如何驾驭这些功能即可.    
    
### 2 hibernate框架是什么
<img src="/images/xmind/hibernate框架.jpg" width="100%">


### 3 hibernate的好处   
* 操作数据库的时候 ，可以以面向对象的方式来完成，不需要书写SQL语句      

### 4 hibernate是一款orm框架
* orm:object relationg mapping. 对象关系映射
* <img src="/images/xmind/orm.jpg" width="100%">  

*  orm分4级    
    1. hibernate属于4级:完全面向对象操作数据库
    2. mybatis属于2级
    3. dbutils属于1级

<hr />

# hibernate框架的搭建   
###  1. 导包
需要的jar包
<img src="/images/xmind/1122_01.png" width="100%">  
连接数据库的包  
<img src="/images/xmind/1122_02.png" width="100%"> 

###  2. 创建数据库,准备表,实体
<img src="/images/xmind/1122_03.png" width="100%"> 

###  3. 书写orm元数据(对象与表的映射配置文件)
#### 导入约束
<img src="/images/xmind/1122_04.png" width="100%">

#### 实体   
<img src="/images/xmind/1122_05.png" width="100%">  

#### orm元数据
<img src="/images/xmind/1122_06.png" width="100%"> 

###  4. 书写主配置文件
<img src="/images/xmind/1122_07.png" width="100%"> 

###  5.书写代码测试
<img src="/images/xmind/1122_08.png" width="100%"> 
