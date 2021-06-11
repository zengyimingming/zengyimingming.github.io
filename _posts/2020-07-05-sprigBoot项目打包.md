---
layout: post
title:  "SpringBoot项目打包部署在linux上，实现win客户端浏览器访问"
imges: 
date:   2020-07-05 10:04:00 +0800
description: 
tags: java
---


前提：windows客户端能ping通自己的虚拟机，可cmd测试能否ping通，
在这里先推荐两款软件

1.Xshell6:实现远程访问你的虚拟机的工具
2.winscp：可以将windows本地的文件传输到你的虚拟机上

下载链接，按住Ctrl+鼠标左键点击
[Xshell6](https://pc.qq.com/search.html#!keyword=xshell6)   
[winscp](https://pc.qq.com/search.html#!keyword=winscp)

可以直接下载安装在windows本地即可，安装方式和你安装qq一样，一直点击下一步即可

安装了上面两个软件之后呢，笔者在这里给了你此次需要的jdk和tomcat，请使用百度网盘下载在你的Windows本地，后面的步骤需要用到你这里下载的东西。

> 简单的springboot项目搭建 在这里可以不用装我下面步骤里的tomcat  使用内置的即可，所以你直接看步骤1 步骤3

#### 百度网盘：
~~~
链接：https://pan.baidu.com/s/14cpSOBaL65-ux2WcoBl-tg 
提取码：87e3
请一定下载
~~~
#### 1.安装jdk1.8
在这里你需要先将jdk传递到你的centos上，如下图1
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200629203427762.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjY4MjgwMA==,size_16,color_FFFFFF,t_7)
<center style="font-size:14px;color:#C0C0C0;text-decoration:underline"> 图 1 传输jdk到虚拟机上</center> 


 

传输到了虚拟机中，接着去查看你上面传输的jdk的包，笔者的如下图2
 ![这是啥](https://img-blog.csdnimg.cn/20200629203446465.png)

<center style="font-size:14px;color:#C0C0C0;text-decoration:underline"> 图2. 查看自己的jdk压缩包</center> 

**/root/Desktop**这个路径就是笔者使用远程工具**winscp**传输过来的，你只需要记住自己的路径即可。不必和我的相同。
知道自己的jdk的压缩包在哪里之后，接着就是安装了
在这里笔者建立一个文件夹
路径是在 /usr下新建一个文件夹java1.8
命令如下：
~~~
1.	cd  
2.	mkdir /usr/java1.8  
~~~

创建好这个文件夹后，如下图
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/2020062920352290.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjY4MjgwMA==,size_16,color_FFFFFF,t_70)

<center style="font-size:14px;color:#C0C0C0;text-decoration:underline">图 3创建java1.8目录</center> 



创建好这个目录java1.8之后，将之前通过远程工具传输的jdk的压缩包复制到这个目录下，并解压即可。
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200629203533941.png)

<center style="font-size:14px;color:#C0C0C0;text-decoration:underline">图 4将jdk1.8压缩包复制到上面新建的java1.8中</center> 


~~~
解压：tar -zxf jdk-8u211-linux-x64.tar.gz
~~~
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200629203540437.png)

<center style="font-size:14px;color:#C0C0C0;text-decoration:underline">图 5解压jdk压缩包</center> 

解压完毕后你可以看到这里多了个目录
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200629203547678.png)

<center style="font-size:14px;color:#C0C0C0;text-decoration:underline">图 6查看解压的结果</center> 

解压后，你进入这个目录，然后输入pwd查看当前的路径，笔者的路径如下
/usr/java1.8/jdk1.8.0_211

这个就是你的jdk的路径，一定要copy下，下面的步骤需要用到
到这一步之后，你还需要完成一个配置文件
~~~
[root@localhost ~]# vim /etc/profile
~~~
进入这个文件之后
在文件的最后一行加上下面几行
~~~
#java environment
export JAVA_HOME=/usr/java1.8/jdk1.8.0_211
export CLASSPATH=.:${JAVA_HOME}/jre/lib/rt.jar:${JAVA_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar
export PATH=$PATH:${JAVA_HOME}/bin
~~~
表格中的jdk的路径就是上面笔者的路径，你需要查看自己的路径，不能直接和笔者雷同
好的到了这一步之后，jdk就安装完毕了
现在就去查看安装是否成功吧
~~~
[root@yiming java1.8]#source /etc/profile
[root@yiming java1.8]# java -version
~~~
结果如下即表示安装成功
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200629203558312.png)

<center style="font-size:14px;color:#C0C0C0;text-decoration:underline">图 7查看jdk</center> 


#### 2.安装tomcat7,springboot内置了tomcat，这里你可以先跳过tomcat7安装
安装tomcat的步骤和jdk非常相似
先将Windows本地的Apache-tomcat的压缩包传输到centos，笔者依然是传输到/root/Desktop目录下
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/2020062920360698.png)

<center style="font-size:14px;color:#C0C0C0;text-decoration:underline">图 8 传输tomcat到centos</center> 



传输完毕后，笔者在/usr目录下新建一个目录tomcat7
~~~
[root@yiming ~]# mkdir /usr/tomcat7
~~~

新建这个目录完毕后，将上面传输的apache-toncat的压缩包复制到这个目录下，并解压

进入/root/Desktop目录
~~~
[root@yiming Desktop]# cp apache-tomcat-7.0.47.tar.gz /usr/tomcat7/
~~~
进入/usr/tomcat7目录
~~~
[root@yiming tomcat7]# tar -zxf apache-tomcat-7.0.47.tar.gz
~~~
解压完毕后如下图
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200629203613837.png)

<center style="font-size:14px;color:#C0C0C0;text-decoration:underline">图 9解压tomcat</center> 


现在解压完毕后，进入apace-tomcat-7.0.47目录，
执行pwd,查看当前的tomcat 的路径，笔者的路径如下，这个路径也很重要
~~~
/usr/tomcat7/apache-tomcat-7.0.47
~~~
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200629203621482.png)

<center style="font-size:14px;color:#C0C0C0;text-decoration:underline">图 10 tomcat安装路径</center> 


记住这个路径后，
跟着笔者的命令走
~~~
[root@yiming ~]# vim /etc/profile
~~~
打开后 在最后一行添加如下代码
~~~
export CATALINA_HOME=/usr/tomcat7/apache-tomcat-7.0.47
~~~
这样就完成了

#### 3.打包你的springboot项目
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200629203629600.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjY4MjgwMA==,size_16,color_FFFFFF,t_70)

<center style="font-size:14px;color:#C0C0C0;text-decoration:underline">图 11springboot项目打包</center> 


右击图中package，在弹出层选择第一项Run Maven Build
等待完成，去找到你的jar包
笔者的路径是
D:\IdeaProjects\transaction\target
在target路径下找到jar包

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200629203638979.png)

<center style="font-size:14px;color:#C0C0C0;text-decoration:underline">图 12查看生成的jar包</center> 


将这个包使用winscp工具传输到centos中
笔者不再重复演示传输了
传输到笔者的/root/Desktop路径下

建议先关闭防火墙  Centos7关闭防火墙
~~~
systemctl stop firewalld.service
~~~
关闭后，进入到你传输的jar包的路径下
笔者就进入/root/Desktop目录下
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200629203647945.png)

<center style="font-size:14px;color:#C0C0C0;text-decoration:underline">图 13查看centos上的jar文件</center> 


启动服务
~~~
[root@yiming ~]# java -jar transaction-0.0.1-SNAPSHOT.jar
~~~
浏览器访问下面地址
[http://39.105.116.117:8080/swagger-ui.html](http://39.105.116.117:8080/swagger-ui.html)
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200629203653661.png)

<center style="font-size:14px;color:#C0C0C0;text-decoration:underline">图 14访问服务</center> 




