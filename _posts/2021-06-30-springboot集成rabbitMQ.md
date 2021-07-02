---

layout: post

title: "Spring Boot 中简单使用RabbitMQ"

imges: 

date: 2021-06-30 10:00:00 +0800

description: 此文章仅作为个人学习使用，笔者也是看了尚硅谷的视频自己再总结的

---



>  说明：本文仅作为个人学习使用，温故而知新

### springBoot集成RabbitMQ粗略教程



### 1. centos7安装rabbitMQ

先下载安装所需软件

链接: https://pan.baidu.com/s/1UGZLILSfAxXqbLeQQCb5KQ 提取码: hk7p 

具体安装教程请自行百度！笔者是跟着尚硅谷学习滴，[点我跳转](https://www.bilibili.com/video/BV1cb4y1o7zz?p=10)



### 2.本地创建springboot项目



<img src="https://gitee.com/zengyimingming/picrepo/raw/master/images/20210630111802.png" style="zoom: 33%;" />

笔者用的springboot版本2.3.1的··读者自行修改自己的即可，壁纸jdk8

1.安装依赖--swagger可以不要

```xml-dtd
 <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <!--RabbitMQ 依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
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
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.47</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <!--swagger-->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.9.2</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.9.2</version>
        </dependency>
        <!--RabbitMQ 测试依赖-->
        <dependency>
            <groupId>org.springframework.amqp</groupId>
            <artifactId>spring-rabbit-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

安装完成后务必检查依赖是否下载完毕！

> 修改配置文件

```properties
spring.rabbitmq.host=192.168.201.23
spring.rabbitmq.port=5672
spring.rabbitmq.username=admin
spring.rabbitmq.password=123
```

> swagger配置，需导入相关依赖

```java
/**
 * @Author: ZYM
 * @Date: 2021年06月30日 14:00
 * @Description:swagger配置
 * @Configuration  配置类
 * @EnableSwagger2 开启swagger
 */
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket webApiConfig() {
        return new Docket(DocumentationType.SWAGGER_2)
                .groupName("webApi")
                .apiInfo(webApiInfo())
                .select()
                .build();
    }

    private ApiInfo webApiInfo() {
        return new ApiInfoBuilder()
                .title("rabbitmq 接口文档")
                .description("本文档描述了 rabbitmq 微服务接口定义")
                .version("1.0")
                .contact(new Contact("zeng", "zengyimingming.github.io", "1012785014@qq.com")).build();
    }
}

```

访问swagger的地址笔者是：http://localhost:8080/swagger-ui.html

![](https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702092744.png)

访问地址能正确显示即可，swagger非必须配置【笔者只是方便自己测试】

基本的搭建完毕 现在简单测试direct类型交换机



### Direct(直接)交换机测试



<img src="https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702093418.png" style="zoom: 80%;" />

笔者理解如下，生产者P只负责生产消息，然后将消息发送给交换机，并指定发送的routingkey，生产者不知道消息是到哪一个队列，这里有点类似于网络中的路由器RIP协议，路由器只是保存了直连路由器上的信息，通过指定端口发送报文，报文就从指定端口发送出去。 上图中定义了交换机**direct_exchange_x**，并且此交换机分别通过**routingkey01**和**routingkey02**绑定到队列**queue1**和**queue2**，队列1的消费者是C1，队列2的消费者是C2，

对于**绑定**这个概念可以这么理解：**队列只对它绑定的交换机的消息感兴趣**  

直接类型交换机的工作方式是，消息只去到它绑定的routingKey 队列中去  

现在笔者演示通过生产者P发送一条消息到队列1上，C1负责消费消息

按图所示，需要创建一个交换机**direct_exchange_x**，两个队列**direct_exchange_x_banding_queue1**和**direct_exchange_x_banding_queue2**，队列1通过routingKey01绑定到交换机，队列2通过routingKey02绑定到交换机

```java
package com.zeng.config.queueandexchange;

import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.*;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @Author: ZYM
 * @Date: 2021年07月01日 16:18
 * @Description:直接类型交换机demo
 */
@Configuration
@Slf4j
public class DirectExchangeConfig {
    /**
     *直接类型交换机名称
     */
    public static final String DIRECT_EXCHANGE_NAME = "direct_exchange_x";
    /**
     队列1名称
     */
    public static final String DIRECT_EXCHANGE_BINDING_QUEUE01 = "direct_exchange_x_banding_queue1";
    /**
     队列2名称
     */
    public static final String DIRECT_EXCHANGE_BINDING_QUEUE02 = "direct_exchange_x_banding_queue2";
    /**
     * 指定的routingKey
     */
    public static final String ROUTINGKEY01 = "routingKey01";
    public static final String ROUTINGKEY02 = "routingKey02";


    /**
     * 声明队列1
     * @author ZYM
     * @date 2021/7/1 16:41
     * @return org.springframework.amqp.core.Queue
     */
    @Bean(value = "createQueue")
    public Queue createQueue(){
        return QueueBuilder.durable(DIRECT_EXCHANGE_BINDING_QUEUE01).build();
    }

    /**
     * 声明直接类型交换机 非持久化
     * @author ZYM
     * @date 2021/7/1 16:46
     * @return org.springframework.amqp.core.DirectExchange
     */
    @Bean(value = "createDirectExchange")
    public DirectExchange createDirectExchange(){
        return ExchangeBuilder.directExchange(DIRECT_EXCHANGE_NAME).durable(false).build();
    }

    /**
     * 队列1绑定direct交换机
     * @author ZYM
     * @date 2021/7/1 16:51
     * @param createQueue
     * @param DirectExchange
     * @return org.springframework.amqp.core.Binding
     */
    @Bean
    public Binding queueBindingExchangeX(@Qualifier("createQueue")Queue createQueue,@Qualifier("createDirectExchange")DirectExchange DirectExchange){
        return BindingBuilder.bind(createQueue).to(createDirectExchange()).with(ROUTINGKEY01);
    }

    //------------------------------------------------------------------------//
    /**
     * 声明队列2
     * @author ZYM
     * @date 2021/7/1 16:41
     * @return org.springframework.amqp.core.Queue
     */
    @Bean(value = "createQueue2")
    public Queue createQueue2(){
        return QueueBuilder.durable(DIRECT_EXCHANGE_BINDING_QUEUE02).build();
    }

    /**
     * 队列2绑定direct交换机
     * @author ZYM
     * @date 2021/7/1 16:51
     * @param createQueue2
     * @param DirectExchange
     * @return org.springframework.amqp.core.Binding
     */
    @Bean
    public Binding queueBindingExchangeX2(@Qualifier("createQueue2")Queue createQueue2,@Qualifier("createDirectExchange")DirectExchange DirectExchange){
        return BindingBuilder.bind(createQueue2).to(createDirectExchange()).with(ROUTINGKEY02);
    }

}

```

代码中就已经完成两个队列和交换机的绑定关系

现在启动springboot程序，观察rabbitMQ控制面板是否新增了交换机和两个队列

<img src="https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702095653.png" style="zoom: 67%;" />

<img src="https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702095759.png" style="zoom:50%;" />

<img src="https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702095920.png" style="zoom:67%;" />

第一张图是交换机 第二张图是两个队列 第三章图中表示交换机和队列的绑定关系

交换机和队列已经完毕！ 剩下的就是测试发送消息给交换机 消费者消费消息

所以还需要再创建生产者和两个消费者

**创建生产者**

```java
@RestController
@RequestMapping("/helloWorld")
@Slf4j
public class HelloWorldController {
    @Resource
    private RabbitTemplate rabbitTemplate;
    /**
     * 使用默认类型交换机
     * @author ZYM
     * @date 2021/7/1 16:22
     * @param msg
     */
    @GetMapping("/send/{msg}")
    public void sendMessage(@PathVariable("msg")String msg){
        log.info("生产者发送的消息:{},当前时间：{}",msg,new Date().toString());
        rabbitTemplate.convertAndSend(DirectExchangeConfig.DIRECT_EXCHANGE_NAME,DirectExchangeConfig.ROUTINGKEY01,msg);
    }
}
```

**注意**：上述生产者的参数 

参数1：交换机名称  **direct_exchange_x**

参数2：routingkey  **routingKey01**

参数3：消息本身

生产者发送消息给**direct_exchange_x**，并且指定routingkey是**routingKey01**，所以是准备发送消息到队列1中去



现在演示发送一条消息

<img src="https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702101016.png" style="zoom:67%;" />

idea控制台打印

<img src="https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702101058.png" style="zoom:67%;" />

再去rabbitMQ控制面板查看队列1中是否有消息

![](https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702101220.png)

可以看到消息已经发送到目标队列1中去了



最后还剩下消费者**C1和C2**需要

```java
@Slf4j
@Component
//消费者C1 监听的队列是 direct_exchange_x_banding_queue1
public class Consumer01 {
    @RabbitListener(queues = DirectExchangeConfig.DIRECT_EXCHANGE_BINDING_QUEUE01)
    public void receiveMessage(Message message) throws Exception {
        log.info("Consumer01接收消息：{} 当前时间：{}",new String(message.getBody(),"UTF-8"),new Date().toString());
    }
}
```

```java
@Slf4j
@Component
//消费者C2 监听的队列是 direct_exchange_x_banding_queue2
public class Consumer02 {
    @RabbitListener(queues = DirectExchangeConfig.DIRECT_EXCHANGE_BINDING_QUEUE02)
    public void receiveMessage(Message message) throws Exception {
        log.info("Consumer02接收消息：{}",new String(message.getBody(),"UTF-8"));
    }
}
```

在springboot中 是通过**@RabbitListener**监听指定的队列的，只要监听的队列里有消息，这个方法就会从队列中去获取消息

在MQ服务器上，有队列1和队列2 ，并且java代码中C1和C2分别指定消费队列1和队列2的消息

现在再次重启springboot，查看控制台消费者1是否消费了消息，消费者2又是否会打印到控制台呢？

<img src="https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702102012.png" style="zoom:67%;" />

从图中可知，只有消费者C1能获取消息，消费者C2并没有获取到

现在再去RabbitMQ控制面板查看消费是否被消费

<img src="https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702102135.png" style="zoom:80%;" />

结果：从图中看到  消息已经被消费掉了，并且可指定发送到具体的一个队列

这就是直接交换机的一个最基本应用

适用场景：**有优先级的任务，根据任务的优先级把消息发送到对应的队列，这样可以派更多的资源去处理高优先级的队列**。



### Fanout(扇形)交换机测试

介绍：扇形交换机是最基本的交换机类型，它能做的事非常简单：**广播消息**，扇形交换机会把能接收到的消息全部发送给绑定在自己身上的队列。因为广播不需要"思考"，所以**扇形交换机**处理消息的速度也是所有的交换机类型里面**最快**的

<img src="https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702105108.png" style="zoom: 67%;" />

三个队列**queue03**、**queue04**、**queue04**，交换机**X** 

所以先定义三个队列和一个交换机，并且将队列和交换机绑定

伪代码如下

```java
@Configuration
public class FanoutExchangeConfig {
    //创建3个队列 queue03  queue04  queue05
    //这三个队列默认持久化的 参数1：队列名称
    public static final String QUEUE03 = "queue03";
    public static final String QUEUE04 = "queue04";
    public static final String QUEUE05 = "queue05";

    public static final String FANOUT_EXCHANGE_NAME = "X";

    @Bean(value = "queue03")
    public Queue queue03() {
        return QueueBuilder.durable(QUEUE03).build();
    }

    @Bean(value = "queue04")
    public Queue queue04() {
        return QueueBuilder.durable(QUEUE04).build();
    }

    @Bean(value = "queue05")
    public Queue queue05() {
        return QueueBuilder.durable(QUEUE05).build();
    }

    /**
     * 创建扇形交换机 X
     * 参数1 交换机名称 X
     * 参数2 非持久化（意思就是MQ服务重启后，这个交换机就么有了，保存在内存中而非磁盘中）
     *
     * @return org.springframework.amqp.core.FanoutExchange
     * @author ZYM
     * @date 2021/7/2 11:03
     */
    @Bean(value = "createFanoutExchange")
    public FanoutExchange createFanoutExchange() {
        return ExchangeBuilder
                .fanoutExchange(FANOUT_EXCHANGE_NAME)
                .durable(false)
                .build();
    }

    //三个队列和交换机绑定  并且不需指定routingKey 因为是广播的
    @Bean
    public Binding quque03BindingExchangeX(@Qualifier("queue03") Queue queue03, @Qualifier("createFanoutExchange") FanoutExchange createFanoutExchange) {
        return BindingBuilder.bind(queue03).to(createFanoutExchange);
    }

    @Bean
    public Binding quque04BindingExchangeX(@Qualifier("queue04") Queue queue04, @Qualifier("createFanoutExchange") FanoutExchange createFanoutExchange) {
        return BindingBuilder.bind(queue04).to(createFanoutExchange);
    }

    @Bean
    public Binding quque05BindingExchangeX(@Qualifier("queue05") Queue queue05, @Qualifier("createFanoutExchange") FanoutExchange createFanoutExchange) {
        return BindingBuilder.bind(queue05).to(createFanoutExchange);
    }
}
```

现在重启springboot 然后观察rabbitMQ控制面板，是否创建了fanout(扇形)交换机**X**和三个队列**queue03**，**queue04**，**queue05**，并且交换机是否正确绑定了三个队列

<img src="https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702111327.png" style="zoom:80%;" />

上图扇形交换机创建完毕

![](https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702111412.png)

队列创建完毕

<img src="https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702111456.png" style="zoom:67%;" />

交换机和队列绑定关系已完毕



现在就需要一个生产者向此交换机发送消息，并且需要三个消费者**C3**、**C4**、**C5**分别消费**queue03**、**queue04**、**queue05**的消息

生产者伪代码如下

```java
/**
 * 发送消息到扇形交换机
 * @author ZYM
 * @date 2021/7/2 11:18
 * @param msg
 */
@GetMapping("/fanoutSend/{msg}")
public void fanoutSendMessage(@PathVariable("msg")String msg){
    log.info("生产者发送的消息:{},当前时间：{}",msg,new Date().toString());

    /**
     参数1  交换机名称 X
     参数2  routingkey 此时设置空字符串即可
     参数3 消息本身
     */
    rabbitTemplate.convertAndSend(FanoutExchangeConfig.FANOUT_EXCHANGE_NAME,"",msg);
}
```

重启springboot程序 调用接口让生产者发送消息扇形交换机**X**，再去rabbitMQ控制面板查看队列是否收到消息

![](https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702131105.png)

![](https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702131221.png)

查看rabbitMQ控制面板，三个队列都收到一条消息

![](https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702131323.png)



创建消费者**C3**、**C4**、**C5** 分别消费**queue03**、**queue04**、**queue05**中的消息

伪代码如下

```java
/**
 * @Author: ZYM
 * @Date: 2021年07月02日 13:16
 * @Description:
 * C3----->queue03
 * C4----->queue04
 * C5----->queue05
 */
@Component
@Slf4j
public class Consumer03_04_05 {
    @RabbitListener(queues = "queue03")
    public void consumer03(Message message)throws Exception{
        log.info("消费者03收到消息：{}",new String(message.getBody(),"UTF-8"));
    }

    @RabbitListener(queues = "queue04")
    public void consumer04(Message message)throws Exception{
        log.info("消费者04收到消息：{}",new String(message.getBody(),"UTF-8"));
    }

    @RabbitListener(queues = "queue05")
    public void consumer05(Message message)throws Exception{
        log.info("消费者05收到消息：{}",new String(message.getBody(),"UTF-8"));
    }
}
```



重启springboot 查看控制台，消费者已经消费了消息

![](https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702132429.png)

此时查看rabbitMQ控制面板**queue03**、**queue04**、**queue05**中是否还存在消息

![](https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702132553.png)

三个队列中的消息已经被消费

结果：生产者(消息发送方)生产消息给扇形交换机，所有队列都能收到消息，队列和交换机并没有通过routingkey绑定，因此**扇形交换机采用的是广播的模式**，

适用场景：**需要给所有绑定该交换机的队列直接发送消息时使用**。



### topic(主题)交换机

直连交换机的**routingkey**方法简单，如果希望将一条消息发送给多个队列，那么这个交换机需要绑定非常多的**routingkey**，这样的话消息的管理就会非常的困难。
所以**RabbitMQ**提供了一种主题交换机，发送到主题交换机上的消息需要携带指定规则的**routingkey**，主题交换机会根据这个规则将数据发送到对应的队列上。
主题交换机的**routingkey**需要有一定的规则，交换机和队列绑定时候设置的**bindingkey**需要采用*.#.*…的格式，每个部分用"."分开

在这个规则列表中，其中有两个替换符是大家需要注意的
*****(星号)可以代替一个单词
**\#**(井号)可以替代零个或多个单词  

百度图如下

<img src="https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fupload-images.jianshu.io%2Fupload_images%2F8579438-6889c2b198f756b1.png&refer=http%3A%2F%2Fupload-images.jianshu.io&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1627798524&t=53997c54fca0696d2e4833d90d377e13" style="zoom: 80%;" />



**ToPic案列，笔者参考尚硅谷视频**

下图绑定关系如下

queue06-->绑定的是
中间带 orange 带 3 个单词的字符串(\*.orange.\*)
queue07-->绑定的是
最后一个单词是 rabbit 的 3 个单词(\*.\*.rabbit)
第一个单词是 lazy 的多个单词(lazy.#)  

![](https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702140022.png)



下表可以用于理解下匹配规则

| routingKey               | 能收到消息的队列                           |
| ------------------------ | ------------------------------------------ |
| quick.orange.rabbit      | queue06 queue07                            |
| lazy.orange.elephant     | queue06 queue07                            |
| quick.orange.fox         | queue06                                    |
| lazy.brown.fox           | queue07                                    |
| lazy.pink.rabbit         | queue07                                    |
| quick.brown.fox          | 不匹配任何绑定不会被任何队列接收到会被丢弃 |
| quick.orange.male.rabbit | 四个单词不匹配任何绑定会被丢弃             |
| lazy.orange.male.rabbit  | queue07                                    |

当队列绑定关系是下列这种情况时需要引起注意
**当一个队列绑定键是#,那么这个队列将接收所有数据，就有点像 fanout 了**
**如果队列绑定键当中没有#和*出现，那么该队列绑定类型就是 direct 了**  



按照图示配置主题交换机和对应的队列，将**queue06**绑定到交换机**XX**，对应**routingkey**是**\*.orange.\***；**queue07**绑定到交换机**XX**，对应**routingkey**是**\*.\*.rarbbit** 和**lazy.#**；

伪代码如下

```java
@Slf4j
@Configuration
public class TopicExchangeConfig {
    public static final String QUEUE06 = "queue06";
    public static final String QUEUE07 = "queue07";
    //交换机名称 xx
    public static final String TOPIC_EXCHANGE = "XX";

    /**
     * 创建队列 queue06 queue07
     */
    @Bean(value = "queue06")
    public Queue queue06() {
        return QueueBuilder.durable(QUEUE06).build();
    }

    @Bean(value = "queue07")
    public Queue queue07() {
        return QueueBuilder.durable(QUEUE07).build();
    }

    /**
     * 创建topic交换机 名称： XX
     */
    @Bean(value = "topicExchange")
    public TopicExchange topicExchange() {
        return ExchangeBuilder.topicExchange(TOPIC_EXCHANGE).durable(false).build();
    }

    /**
     * 队列queue06 绑定到主题交换机XX
     * routingKey:  *.orange.*
     */
    @Bean
    public Binding queue06BindingExchangeXX(@Qualifier("queue06") Queue queue06, @Qualifier("topicExchange") TopicExchange topicExchange) {
        return BindingBuilder.bind(queue06).to(topicExchange).with("*.orange.*");
    }

    /**
     * 队列queue07绑定到主题交换机XX
     * routingKey:  *.*.rabbit
     */
    @Bean
    public Binding queue07BindingExchangeXX(@Qualifier("queue07") Queue queue07, @Qualifier("topicExchange") TopicExchange topicExchange) {
        return BindingBuilder.bind(queue07).to(topicExchange).with("*.*.rabbit");
    }

    /**
     * 队列queue07绑定到主题交换机XX
     * routingKey:  lazy.#
     */
    @Bean
    public Binding queue07BindingExchangeXX2(@Qualifier("queue07") Queue queue07, @Qualifier("topicExchange") TopicExchange topicExchange) {
        return BindingBuilder.bind(queue07).to(topicExchange).with("lazy.#");
    }

}
```

这时重启springboot 查看rabbitMQ控制面板，是否创建对应的交换机和队列，绑定关系是否成功

<img src="https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702145131.png" style="zoom:67%;" />

<img src="https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702145207.png" style="zoom: 50%;" />

交换机和队列创建完毕

<img src="https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702145302.png" style="zoom: 67%;" />

对应绑定关系正确

现在可以通过消费者发送消息到交换机**XX**上，routingkey就参考上表中的，并同时创建消费者C6、C7消费消息

**生产者伪代码如下**

```java
/**
 * 发送消息到主题交换机 生产者
 * @author ZYM
 * @date 2021/7/2 15:04
 * @param msg
 */
@GetMapping("/topicSend/{msg}")
public void topicSendMessage(@PathVariable("msg")String msg){
    log.info("生产者发送的消息:{},当前时间：{}",msg,new Date().toString());
    Map<String,String> map = new LinkedHashMap<>();
    map.put("quick.orange.rabbit","这条消息的routingkey是"+"quick.orange.rabbit"+"此消息queue06 queue07都能收到："+msg);
    map.put("lazy.orange.elephant","这条消息的routingkey是"+"lazy.orange.elephant"+"此消息queue06 queue07都能收到："+msg);
    map.put("quick.orange.fox","这条消息的routingkey是"+"quick.orange.fox"+"此消息只有queue06能收到："+msg);
    map.put("lazy.brown.fox","这条消息的routingkey是"+"lazy.brown.fox"+"此消息只有queue07能收到："+msg);
    map.put("lazy.pink.rabbit","这条消息的routingkey是"+"lazy.pink.rabbit"+"此消息只有queue07能收到："+msg);
    map.put("quick.brown.fox","这条消息的routingkey是"+"quick.brown.fox"+"不匹配任何绑定不会被任何队列接收到会被丢弃："+msg);
    map.put("quick.orange.male.rabbit","这条消息的routingkey是"+"quick.orange.male.rabbit"+"四个单词不匹配任何绑定会被丢弃："+msg);
    map.put("lazy.orange.male.rabbit ","这条消息的routingkey是"+"lazy.orange.male.rabbit"+"此消息只有queue07能收到："+msg);

    Set<Map.Entry<String, String>> entries = map.entrySet();
    for (Map.Entry<String, String> entry : entries) {
        rabbitTemplate.convertAndSend(TopicExchangeConfig.TOPIC_EXCHANGE,entry.getKey(),entry.getValue());
        log.info("通过routingkey："+entry.getKey()+"发送的消息已经成功发送！");
    }
    log.info("所有消息发送完毕");
}
```

**消费者伪代码如下**

```java
/**
 * @Author: ZYM
 * @Date: 2021年07月02日 15:14
 * @Description:消费者 消费queue06 和 queue07的消息
 */
@Component
@Slf4j
public class Consumer06_07 {
    @RabbitListener(queues = "queue06" ,containerFactory="rabbitListenerContainerFactory")
    public void consumer06(Message message) throws Exception {
        log.info("C6获取到的消息====>"+new String(message.getBody(),"UTF-8"));
    }
    @RabbitListener(queues = "queue07" ,containerFactory="rabbitListenerContainerFactory")
    public void consumer07(Message message) throws Exception {
        log.info("C7获取到的消息====>"+new String(message.getBody(),"UTF-8"));
    }
}
```

重启springboot 调用接口发送消息给主题交换机，查看消息是否如表格中的规则发送到对应的队列

![](https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702152234.png)

查看控制台

![](https://gitee.com/zengyimingming/picrepo/raw/master/images/20210702152624.png)

可以看到，发送消息是严格按照规则走的

