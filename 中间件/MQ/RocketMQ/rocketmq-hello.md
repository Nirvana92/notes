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