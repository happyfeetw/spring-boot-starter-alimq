## 基于aliyun MQ服务封装的SpringBoot starter [源码地址>>>](https://github.com/jibaole/spring-boot-starter-alimq)


### 一、接入概要说明

* 快速入门：<https://help.aliyun.com/document_detail/34411.html>
* 官网Demo地址：<https://github.com/AliwareMQ/mq-demo>

##### 1、通用参数说明

| 参数名	     | 参数说明  |
| --------   | -----  |
| onsAddr        | 设置 MQ TCP 协议接入点，参考上面表格（推荐）      |
| AccessKey        | 您在阿里云账号管理控制台中创建的 AccessKey，用于身份认证      |
| SecretKey       | 您在阿里云账号管理控制台中创建的 SecretKey，用于身份认证     |
| producerId       | 您在阿里云账号管理控制台中创建的producerId,用户发送消息    |
| consumerId       | 您在阿里云账号管理控制台中创建的consumerId,用户订阅消息    |




##### 2、发送消息参数说明

|参数名|	参数说明|
| --------   | -----  |
|ProducerId|	您在控制台创建的 Producer ID|
|SendMsgTimeoutMillis	|设置消息发送的超时时间，单位（毫秒），默认：3000|
|CheckImmunityTimeInSeconds（事务消息）|	设置事务消息第一次回查的最快时间，单位（秒）|
|shardingKey（顺序消息）|	顺序消息中用来计算不同分区的值|

![image](http://om9j2ardo.bkt.clouddn.com/QQ%E5%9B%BE%E7%89%8720180607122953.png)
##### 3、订阅消息参数说明

|参数名|	参数说明|
| --------   | -----  |
|ConsumerId	|您在 MQ 控制台上创建的 Consumer ID|
|MessageModel|设置 Consumer 实例的消费模式，默认为集群消费（值：CLUSTERING）;广播消费（BROADCASTING）|
|ConsumeThreadNums	|设置 Consumer 实例的消费线程数，默认：64|
|MaxReconsumeTimes|	设置消息消费失败的最大重试次数，默认：16|
|ConsumeTimeout	|设置每条消息消费的最大超时时间，超过设置时间则被视为消费失败，等下次重新投递再次消费。每个业务需要设置一个合理的值，单位（分钟）。默认：15|
|suspendTimeMillis|（顺序消息）	只适用于顺序消息，设置消息消费失败的重试间隔时间|

![image](http://om9j2ardo.bkt.clouddn.com/QQ%E5%9B%BE%E7%89%8720180607122207.png)

##### 4、TCP协议接入域名

|环境说明	|接入点|
| --------   | -----  |
|公共云内网接入|（阿里云经典网络/VPC）：
华东1、华东2、华北1、华北2、华南1、香港|	http://onsaddr-internal.aliyun.com:8080/rocketmq/nsaddr4client-internal|
|公共云公网接入（非阿里云选择这个）|	http://onsaddr-internet.aliyun.com/rocketmq/nsaddr4client-internet|
|公共云 Region：亚太东南1(新加坡)	|http://ap-southeastaddr-internal.aliyun.com:8080/rocketmq/nsaddr4client-internal|
|公共云 Region：亚太东南3(吉隆坡)	|http://ons-ap-southeast-3-internal.aliyun.com:8080/rocketmq/nsaddr4client-internal|
|公共云 Region：亚太东北 1 (东京)|	http://ons-ap-northeast-1-internal.aliyun.com:8080/rocketmq/nsaddr4client-internal|
|金融云 Region：华东1	|http://jbponsaddr-internal.aliyun.com:8080/rocketmq/nsaddr4client-internal|
|金融云 Region：华东2、华南1|	http://mq4finance-sz.addr.aliyun.com:8080/rocketmq/nsaddr4client-internal|




### 二、starter的基本用法：

> 1、pom.xml引入依赖

```xml
       <dependency>
            <groupId>cn.knowbox.book</groupId>
            <artifactId>spring-boot-starter-alimq</artifactId>
            <version>1.0.0.RELEASE</version>
        </dependency>
```
         
		
> 2、application配置文件中添加相应配置


```properties

    aliyun.mq.onsAddr=http://onsaddr-internet.aliyun.com/rocketmq/nsaddr4client-internet
    aliyun.mq.accessKey=xxxxxxx
    aliyun.mq.secretKey=xxxxxx
    #为false表示不引入producer，为true则producerId必须提供
    aliyun.mq.producer.enabled=true
    aliyun.mq.producer.producerId=PID_PRODUCER_SMS 
    #为false表示不引入consumer，为true则consumerId必须提供
    aliyun.mq.consumer.enabled=true
    aliyun.mq.consumer.consumerId=CID_CONSUMER_SMS
    
```





> 3、 使用producer只需要在相应类中依需要注入对应实例

```java

/**
 * @author jibaole
 * @version 1.0
 * @desc 生产者-通用方法
 * @date 2018/7/7 下午5:19
 */
@Slf4j
@SuppressWarnings({"WeakerAccess", "unused"})
public class RocketMQTemplate {

    @Resource
    private ProducerBean producer;

    /****
     * @Description: 同步发送
     * @Param: [event]
     * @Author: jibaole
     */
    public SendResult send(MessageEvent event) {}

    /****
     * @Description: 同步发送(带延迟时间)
     * @Param: [event, delay]
     * @Author: jibaole
     */
    public SendResult send(MessageEvent event, long delay) {}

    /**
     * @Description: 单向发送
     * @Param: [event]
     * @Author: jibaole
     */
    public void sendOneway(MessageEvent event) {}

    /**
     * @Description: 异步发送
     * @Param: [event]
     * @Author: jibaole
     */
    public void sendAsync(MessageEvent event) {}


    /**
     * @Description: 异步发送(带延迟时间)
     * @Param: [event, delay]
     * @Author: jibaole
     */
    public void sendAsync(MessageEvent event, long delay) {}


}

```

```java


@Service
public class ALiService {
@Autowired
   private RocketMQTemplate rocketMQTemplate;
   
    /**
     * 生产者-调用
     */
    public void sentMsg() {
        /**封装消息*/
        MessageEvent event = new MessageEvent();
        event.setTopic("base_sms");
        event.setTag("Tag_user");
       
        User user = new User();
        user.setName("Paul");
        user.setAdds("北京市 昌平区 龙锦苑东二区");
         /**封装任意类型领域对象*/
        event.setDomain(user);
        
        rocketMQTemplate.send(event);
    }
}

```
> 4、 consumer监听处理类实现，继承AbstractMessageListener类，实现handle方法即可，如：

```java


/**
 * @author jibaole
 * @version 1.0
 * @desc 用户事件监听处理
 * @date 2018/7/5 上午11:18
 */

@Service
@RocketMQMessageListener(topic = "base_sms",tag = "Tag_user")
public class UserMessageListener extends AbstractMessageListener<User> {
    /**
     * 生产者
     */
    @Override
    public void handle(User user) {
        System.out.println(user instanceof User);

        System.out.println(user.toString());
    }
}

```
* topic为必填，可以配置多个不同topic监听业务处理
* tag为`*`时，表示：接收所有topic消息；业务处理逻辑相同多个tag时，用tag1||tag2||tag3||……||tagn
* 定义泛型：接收对象类型要与发送对象类型一致

> 5、优点
* 1、 使用简单：开箱即用,只需要简单配置
* 2、分业务处理：生产-消费端可以自定义配置不同topic、不同tag。		

