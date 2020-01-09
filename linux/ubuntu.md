###1、Ubuntu自带输入法无法选择候选字
- 删除配置文件 rm -rf ~/.cache/ibus/libpinyin
- ibus-daemon -drx

###2、apt
sudo apt --fix-broken install
自动检查并安装依赖
