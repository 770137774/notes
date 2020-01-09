###1、认证授权样例
https://github.com/floodliu/bce-samples.git
###2、百度kafka生产者
```
class KafkaProducer
{
    public static $producer;
    public function getProducer($options = [])
    {
        if (!isset($options['topic']) ) {
            $options['topic'] = 'e67d9e19563e4ea7bcb590baaab8fcbb__kafka';
        }
        if(!isset(self::$producer[$options['topic']])){
            $topic = $options['topic'];
            $broker = isset($options['broker']) ? $options['broker'] : 'kafka.bj.baidubce.com:9091';
            $security_protocol = isset($options['security_protocol']) ? $options['security_protocol'] : 'ssl';
            $client_pem = isset($options['client_pem']) ? $options['client_pem'] :'/data/msg/client.pem';
            $client_key = isset($options['client_key']) ? $options['client_key'] :'/data/msg/client.key';
            $ca_pem = isset($options['ca_pem']) ? $options['ca_pem'] : '/data/msg/ca.pem';

            $conf = new \RdKafka\Conf();
            $conf->set('metadata.broker.list', $broker);
            $conf->set('security.protocol', $security_protocol);
            $conf->set('ssl.certificate.location', $client_pem);
            $conf->set('ssl.key.location', $client_key);
            $conf->set('ssl.ca.location', $ca_pem);
            $conf->set('request.required.acks', '1');
            $conf->setDrMsgCb(function ($kafka, $message) {
                $logger = new KafkaLogsDao();
                $logger->loggerBySql(var_export($message, true).PHP_EOL,1);
            });
            $conf->setErrorCb(function ($kafka, $err, $reason){
                $logger = new KafkaLogsDao();
                $logger->loggerBySql(sprintf("Kafka error: %s (reason: %s)", rd_kafka_err2str($err), $reason).PHP_EOL,0);
            });
            $rk = new \RdKafka\Producer($conf);
//            $rk->setLogLevel(LOG_DEBUG);
            $rk->addBrokers($broker);
            self::$producer[$options['topic']] = $rk;
        }
        return self::$producer[$options['topic']];
    }
    /**
     *
     * @options $options  = [
        "topic:",
        "broker::",         // default = kafka.bj.baidubce.com:9091. The broker address can be found in https://cloud.baidu.com/doc/Kafka/QuickGuide.html
        "security_protocol::",  // default = ssl. SSL is required to access Baidu Kafka service.
        "client_pem::",     // default = client.pem. File path to client.pem provided in kafka-key.zip from console.
        "client_key::",     // default = client.key. File path to client.key provided in kafka-key.zip from console.
        "ca_pem::",         // default = ca.pem. File path to ca.pem provided in kafka-key.zip from console.
        ];
     *
     */
    public function producer($data,$options = [],$key = NULL)
    {
        if (!isset($options['topic']) ) {
            $options['topic'] = 'e67d9e19563e4ea7bcb590baaab8fcbb__kafka';
        }
        $rk = $this->getProducer($options);
        $topic = $rk->newTopic($options['topic']);
        $topic->produce(RD_KAFKA_PARTITION_UA, 0, json_encode($data,JSON_UNESCAPED_UNICODE),$key);
        sgo(function () use ($rk){
            $rk->poll(-1);
        },true);
        return true;
    }
}
``` 
###3、百度kafka消费者样例
```
class KafkaConsumer
{
    /*
     *
     * @params $options = [
            "topic:",
            "broker::",         // default = kafka.bj.baidubce.com:9091. The broker address can be found in https://cloud.baidu.com/doc/Kafka/QuickGuide.html
            "security_protocol::",  // default = ssl. SSL is required to access Baidu Kafka service.
            "client_pem::",     // default = client.pem. File path to client.pem provided in kafka-key.zip from console.
            "client_key::",     // default = client.key. File path to client.key provided in kafka-key.zip from console.
            "ca_pem::",         // default = ca.pem. File path to ca.pem provided in kafka-key.zip from console.
        ];
    */
    public function consumer($options = [])
    {
        if (!isset($options['topic'])) {
            $options['topic'] = 'e67d9e19563e4ea7bcb590baaab8fcbb__kafka';
        }

        $topic = $options['topic'];
        $broker = isset($options['broker']) ? $options['broker'] : 'kafka.bj.baidubce.com:9091';
        $security_protocol = isset($options['security_protocol']) ? $options['security_protocol'] : 'ssl';
        $client_key = isset($options['client_key']) ? $options['client_key'] :'/data/msg/client.key';
        $client_pem = isset($options['client_pem']) ? $options['client_pem'] :'/data/msg/client.pem';
        $ca_pem = isset($options['ca_pem']) ? $options['ca_pem'] : '/data/msg/ca.pem';
        $group_id = 'e67d9e19563e4ea7bcb590baaab8fcbb__'.explode('__', $topic)[1];

        $conf = new \RdKafka\Conf();
        $conf->set('metadata.broker.list', $broker);
        $conf->set('group.id', $group_id);
        $conf->set('security.protocol', $security_protocol);
        $conf->set('ssl.certificate.location', $client_pem);
        $conf->set('ssl.key.location', $client_key);
        $conf->set('ssl.ca.location', $ca_pem);
        $conf->set('auto.offset.reset', 'smallest');
//        $conf->set('enable.auto.commit', 'false');
//        $conf->set('request.required.acks', '-1');
        $consumer = new \RdKafka\KafkaConsumer($conf);
        $consumer->subscribe([$topic]);
        while (true) {
            $message = $consumer->consume(120*1000);
            switch ($message->err) {
                case RD_KAFKA_RESP_ERR_NO_ERROR:
                    echo "partition:", $message->partition,", offset:", $message->offset,", ", $message->payload, "\n";
                    $this->deal($message);
//                    $consumer->commit($message);
                    break;
                case RD_KAFKA_RESP_ERR__PARTITION_EOF:
                    echo "No more messages; will wait for more\n";
                    break;
                case RD_KAFKA_RESP_ERR__TIMED_OUT:
                    echo "Timed out\n";
                    break;
                default:
//                    throw new \Exception($message->errstr(), $message->err);
                    break;
            }
        }

    }

    private function deal($message){
        $call = new KafkaCallDao();
        $res =$call->call($message);
        if($res['status']){
//            $log = new Logger('call_log');
//            $log->pushHandler(new StreamHandler('runtime/logs/dmq/call.log', Logger::INFO));
//            $log->addInfo(json_encode($res,JSON_UNESCAPED_UNICODE));
        }else{
            $log = new Logger('call_error_log');
            $log->pushHandler(new StreamHandler(env('LOG_DIR').'/logs/kafka/call_error.log', Logger::INFO));
            $log->addWarning(json_encode(['message'=>$message,'result'=>$res],JSON_UNESCAPED_UNICODE));
        }
    }
}
```