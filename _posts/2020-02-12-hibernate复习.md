---
layout: post
title:  "Hibernate-复习笔记"
imges: 
date:   2019-11-23 11:00:00 +0800
description: 学习笔记
---

## hibernate框架

### 1.什么是框架
	提高我们的开发效率.可以理解成是一个半成品项目.
### 2.hibernate框架
	dao层框架
	操作数据库.
	以面向对象的方式操作数据库.
	orm 思想. 对象关系映射. 通过映射文件配置对象与数据库中表的关系.

### 3.hibernate框架搭建
	1> 导包
		required+驱动包
		
	2> 准备实体类 以及 orm元数据
	
	3> 创建主配置文件
	
	4>书写代码测试
	
### 4.配置文件详解
	orm元数据(xxx.hbm.xml)
		<hibernate-mapping package="">
			<class name table>
				<id name >
					<generator class="">
				</id>
				<property name="" />
		 
	hibernate.cfg.xml
		必选配置
			4+1 方言
		可选配置
			显示sql
			格式化sql
			自动生成表
				|- update
		orm元数据引入
			<mapping resource="" />
	
### 5.api详解
	Configuration 	读取配置
	sessionFactory	创建session
	Session			获得事务操作对象,以及数据增删改查
	Transaction		控制事务
	
	
