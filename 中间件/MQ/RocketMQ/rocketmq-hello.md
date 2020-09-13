`RocketMq`生产者的代码: 

引入的`RocketMQ` 依赖: 

```xml
<dependency>
  <groupId>org.apache.rocketmq</groupId>
  <artifactId>rocketmq-client</artifactId>
  <version>4.3.0</version>
</dependency>
```

生产者生产力类型: 

参考文档:  `https://github.com/apache/rocketmq/blob/master/docs/cn/RocketMQ_Example.md` 

1. 同步消息
2. 异步消息
3. 单向发送消息

同步消息: 

```java
public static void main(String[] args) throws MQClientException, UnsupportedEncodingException, RemotingException, InterruptedException, MQBrokerException {
        String groupName = "test-group";
        String namesrvAddr = "localhost:9876";
        DefaultMQProducer defaultMQProducer = new DefaultMQProducer(groupName);
        defaultMQProducer.setNamesrvAddr(namesrvAddr);
        defaultMQProducer.start();

        String topic = "test-topic", tag = "test-top", messageBody = "这是一个测试的message";
        Message message = new Message(topic, tag, messageBody.getBytes(RemotingHelper.DEFAULT_CHARSET));
        SendResult sendResult = defaultMQProducer.send(message);
        System.out.printf("%s%n", sendResult);

        defaultMQProducer.shutdown();
    }
```

然后执行报错: router 没有找到

修改引入的依赖版本为 `4.4.0`



分布式事务: 

`2pc[rocketmq实现方式]`: 实现xa协议;角色a第一阶段尝试提交到角色b;角色a第二阶段确认提交到角色b; 角色b 需要维护一个状态信息。

`tcc`: try, confirm, cancel;

长轮询的实现方式: cilent端向server端建立连接。如果没有消息需要消费。请求挂起。当有消息需要消费。server端想client端返回数据包。控制权在client端。可以通过超时机制设置超时时间。

轮询: 每次client 端向server 端发送请求, 有消息返回处理。没消息返回，等下次再次请求。

长连接: client和server 建立长连接。然后有消息服务端推送消息过来。缺点: 消费者消费能力不够的时候，server端可能还会一直往client端推送消息。

**确认消息必须被消费一次:** RocketMQ通过消息消费确认机制(ACK)来确保消息至少被消费一次, 但由于ACK消息有可能丢失等其他原因, RocketMQ无法做到消息只被消费一次。有重复消费的可能。

**回溯消息:** 回溯消息是指消息消费端已经消费成功的消息, 由于业务要求需要重新消费消息。RocketMQ支持按时间回溯消息, 时间维度可精确到毫秒, 可以向前或向后回溯。

