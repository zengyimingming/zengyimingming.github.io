---
layout: post
title:  "springboot整合shiro"
imges: 
date:   2021-06-03 10:00:00 +0800
description: 若有侵权，请联系我删除
---


# 个人学习使用springboot代码参考

### 1.导入依赖


```xml
 <dependencies>
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-core</artifactId>
            <version>1.7.1</version>
        </dependency>

        <!-- configure logging -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>1.7.21</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jcl-over-slf4j</artifactId>
            <version>1.7.21</version>
        </dependency>

        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
    </dependencies>
```



依赖中最后一项加了log4j ，在这里笔者再加一个log4j的配置文件，以便看到日志信息

>  log4j.properties

```properties
log4j.rootLogger=INFO, stdout

log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m %n

# General Apache libraries
log4j.logger.org.apache=WARN

# Spring
log4j.logger.org.springframework=WARN

# Default Shiro logging
log4j.logger.org.apache.shiro=INFO

# Disable verbose logging
log4j.logger.org.apache.shiro.util.ThreadContext=WARN
log4j.logger.org.apache.shiro.cache.ehcache.EhCache=WARN
```



### 2. 配置shiro.ini

```ini
[users]
# user 'root' with password 'secret' and the 'admin' role
root = secret, admin
# user 'guest' with the password 'guest' and the 'guest' role
guest = guest, guest
# user 'presidentskroob' with password '12345' ("That's the same combination on
# my luggage!!!" ;)), and role 'president'
presidentskroob = 12345, president
# user 'darkhelmet' with password 'ludicrousspeed' and roles 'darklord' and 'schwartz'
darkhelmet = ludicrousspeed, darklord, schwartz
# user 'lonestarr' with password 'vespa' and roles 'goodguy' and 'schwartz'
lonestarr = vespa, goodguy, schwartz

# -----------------------------------------------------------------------------
# Roles with assigned permissions
#
# Each line conforms to the format defined in the
# org.apache.shiro.realm.text.TextConfigurationRealm#setRoleDefinitions JavaDoc
# -----------------------------------------------------------------------------
[roles]
# 'admin' role has all permissions, indicated by the wildcard '*'
admin = *
# The 'schwartz' role can do anything (*) with any lightsaber:
schwartz = lightsaber:*
# The 'goodguy' role is allowed to 'drive' (action) the winnebago (type) with
# license plate 'eagle5' (instance specific id)
goodguy = winnebago:drive:eagle5

```



### 3. Quickstart

```java
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.*;
import org.apache.shiro.config.IniSecurityManagerFactory;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.session.Session;
import org.apache.shiro.subject.Subject;
import org.apache.shiro.util.Factory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * @Author: ZYM
 * @Date: 2021年06月03日 13:19
 * @Description:
 */
public class Quickstart {
    private static final transient Logger log = LoggerFactory.getLogger(Quickstart.class);


    public static void main(String[] args) {
        Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
        SecurityManager securityManager = factory.getInstance();
        SecurityUtils.setSecurityManager(securityManager);

        // Now that a simple Shiro environment is set up, let's see what you can do:

        // get the currently executing user:
        //获取当前用户对象 subject
        Subject currentUser = SecurityUtils.getSubject();

        // Do some stuff with a Session (no need for a web or EJB container!!!)
        //通过当前用户拿到session
        Session session = currentUser.getSession();

        session.setAttribute("someKey", "aValue");
        String value = (String) session.getAttribute("someKey");
        if (value.equals("aValue")) {
            log.info("Subject-->session->value: [" + value + "]");
        }

        //判断当前用户是否被认证
        if (!currentUser.isAuthenticated()) {
            //Token 令牌
            UsernamePasswordToken token = new UsernamePasswordToken("lonestarr2", "vespa");
            token.setRememberMe(true);
            try {
                //执行登录操作--这里比较复杂
                currentUser.login(token);
            } catch (UnknownAccountException uae) {
                log.info("不存在此用户： " + token.getPrincipal());
            } catch (IncorrectCredentialsException ice) {
                log.info("密码不对： " + token.getPrincipal() + " was incorrect!");
            } catch (LockedAccountException lae) {
                log.info("The account for username " + token.getPrincipal() + " is locked.  " +
                        "Please contact your administrator to unlock it.");
            }
            // ... catch more exceptions here (maybe custom ones specific to your application?
            catch (AuthenticationException ae) {
                //unexpected condition?  error?
            }
        }

        //say who they are:
        //print their identifying principal (in this case, a username):
        log.info("User [" + currentUser.getPrincipal() + "] logged in successfully.");

        //test a role:
        if (currentUser.hasRole("schwartz")) {
            log.info("May the Schwartz be with you!");
        } else {
            log.info("Hello, mere mortal.");
        }

        //粗粒度的权限
        //test a typed permission (not instance-level)
        if (currentUser.isPermitted("lightsaber:wield")) {
            log.info("You may use a lightsaber ring.  Use it wisely.");
        } else {
            log.info("Sorry, lightsaber rings are for schwartz masters only.");
        }

        //细粒度的权限
        //a (very powerful) Instance Level permission:
        if (currentUser.isPermitted("winnebago:drive:eagle5")) {
            log.info("You are permitted to 'drive' the winnebago with license plate (id) 'eagle5'.  " +
                    "Here are the keys - have fun!");
        } else {
            log.info("Sorry, you aren't allowed to drive the 'eagle5' winnebago!");
        }

        //注销
        //all done - log out!
        currentUser.logout();

        System.exit(0);
    }
}

```



> 整体的结构如下

<img src="https://gitee.com/zengyimingming/picrepo/raw/master/images/20210603133415.png" style="zoom:50%;" />



==Quickstart中可以获取如下信息==

1. ```java
   1. Subject currentUser = SecurityUtils.getSubject();
   2. Session session = currentUser.getSession();
   3. currentUser.isAuthenticated();
   4. currentUser.getPrincipal();
   5. currentUser.hasRole("schwartz");
   6. currentUser.isPermitted("lightsaber:wield");
   7. currentUser.logout();
   ```



### springBoot 集成shiro

### 1.导入依赖

```xml
<properties>
        <java.version>1.8</java.version>
        <spring-boot.mybatis.version>2.1.3</spring-boot.mybatis.version>
        <mysql-connector>5.1.39</mysql-connector>
        <druid>1.0.18</druid>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.thymeleaf</groupId>
            <artifactId>thymeleaf-spring5</artifactId>
            <version>3.0.11.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.thymeleaf.extras</groupId>
            <artifactId>thymeleaf-extras-java8time</artifactId>
            <version>3.0.4.RELEASE</version>
        </dependency>

        <!--shiro整合spring-->
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-spring</artifactId>
            <version>1.7.1</version>
        </dependency>
        <!-- Spring Boot Mybatis 依赖 -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>${spring-boot.mybatis.version}</version>
        </dependency>
        <!-- MySQL 连接驱动依赖 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql-connector}</version>
        </dependency>
        <!-- Druid 数据连接池依赖 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>${druid}</version>
        </dependency>
    </dependencies>
```

> application.properties

```properties
server.port=8080
spring.application.name=springboot-shiro
spring.datasource.username=
spring.datasource.password=
spring.datasource.url=jdbc:mysql://localhost:3306/kuankuping?useUnicode=true&characterEncoding=utf8&useSSL=false&autoReconnect=true&failOverReadOnly=false&allowMultiQueries=true
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

logging.level.com.zeng.mapper=debug

#指定mapper位置
mybatis.mapper-locations=classpath:mapper/*.xml
```

### 2. 配置书写：

#### 自定义realm

```java
package com.zeng.config;

import com.zeng.model.UserDO;
import com.zeng.service.UserService;
import org.apache.shiro.authc.*;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;

import javax.annotation.Resource;

/**
 * @Author: ZYM
 * @Date: 2021年06月03日 14:16
 * @Description:
 */
//自定义realm 继承 AuthorizingRealm
public class UserRealm extends AuthorizingRealm {
    @Resource
    private UserService userService;
    /**
     * 授权
     * @author ZYM
     * @date 2021/6/3 14:17
     * @param principalCollection
     * @return org.apache.shiro.authz.AuthorizationInfo
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        System.out.println("执行了授权：principalCollection");
        UserDO currentUser = (UserDO) principalCollection.getPrimaryPrincipal();
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        info.addStringPermission(currentUser.getPerms());
        return info;
    }

    /**
     * 认证
     * @author ZYM
     * @date 2021/6/3 14:17
     * @param authenticationToken
     * @return org.apache.shiro.authc.AuthenticationInfo
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        System.out.println("执行了认证：authenticationToken");
        UsernamePasswordToken userToken = (UsernamePasswordToken) authenticationToken;
        //连接真实数据
        UserDO userDO = userService.queryByUserName(userToken.getUsername());
        //用户名由我们自动判断
        if(null == userDO){
            //这里return null 则会抛出异常UnknownAccountException 控制层就能捕获到此异常
            return null;
        }
        //对于密码的校验 shiro已经帮我们做了 --其实是shiro不想让我们去操作密码
        return new SimpleAuthenticationInfo( userDO,userDO.getPassword() ,"");
    }
}

```

#### 书写shiroconfig

```java
package com.zeng.config;

import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.LinkedHashMap;
import java.util.Map;

/**
 * @Author: ZYM
 * @Date: 2021年06月03日 14:14
 * @Description:配置反着写 先创建realm对象 再defaultWebsecurityManager 最后shiroFilterFactoryBean
 */
@Configuration
public class ShiroConfig {
    //shiroFilterFactoryBean  =======第三步
    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("securityManager") DefaultWebSecurityManager defaultWebSecurityManager){
        ShiroFilterFactoryBean factoryBean = new ShiroFilterFactoryBean();
        //设置安全管理器
        factoryBean.setSecurityManager(defaultWebSecurityManager);
        //添加shiro内置过滤器
        /**
         *  anon: 无需认证就可访问
         *  authc: 必须认证才能访问
         *  user: 必须拥有 记住我 功能才能用
         *  perms: 拥有对某个资源的权限才能访问
         *  role: 拥有某个角色权限才能访问
         */

        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<>();

        filterChainDefinitionMap.put("/user/add","perms[user:add]");
        filterChainDefinitionMap.put("/user/update","perms[user:update]");
        filterChainDefinitionMap.put("/user/*","authc");

        factoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        //设置登录请求
        factoryBean.setLoginUrl("/toLogin");
        //设置未授权情趣
        factoryBean.setUnauthorizedUrl("/toUnauthor");
        return factoryBean;
    }

    //defaultWebsecurityManager  ======第二步
    @Bean(name = "securityManager")
    public DefaultWebSecurityManager getdefaultWebSecurityManager(@Qualifier("userRealm") UserRealm userRealm){
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        //关联realm
        securityManager.setRealm(userRealm);
        return securityManager;
    }

    //创建realm对象 需要自定义类  =======第一步
    @Bean
    public UserRealm userRealm(){
        return new UserRealm();
    }
}

```



### 源码

[点我下载源码](https://zengyimingming.lanzoui.com/iM1gupqv03a)

笔记是参考狂神说的视频自己敲的，如果有兴趣可以自行看视频 [点我看视频](https://www.bilibili.com/video/BV1NE411i7S8?p=1?_blank)

如果本文档比较粗糙 可以参考此链接 [点我跳转](https://www.kuangstudy.com/bbs/1390249531194593281?_blank)

这里是后期参考别人的博客，springboot结合shiro ，搭配经典5张表 可以细粒度的控制角色和对应的权限[点我跳转](https://blog.csdn.net/qq_35387940/article/details/105728972?_blank)















