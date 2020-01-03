###查看linux库
```
ldconfig -p | grep xxx
``` 
说明：使用 ldconfig -p 命令用来打印出当前缓存所保存的所有库的名字，然后用管道符传递给 grep lts 命令用于解析出 
liblts.so 共享库的路径是否已加入缓存中。
###sudo带当前用户环境变量执行
```
sudo -E
```