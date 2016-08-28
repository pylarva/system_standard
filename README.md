# 操作系统安装及初始化规范
> v1.0
## 操作系统安装规范
### 初始化操作



##系统初始化规范

###目录规范
* 脚本放置目录：/opt/shell
* 脚本日志目录：/opt/shell/log
* 脚本锁文件目录：/opt/shell/lock

###服务安装规范
1. 源码安装路径：/usr/local/appname.version
2. 创建软链接： ln -s /usr/local/appname.version /usr/local/appname