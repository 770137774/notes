###1、fastcgi配置
fastcgi的参数一般会在单独文件中初始化，在nginx的配置文件中引入
```
#fastcgi_param
fastcgi_param  QUERY_STRING       $query_string;
fastcgi_param  REQUEST_METHOD     $request_method;
fastcgi_param  CONTENT_TYPE       $content_type;
fastcgi_param  CONTENT_LENGTH     $content_length;

fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
fastcgi_param  REQUEST_URI        $request_uri;
fastcgi_param  DOCUMENT_URI       $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;
fastcgi_param  REQUEST_SCHEME     $scheme;
fastcgi_param  HTTPS              $https if_not_empty;

fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;

fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;

# PHP only, required if PHP was built with --enable-force-cgi-redirect
fastcgi_param  REDIRECT_STATUS    200;

```
```
user nobody;
worker_processes  8;
worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;
worker_rlimit_nofile 8192;

error_log   "/home/work/nginx/logs/error_log"   notice;
pid         "/home/work/nginx/var/nginx.pid";

events {
    use epoll;
    worker_connections  8192; 
}

http {
    include       mime.types; 
    #include       upstream.conf;
    default_type  application/octet-stream;
    

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" "$http_cookie" "$http_user_agent" '
                      '$request_time $remote_addr $server_addr $upstream_addr $host '
                      '"$http_x_forwarded_for" $uuid  $msec $msec';

    #lua_package_path "$prefix/lua/?.lua;/test/lua/?.lua;";
    #init_by_lua_file lua/util.lua;
    access_log  "/home/work/nginx/logs/access_log"  main;

    client_body_temp_path /home/work/nginx/cache/client_body;
    fastcgi_temp_path /home/work/nginx/cache/fastcgi;
    proxy_temp_path /home/work/nginx/cache/proxy;
    uwsgi_temp_path /home/work/nginx/cache/uwsgi;
    scgi_temp_path /home/work/nginx/cache/scgi;

    server_names_hash_bucket_size 128;
    client_header_buffer_size 4k;
    large_client_header_buffers 4 32k;
    client_max_body_size 100m;
    client_body_buffer_size 513k;

    sendfile        off;
    tcp_nopush      on;
    tcp_nodelay     on;

    fastcgi_connect_timeout 5;
    #fastcgi_send_timeout 10;
    fastcgi_send_timeout 30;
    #fastcgi_read_timeout 10;
    fastcgi_read_timeout 30;
    fastcgi_buffer_size 64k;
    #fastcgi_buffers 4 64k;
    fastcgi_buffers 8 128k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;
    fastcgi_intercept_errors on;
    #fastcgi_param  HTTPS $fastcgi_https;

    keepalive_timeout  0;
    #keepalive_timeout  65;

    gzip on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_http_version 1.0;
    gzip_comp_level 2;
    gzip_types text/plain application/x-javascript text/css application/xml;
    gzip_vary on;

    proxy_connect_timeout 15s;
    proxy_read_timeout 24s;
    proxy_send_timeout 30s;
    proxy_buffer_size 1024k;
    proxy_buffers 16 1024k;
    proxy_busy_buffers_size 2048k;
    proxy_temp_file_write_size 2048k;
    #proxy_buffer_size 64k;
    #proxy_buffering on;
    #proxy_buffers 4 64k;
    #proxy_busy_buffers_size 128k;
    #proxy_temp_file_write_size 128k;
    proxy_set_header Host $http_host;
    proxy_set_header LOGID $uuid;
#    fastcgi_param PHPLOIGID  $uuid;
    include fastcgi_params;//这行就是引入fastcgi的配置
    uninitialized_variable_warn off;
    #proxy_redirect http:// https://;

    set_real_ip_from 10.0.0.0/8;
    real_ip_header CLIENTIP;
    #include vhost/php.conf;
    include vhost/*.conf;
}

```
###2、作用域和修改
分为三级，http、server、location，一般来讲大括号内有效  
使用set 来定义  
```
server {
    listen              8001;
    server_name         app.zhugexuetang.com;
    set $php_upstream   'unix:/home/work/dsp/var/php-cgi.sock';

    location / {
        root /home/work/dsp/php/phplib/yii/web/;

        fastcgi_pass    $php_upstream;
        fastcgi_index   index.php;
        include         fastcgi.conf;
        fastcgi_param   SCRIPT_FILENAME $document_root/index.php;//定义
        fastcgi_param   APPNAME zgxt_admin;//定义


        if ($http_LOGID != '' ) {    
            set $uuid $http_LOGID;
        }
        if ($http_LOGID = '' ) {    
           set  $uuid $request_id;
        }

        add_header 'Access-Control-Allow-Methods' 'GET,OPTIONS,PUT,DELETE' always;
        add_header 'Access-Control-Allow-Credentials' 'true' always;
        add_header 'Access-Control-Allow-Origin' '$http_origin' always;
        add_header 'Access-Control-Expose-Headers' 'Access-Token' always;
        add_header 'Access-Control-Allow-Headers' 'Authorization,DNT,User-Agent,Keep-Alive,Content-Type,accept,origin,X-Requested-With, Access-Token,Token,Device,backend,City,apiVersion' always;
    }
}

```
###3、php端访问
```
echo $_SERVER['APPNAME']
```
输出：
>zgxt_admin

参考
 - https://www.cnblogs.com/dongzhanyi123/p/12081448.html