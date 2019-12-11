##1、jsWebsocket
```
<script>
var ws = new WebSocket("ws://127.0.0.1:18308/chat");

ws.onopen = function(evt) {  //绑定连接事件
　　console.log("Connection open ...");
　　let data = '{"data":"发送的数据"}';
    ws.send(data);
};

ws.onmessage = function(evt) {//绑定收到消息事件
　　console.log( "Received Message: " + evt.data);
};

ws.onclose = function(evt) { //绑定关闭或断开连接事件
　　console.log("Connection closed.");
};
</script>
```