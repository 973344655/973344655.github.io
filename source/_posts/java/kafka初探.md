---
title: kafka初了解
date: 2019-10-9 13:23:25
tags: [java,spring]
---


win下Kafka测试环境搭建：https://blog.csdn.net/u012050154/article/details/76270655

# 一.Kafka是什么

## 1.是什么

我的理解是，kafka是一个分布式的消息队列，提供发布订阅功能.

## 2.专有名词

Broker/Topic/Partition/Producer/Consumer/Consumer Group<br>

- Broker

  Kafka集群包含一个或多个服务器，这种服务器被称为broker.

- topic

  每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic（物理上不同Topic的消息分开存储[表现于日志落地]，逻辑上一个Topic的消息虽然保存于一个或多个broker上但用户只需指定消息的Topic即可生产或消费数据而不必关心数据存于何处）

- partition

  Parition是物理上的概念，每个Topic包含一个或多个Partition.

- Producer

  消息生产者

- Consumer

  消息消费者

- Consumer Group

  消费者所属组，组内的所有消费者协调在一起来消费订阅主题(subscribed topics)的所有分区(partition)。当然，每个分区只能由同一个消费组内的一个consumer来消费.<br>
  重要的再说一遍: Consumer Group下订阅的topic下的某一个分区只能分配给某个group下的一个consumer(当然该分区还可以被分配给其他group).

# 二.Kafka作用与意义

为什么要用它,与其它产品相比怎么样？

## 1.为什么用它

### 高吞吐率，高性能

<br>怎么实现？<br>


  - 写入

  基于操作系统的页缓存来写入文件，相当于kafka将文件写入内存，然后由操作系统来决定什么时候将内存中的信息写入磁盘。

  以磁盘顺序来写入数据，不会在文件随机位置修改数据。
  - 读取

  零拷贝技术，省略了两次拷贝(OS Cache将数据拷贝到应用程序缓存和 应用程序缓存拷贝数据到socket缓存)，直接将数据发送到网卡.

  ![kafka_data_transmission](http://67.216.218.49:8000/file/blogs/java/bigdata/kafka_data_transmission_01.png)

  原图出处: https://juejin.im/post/5cc00ac95188250a59405322

### 分布式

数据分布式存储(特点就是可以存极大数据)，消费者可以根据分区来进行读取。

###  消息队列的通用优点

灵活，销峰，缓冲等。

## 2.和redis区别

###  kafka发布订阅模式，和redis有啥不同？


# 三.怎么用


## 1.springboot中kafka集成
```
<dependency>
      <groupId>org.springframework.kafka</groupId>
      <artifactId>spring-kafka</artifactId>
      <version>2.2.9.RELEASE</version>
</dependency>
```

配置文件application.properties
```

server.port=8888

#kafka，更多配置查看kafka的属性类：org.springframework.boot.autoconfigure.kafka.KafkaProperties
#指定kafka 代理地址，可以多个
#spring.kafka.bootstrap-servers=192.168.1.159:9092,192.168.1.159:9093,192.168.1.159:9094
spring.kafka.bootstrap-servers=localhost:9092
#指定默认topic id
spring.kafka.template.default-topic=testDemo
#指定listener 容器中的线程数，用于提高并发量
spring.kafka.listener.concurrency=3
#每次批量发送消息的数量
spring.kafka.producer.batch-size=1000
#将Java对象转换成字节数组发送给broker
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer

#指定默认消费者group id
spring.kafka.consumer.group-id=myGroup1
#若设置为earliest，那么会从头开始读partition，latest读取最新的
spring.kafka.consumer.auto-offset-reset=latest
#将字节数组读取
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer
```

## 2.生产一条消息
KafkaTemplate 是Spring提供的用来发消息的实现类，可以在application.properties中进行配置属性，也可以用kafka自己的KafkaProducer来send()消息.

```
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Component;

import java.util.Properties;

/**
 * kafka生产者send()消息demo
 *
 * 两种，一种是Spring提供的可以用spring配置的KafkaTemplate
 * 一种是Kafka自带的，需要自己注入配置信息的KafkaProducer
 */

@Component
public class ProducerDemo {


    @Autowired
    private KafkaTemplate kafkaTemplate;


    public void send1(){
        try {
            kafkaTemplate.send("testDemo","data1");
            System.out.println("[*]: send success 1 !");
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    public void send2(){
        Properties kafkaProps = new Properties();
        kafkaProps.put("bootstrap.servers","localhost:9092");
        kafkaProps.put("key.serializer","org.apache.kafka.common.serialization.StringSerializer");
        kafkaProps.put("value.serializer","org.apache.kafka.common.serialization.StringSerializer");

        KafkaProducer producer = new KafkaProducer<String, String>(kafkaProps);

        ProducerRecord record = new ProducerRecord("testDemo","data2");

        try{
            producer.send(record);
            System.out.println("[*]: send success 2 !");
        }catch (Exception e){
            e.printStackTrace();
        }

    }

}

```

## 3.订阅消费消息
```
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.List;
import java.util.Properties;


/**
 * kafka消费消息demo
 *
 * 1.@KafkaListener 注解为Spring提供用来监听数据的，
 * 由KafkaListenerAnnotationBeanPostProcessor类在postProcessAfterInitialization()时读取所有@Kafkalistener注解
 *
 * 2.KafkaConsumer为kafka自带的类，需要自己初始化配置文件
 */
@Component
public class ConsumerDemo {

    @KafkaListener(id = "testConsumer1", topics = "testDemo", groupId = "myGroup1")
    public void receive1(String data){
        System.out.println("[*]: receive success 1 !");
        System.out.println(data);

    }

    public void receive2(){
        Properties kafkaProps = new Properties();
        kafkaProps.put("bootstrap.servers","localhost:9092");
        kafkaProps.put("group.id","myGroup1");
        kafkaProps.put("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
        kafkaProps.put("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer");

        KafkaConsumer kafkaConsumer = new KafkaConsumer(kafkaProps);

        //List topics = List.of("testDemo");
        List<String> topics = new ArrayList<>();
        topics.add("testDemo");

        try {
            kafkaConsumer.subscribe(topics);
            System.out.println("[*]: receive success 2 !");
            while(true){
                ConsumerRecords consumerRecords = kafkaConsumer.poll(1000);
                if(!consumerRecords.isEmpty()){
                   for(Object consumerRecord: consumerRecords.records("testDemo")){
                       ConsumerRecord temp = (ConsumerRecord)consumerRecord;
                       System.out.println("[*]: new data: " + temp.value());
                   }
                }
            }
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```

# 四.其它
