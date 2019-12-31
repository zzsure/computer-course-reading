# 第1章 初识Kafka
Kafka起初是由LinkedIn公司采用Scala语言开发的一个多分区、多副本且基于ZooKeeper协调的分布式消息系统，现已捐献给Apache基金会。目前Kafka已经定位为一个分布式流式处理平台，它以高吞吐、可持久化、可水平扩展、支持流数据处理等多种特性而被广泛使用。

## 1.1 基本概念
- 一个典型的Kafka体系架构包括若干Producer、若干Broker、若干Consumer，以及一个ZooKeeper集群。其中ZooKeeper是Kafka用来负责集群元数据的管理、控制器的选举等操作的。Producer将消息发送到Broker，Broker负责将接收到的消息存储到磁盘中，而Consumer负责从Broker订阅并消费消息。
- 一个或多个Broker组成了一个Kafka集群，broker代表服务代理节点。
- Kafka中的消息以主题为单位进行归类，生产者将消息发送到特定的主题，而消费者负责订阅主题并进行消费
- 主题是一个逻辑上的概念，它还可以细分为多个分区，一个分区只属于单个主题，很多时候也会把分区成为主题分区。同一主题下的不同分区包含的消息是不同的，offset是消息在分区中的唯一标识，Kafka通过它来保证消息在分区内的顺序型，不过offset并不跨越分区，也就是说，kafka保证的是分区有序而不是主题有序。
- 如果一个主题只对应一个文件，那么这个文件所在的机器I/O将会成为这个主题的性能瓶颈。而分区解决了这个问题。通过增加分区的数量可以实现水平扩展
- kafka为分区引入多副本机制，通过增加副本数量可以提升容灾能力。同一分区不同副本中保存的是相同的消息（同一时刻，并非一样），leader副本负责处理读写请求，follower副本负责与leader副本的消息同步。副本处于不同的broker中，当leader出现故障，从follower里面选举新的leader对外提供服务。
- Kafka消费者也具备一定的容灾能力，Consumer使用拉（pull）模式从服务端拉取消息，并且保存消费的具体位置。当消费者宕机恢复后则可以根据之前保存的消息位置重新拉取需要的信息消费。
- 分区中所有副本统称为AR，所有与leader副本保持一定程度同步的副本组成ISR，与leader副本同步滞后过多的副本组成OSR。HW是High Watermark缩写，速成高水位，它标识了一个特定的消息偏移量，消费者只能拉取到这个offset之前的消息。LEO是Log End Offset的缩写，它标识当前日志文件中下一条待写入消息的offset。

## 1.2 安装与配置
- JDK安装与配置
- ZooKeeper安装与配置
- Kafka的安装与配置

## 1.3 生产与消费
- kafka-topics.sh脚本可以创建主题，kafka-console-producer.sh发布消息，kafka-console-consumer.sh消费消息

## 1.4 服务端参数配置
- zookeeper.connect：指明broker要连接的ZooKeeper集群的服务地址（包括端口号），参数必填
- listeners：指明broker监听客户端连接的地址列表，即为客户端要连接broker的入口地址列表，Kafka当前支持协议：PLAINTEXT、SSL、SASL_SSL等
- broker.id，用来指定Kafka集群中broker的唯一标识
- log.dir和log.dirs，配置kafka日志文件存放的根目录
- message.max.bytes，指定broker所能接收消息的最大值

# 第2章 生产者
## 2.1 客户端开发
- 一个正常的生产逻辑需要具备以下几个步骤
  1. 配置生产者客户端参数及创建相应的生产者实例
  2. 构建待发送的消息
  3. 发送消息
  4. 关闭生产者实例

### 2.1.1 必要的参数配置
- Kafka生产者客户端KafkaProducer中有3个参数是必填的
  1. bootstrap.servers：指定生产者客户端连接Kafka所需要的broker清单，可以多个地址以,隔开
  2. key.serializer和value.serializer：broker端接收的消息必须以字节数组（byte[]）的形式存在
- KafkaProducer是线程安全的，可以在多个线程中共享单个KafkaProducer实例，也可以将KafkaProducer实例进行池化来供其他线程调用

### 2.1.2 消息的发送
- ProducerRecord的topic属性和value属性必填
- 发送消息有三种模式：发后即忘（fire-and-forget）、同步（sync）、及异步（async）
- 对于可重试的异常，可以设置retries参数
- 异步发送方式在send()方法指定一个Callback的回调函数
- close()方法会阻塞等待之前所有的发送请求完成再关闭KafkaProducer
