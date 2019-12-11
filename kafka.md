##1，kafka角色
- producer：生产者。
- consumer：消费者。
- topic: 消息以topic为类别记录,Kafka将消息种子(Feed)分类, 每一类的消息称之为一个主题(Topic)。
- broker：以集群的方式运行,可以由一个或多个服务组成;消费者可以订阅一个或多个主题(topic), 并从Broker拉数据,从而消费这些已发布的消息。

 主题topic是发布记录的类别或源名称。kafka中的主题总是多个订阅者；也就是说，一个主题可以有零个、一个或多个consumer订阅写入到它的数据。
对于每个主题，kafka集群都维护一个分区日志，如下所示：

![avatar](http://kafka.apache.org/23/images/log_anatomy.png)

 每个分区都是一个有序的、不可变的记录序列，这些记录连续地追加到结构化提交日志中。每个分区中的记录都被分配了一个称为偏移量的顺序ID号，该偏移量唯一标识分区中的每个记录。
kafka集群持久地保存所有已发布的记录，不管它们是否使用了可配置的保留期。例如，如果保留策略设置为两天，则在记录发布后的两天内，可以使用该消息，之后将丢弃该消息以释放空间。

<img src="http://kafka.apache.org/23/images/log_consumer.png" width="300" hegiht="400" align=center />

 消费者将自己标记为一个消费者组，并将每个发布到主题的记录传递给每个订阅消费群中的一个消费者实例。消费者实例可以在单独的进程中或在单独的机器上。 

![avatar](http://kafka.apache.org/23/images/consumer-groups.png)
##2，安装
```
#官方下载地址：http://kafka.apache.org/downloads
tar -xzf kafka_2.12-1.1.1.tgz
cd kafka_2.12-1.1.0
```
##3，启动kafka server
```
#需先启动zookeeper# -daemon 可启动后台守护模式
bin/zookeeper-server-start.sh config/zookeeper.properties
bin/kafka-server-start.sh config/server.properties
```
##4，创建一个话题，test话题2个分区
```
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 2 --topic test
Created topic "test".
#显示所有话题
bin/kafka-topics.sh --list --zookeeper localhost:2181 test
#显示话题信息
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test
Topic:test    PartitionCount:2    ReplicationFactor:1    Configs:
    Topic: test    Partition: 0    Leader: 0    Replicas: 0    Isr: 0
    Topic: test    Partition: 1    Leader: 0    Replicas: 0    Isr: 0
```
##5，启动一个生产者（输入消息）
```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
>i am a new msg !
>i am a good msg ?
```
##6，启动一个消费者（等待消息） 
```
#注意这里的--from-beginning，每次都会从头开始读取，你可以尝试去掉和不去掉看下效果
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
i am a new msg !
i am a good msg ? 
```
##7，安装kafka的php扩展
```
#安装rdkfka库文件
git clone https://github.com/edenhill/librdkafka.git
cd librdkafka/
./configure 
make
sudo make install
#安装php拓展
git clone https://github.com/arnaud-lb/php-rdkafka.git
cd php-rdkafka
phpize
./configure
make
sudo make install
#修改php配置文件
vim php.ini
extension=rdkafka.so
```
##8，代码样例
- 生产者
```
$conf = new RdKafka\Conf();
#正确回调
$conf->setDrMsgCb(function ($kafka, $message) {
    file_put_contents("./dr_cb.log", var_export($message, true).PHP_EOL, FILE_APPEND);
});
#错误回调
$conf->setErrorCb(function ($kafka, $err, $reason) {
    file_put_contents("./err_cb.log", sprintf("Kafka error: %s (reason: %s)", rd_kafka_err2str($err), $reason).PHP_EOL, FILE_APPEND);
});

$rk = new RdKafka\Producer($conf);
$rk->setLogLevel(LOG_DEBUG);
$rk->addBrokers("127.0.0.1");

$cf = new RdKafka\TopicConf();
// -1必须等所有brokers同步完成的确认 1当前服务器确认 0不确认，这里如果是0回调里的offset无返回，如果是1和-1会返回offset
$cf->set('request.required.acks', 0);
$topic = $rk->newTopic("test", $cf);
#填写消息key
$option = 'qkl';
for ($i = 0; $i < 20; $i++) {
    //RD_KAFKA_PARTITION_UA自动选择分区，可以直接填写partition编号
    //$option可选
    $topic->produce(RD_KAFKA_PARTITION_UA, 0, "qkl . $i", $option);
}

```
- low consumer
```
$conf = new RdKafka\Conf();
#正确回调
$conf->setDrMsgCb(function ($kafka, $message) {
    file_put_contents("./c_dr_cb.log", var_export($message, true), FILE_APPEND);
});
#错误回调
$conf->setErrorCb(function ($kafka, $err, $reason) {
    file_put_contents("./err_cb.log", sprintf("Kafka error: %s (reason: %s)", rd_kafka_err2str($err), $reason).PHP_EOL, FILE_APPEND);
});

//设置消费组
$conf->set('group.id', 'myConsumerGroup');

$rk = new RdKafka\Consumer($conf);
$rk->addBrokers("127.0.0.1");

$topicConf = new RdKafka\TopicConf();
$topicConf->set('request.required.acks', 1);
//在interval.ms的时间内自动提交确认
//$topicConf->set('auto.commit.enable', 1);
//$topicConf->set('auto.commit.interval.ms', 100);
$topicConf->set('auto.commit.enable', 0);

// 设置offset的存储为broker
 $topicConf->set('offset.store.method', 'broker');

//smallest：简单理解为从头开始消费，其实等价于上面的 earliest
//largest：简单理解为从最新的开始消费，其实等价于上面的 latest
//$topicConf->set('auto.offset.reset', 'smallest');

$topic = $rk->newTopic("test", $topicConf);

// 参数1消费分区0
// RD_KAFKA_OFFSET_BEGINNING 重头开始消费
// RD_KAFKA_OFFSET_STORED 最后一条消费的offset记录开始消费
// RD_KAFKA_OFFSET_END 最后一条消费
$topic->consumeStart(0, RD_KAFKA_OFFSET_BEGINNING);
//$topic->consumeStart(0, RD_KAFKA_OFFSET_END); //
//$topic->consumeStart(0, RD_KAFKA_OFFSET_STORED);

while (true) {
    //参数1表示消费分区，这里是分区0
    //参数2表示同步阻塞多久
    $message = $topic->consume(0, 12 * 1000);
    if (is_null($message)) {
        sleep(1);
        echo "No more messages\n";
        continue;
    }
    switch ($message->err) {
        case RD_KAFKA_RESP_ERR_NO_ERROR:
            var_dump($message);
            break;
        case RD_KAFKA_RESP_ERR__PARTITION_EOF:
            echo "No more messages; will wait for more\n";
            break;
        case RD_KAFKA_RESP_ERR__TIMED_OUT:
            echo "Timed out\n";
            break;
        default:
            throw new \Exception($message->errstr(), $message->err);
            break;
    }
}
```
- high consumer
```
$conf = new \RdKafka\Conf();
function rebalance(\RdKafka\KafkaConsumer $kafka, $err, array $partitions = null) {
    switch ($err) {
        case RD_KAFKA_RESP_ERR__ASSIGN_PARTITIONS:
            echo "Assign: ";
            var_dump($partitions);
//            $kafka->assign($partitions);
            $kafka->assign([new RdKafka\TopicPartition("test", 0, 0)]);
            break;

        case RD_KAFKA_RESP_ERR__REVOKE_PARTITIONS:
            echo "Revoke: ";
            var_dump($partitions);
            $kafka->assign(NULL);
            break;

        default:
            throw new \Exception($err);
    }
}

// Set a rebalance callback to log partition assignments (optional)
$conf->setRebalanceCb(function(\RdKafka\KafkaConsumer $kafka, $err, array $partitions = null) {
    rebalance($kafka, $err, $partitions);
});

// Configure the group.id. All consumer with the same group.id will consume
// different partitions.
$conf->set('group.id', 'test-110-g100');

// Initial list of Kafka brokers
$conf->set('metadata.broker.list', '127.0.0.1');

$topicConf = new \RdKafka\TopicConf();

$topicConf->set('request.required.acks', 1);
//在interval.ms的时间内自动提交
//$topicConf->set('auto.commit.enable', 1);
//$topicConf->set('auto.commit.interval.ms', 100);

// 设置offset的存储为broker
// $topicConf->set('offset.store.method', 'broker');

// Set where to start consuming messages when there is no initial offset in
// offset store or the desired offset is out of range.
// 'smallest': start from the beginning
$topicConf->set('auto.offset.reset', 'smallest');

// Set the configuration to use for subscribed/assigned topics
$conf->setDefaultTopicConf($topicConf);

$consumer = new \RdKafka\KafkaConsumer($conf);

//$KafkaConsumerTopic = $consumer->newTopic('qkl01', $topicConf);

// Subscribe to topic 'test'
$consumer->subscribe(['qkl01','test']);

echo "Waiting for partition assignment... (make take some time when quickly re-joining the group after leaving it.)\n";

while (true) {
    $message = $consumer->consume(120*1000);
    switch ($message->err) {
        case RD_KAFKA_RESP_ERR_NO_ERROR:
            var_dump($message);
//            $consumer->commit($message);
//            $KafkaConsumerTopic->offsetStore(0, 20);
            break;
        case RD_KAFKA_RESP_ERR__PARTITION_EOF:
            echo "No more messages; will wait for more\n";
            break;
        case RD_KAFKA_RESP_ERR__TIMED_OUT:
            echo "Timed out\n";
            break;
        default:
            throw new \Exception($message->errstr(), $message->err);
            break;
    }
}
```
- 查看服务器数据
```
$conf = new RdKafka\Conf();
$conf->setDrMsgCb(function ($kafka, $message) {
    file_put_contents("./xx.log", var_export($message, true), FILE_APPEND);
});
$conf->setErrorCb(function ($kafka, $err, $reason) {
    printf("Kafka error: %s (reason: %s)\n", rd_kafka_err2str($err), $reason);
});

$conf->set('group.id', 'myConsumerGroup');

$rk = new RdKafka\Consumer($conf);
$rk->addBrokers("127.0.0.1");

$allInfo = $rk->metadata(true, NULL, 60e3);

$topics = $allInfo->getTopics();

echo rd_kafka_offset_tail(100);
echo "--";

echo count($topics);
echo "--";

foreach ($topics as $topic) {
    $topicName = $topic->getTopic();
    if ($topicName == "__consumer_offsets") {
        continue ;
    }
    $partitions = $topic->getPartitions();
    foreach ($partitions as $partition) {
//        $rf = new ReflectionClass(get_class($partition));
//        foreach ($rf->getMethods() as $f) {
//            var_dump($f);
//        }
//        die();
        $topPartition = new RdKafka\TopicPartition($topicName, $partition->getId());
        echo  "当前的话题：" . ($topPartition->getTopic()) . " - " . $partition->getId() . " - ";
        echo  "offset：" . ($topPartition->getOffset()) . PHP_EOL;
    }
}
```

##参考文档
- [kafka文档](http://kafka.apache.org/documentation/)
- [php-kafka文档](https://arnaud.le-blanc.net/php-rdkafka/phpdoc/index.html)
- [rdkafka库](https://github.com/edenhill/librdkafka/blob/master/CONFIGURATION.md)
