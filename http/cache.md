缓存：
缓存产生的原因：冗余数据传输 | 带宽瓶颈 | 瞬间拥塞| 距离时延
命中和未命中：
再验证：如果内容没有变化，则返回304
首部： If-Modified-Since用来验证再命中，再验证命中/未命中/对象被删除（404）

区分缓存命中还是访问原始服务器命中：返回码都是200

客户端来区分是缓存命中还是原始服务器命中，使用Date首部，将响应中的Date首部与当前时间比较，如果响应中的日期比较早，客户端就可以认为这是一条缓存响应，也可以通过Age首部来验证

缓存类型：私有 公有 代理缓存 网状缓存 

缓存处理步骤：
接收-解析-查询-新鲜度检测-创建响应-发送-日志 P181

新鲜度检测：
文档过期：Expire首部 Cache-control首部
Expires:Fri,05 Jul 2002,05:00:00 GMT //2006-06-29，起始时间
Cache-Control:max-age=484200 //秒钟，需要时间

再验证：
内容发生了变化，获取一份新的文档副本
内容没有发生变化，缓存只需获取首部，包含一个新的过期日期，并对缓存中的首部进行更新就行了

用条件方法进行再验证：
If-Modified-Since:<date> 有变化200 没变化304  //首部 P188
If-None-Match:<tags> v2.6 304 命中   //实体标签

方案：
送回实体标签，使用实体标签验证器
送回了一个last-modified，客户端就可以使用If-Modified-Since验证
如果两者都提供了，就是使用以上两种方法验证

控制缓存：
Cache-Control:no-store|no-cache|must-revalidate|max-age|Expires

no-store 禁止缓存对响应进行复制
no-cache 可以存储在本地缓存区中，只是在与原服务器进行新鲜度验证之前，缓存不能将其提供给客户端使用
max-age 新鲜状态的秒数
Expires 实际过期时间，而不是秒数
must-revalidate 事先没有跟原始服务器进行再验证的情况下，不能提供这个对象的陈旧副本，如果服务器不可用，返回504

客户端的新鲜度限制：
Cache-Control:max-stale
Cache-Control:max-stale/min-fresh/max-age/no-cache/no-store/only-if-cached P194

设置缓存控制:
mod_heads
<Files *.html>
   Header set Cache-control no-cache
</Files>

mod_expires
ExpiresDefault A3600
ExpriesDefault M86400
ExpiresDefault "access plus 1 week"
ExpiresByType text/html "modification plus 2 days 6 hours 12 minutes"

mod_cern_meta

通过HTTP-EQUIV控制html缓存
<HEAD>
  <META HTTP-EQUIV="Cache-control" CONTENT="no-cache">
</HEAD>

缓存与广告点击
正常情况下，缓存会吸收点击，降低广告费用
缓存清除技术，可以通过缓存与服务器命中验证来计算点击，但不提交数据，也可以通过缓存日志等方法来计算，另有一种Meter首部，会周期性的对特定URL命中次数回送给服务器

网关：进去一条请求，出来一个响应
分类：协议网关 资源网关
隧道：用connect请求建立隧道
发送connect请求-->打开443的TCP连接->建立连接-->返回http连接就绪报文-->双向通信，传递数据-->连接关闭

用隧道将非HTTP流量，如加密的SSL传过端口过滤防火墙，为了防止恶意入侵，可以指定特定网关来打开隧道

中继：负责处理HTTP中建立连接的部分，然后对字节进行盲转发，不识别客户端的connection，也不识别服务器回送的keep-alive