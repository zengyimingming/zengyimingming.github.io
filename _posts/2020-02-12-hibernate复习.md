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
	
hibernate day02

### 一.hibernate中的实体创建规则
	1>对象必须有oid.
	2>对象中的属性,尽量使用包装类型
	3>不使用final修饰类
	4>提供get/set方法....

### 二.hibernate主键生成策略(7种)
	increment: 查询最大值.再加1
	identity: 主键自增.
	sequence:Oracle使用的
	hilo: hibernate自己实现自增算法
	native: 根据所选数据库三选一
	uuid: 随机字符串
	assigned: 自然主键.

### 三.对象的三种状态
	瞬时状态
		没有id,没有在session缓存中.
	持久化状态
		有id,再session缓存中。
	托管|游离状态
		有id,不在session缓存中.
		
	持久化: 持久化状态的对象,会在事务提交时,自动同步到数据库中.
			我们使用hibernate的原则.就是将对象转换为持久化状态.

### 四.一级缓存
	缓存: 为了提高效率.
	一级缓存:为了提高效率.session对象中有一个可以存放对象的集合.
	
	查询时: 第一次查询时.会将对象放入缓存.再次查询时,会返回缓存中的.不再查询数据库.
	
	修改时: 会使用快照对比修改前和后对象的属性区别.只执行一次修改.
	
	
### 五.事务管理
	1>如何配置数据库隔离级别
		1	读未提交
		2	读已提交
		4	可重复读
		8	串行化
	2>指定session与当前线程绑定
		hibernate.current_session_context_class	thread
		
### 六.批量查询
	HQL		面向对象的语句查询
	Criteria	面向对象的无语句查询
	SQL		原生SQL	

## 多表关系

### 一对多/多对一
	O 对象			一的一方使用集合. 多的一方直接引用一的一方.
	R 关系型数据库	多的一方使用外键引用一的一方主键.
	M 映射文件		一: <set name="">
							<key column="" />
							<one-to-many class="" />
						</set>
					多: <many-to-one name="" column="" class="" />
	
	操作: 操作管理级别属性. 
	
		cascade: 级联操作	
			减少我们书写的操作代码.
			none(默认值)	不级联
			save-update:	级联保存
			delete:			级联删除
			all:			级联保存+级联删除
		结论: 可以使用save-update.不推荐使用delete. 也可以不用cascade.
		inverse: 反转关系维护
			属于性能优化.关系的两端如果都书写了关系.那么两方都会发送维护关系的语句.
			这样,语句就发生重复.我们可以使用inverse使一的一方放弃维护关系.
			true			放弃
			false(默认值)	维护
		结论: 在一对多中,一的一方可以放弃维护关系.
			


### 多对多
	O 对象			两方都使用集合. 
	R 关系型数据库	使用中间表.至少两列.作为外键引用两张表的主键.
	M 映射文件		多: <set name="" table="中间表名" >
							<key column="别人引用我" />
							<man-to-many class="" column="我引用别人的" />
						</set>
						
	操作:操作管理级别属性. 
		cascade: 级联操作	
			减少我们书写的操作代码.
			none(默认值)	不级联
			save-update:	级联保存
			delete:			级联删除
			all:			级联保存+级联删除
		结论: 可以使用save-update.不推荐使用delete. 也可以不用cascade.
		inverse: 反转关系维护
			属于性能优化.必须选择一方放弃维护主键关系.哪方放弃要看业务方向.