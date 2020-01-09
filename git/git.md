错误信息：
```
Clone failed
 RPC failed; curl 56 GnuTLS recv error (-54): Error in the pull function.
The remote end hung up unexpectedly
early EOF
index-pack failed
```
解决方法：

git config --global http.postBuffer 524288000

修改配置文件

gedit ~/.bashrc

然后在配置文件的最下面加上这三行

export GIT_TRACE_PACKET=1

export GIT_TRACE=1

export GIT_CURL_VERBOSE=1

然后保存退出后运行：source ~/.bashrc  是配置文件生效
 
依旧不行：

需要如下方式命令，只clone深度为一
```
$ git clone https://github.com/JGPY/large-repository.git --depth 1
$ cd large-repository
$ git fetch --unshallow
```
depth用于指定克隆深度，为1即表示只克隆最近一次commit.（git shallow clone）

git clone 默认会下载项目的完整历史版本，如果你只关心最新版的代码，而不关心之前的历史信息，可以使用 git 的浅复制功能：
$ git clone --depth=1 https://github.com/JGPY/large-repository.git

--depth=1 表示只下载最近一次的版本，使用浅复制可以大大减少下载的数据量，例如， CodeIgniter 项目完整下载有近 100MiB ，而使用浅复制只有 5MiB 多，这样即使在恶劣的网络环境下，也可以快速的获得代码。如果之后又想获取完整历史信息，可以使用下面的命令：
$ git fetch --unshallow

###2、git remote -v
```
 #查看git源
 origin  https://github.com/770137774/notes.git (fetch)
 origin  https://github.com/770137774/notes.git (push)

```
