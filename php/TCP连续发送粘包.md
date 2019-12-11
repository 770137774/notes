#swoole tcp
- 连接建立后连续发送数据，如果协议结尾的‘\r\n\r\n‘之后不加一个空白字符会粘包
例如：
```
$client = new \swoole_client(SWOOLE_SOCK_TCP);
if (!$client->connect('127.0.0.1', 18307, 10))
{
    exit("connect sender failed. Error: {$client->errCode}\n");
}
foreach ($all as $value){
    $client->send(json_encode([
            "method" => "1.0::App\Rpc\Lib\MsgInterface::send",
            "params" => [
                $params
            ],
        ])."\r\n\r\n ");
//    $result = $client->recv();
//    echo $params['mobile'] . "---" . $result."\n";
}
$client->close();
```
>ps：如果是同步服务器每次调用recv()可以避免，一部服务器一定要注意，不加空格发送过快可能导致tcp粘连