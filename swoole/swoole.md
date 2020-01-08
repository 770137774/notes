###1、chan使用
```
<?php
$serv = new \swoole_http_server("127.0.0.1", 9503, SWOOLE_BASE);

$serv->on('request', function ($req, $resp) {
    Co::set([
        'trace_flags' => SWOOLE_TRACE_CLOSE
    ]);


    $chan = new \Swoole\Coroutine\Channel();
    function task1(\Swoole\Coroutine\Channel $chan) {
        echo 11;
        Co::sleep(0.005);
        $chan->push([__METHOD__=>__LINE__]);
    }
    function task2(\Swoole\Coroutine\Channel $chan) {
        echo 22;
        Co::sleep(0.005);
        $chan->push([__METHOD__=>__LINE__]);
    }
    go("task1", $chan);
    go("task2", $chan);
    go(function () use ($chan){
        echo  33;
        var_dump($chan->isEmpty());
        while (1){
            var_dump($chan->pop());//pop之后isEmpty会变成false
            var_dump($chan->isEmpty());
            Co::sleep(1);
        }
    });
});
$serv->start();
```
###2、子协程
```
$serv = new \swoole_http_server("127.0.0.1", 9503, SWOOLE_BASE);

$serv->on('request', function ($req, $resp) {
    $chan = new chan(2);
    go(function () use ($chan) {
        $cli = new Swoole\Coroutine\Http\Client('www.qq.com', 80);
            $cli->set(['timeout' => 10]);
            $cli->setHeaders([
            'Host' => "www.qq.com",
            "User-Agent" => 'Chrome/49.0.2587.3',
            'Accept' => 'text/html,application/xhtml+xml,application/xml',
            'Accept-Encoding' => 'gzip',
        ]);
        $ret = $cli->get('/');
        $chan->push(['www.qq.com' => $cli->body]);
    });

    go(function () use ($chan) {
        $cli = new Swoole\Coroutine\Http\Client('www.163.com', 80);
        $cli->set(['timeout' => 10]);
        $cli->setHeaders([
            'Host' => "www.163.com",
            "User-Agent" => 'Chrome/49.0.2587.3',
            'Accept' => 'text/html,application/xhtml+xml,application/xml',
            'Accept-Encoding' => 'gzip',
        ]);
        $ret = $cli->get('/');
        $chan->push(['www.163.com' => $cli->body]);
    });

    $result = [];
    for ($i = 0; $i < 2; $i++)
    {
        $result += $chan->pop();
    }
    $resp->end(json_encode($result));
});
$serv->start();
```