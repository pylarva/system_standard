# 操作系统安装及初始化规范
> v1.0
## 操作系统安装流程
![操作系统安装流程](http://i.imgur.com/5t3Nir8.png)

1. 服务器采购
2. 服务器验收、运维验收负责人签字
3. 服务器上架接通电源并配置raid磁盘阵列
4. 粘贴资产编号、CMDB资产位置信息录入
5. 将服务器接入接入层交换机、划分装机vlan
6. 记录服务器mac地址，加入部署系统system
5. 自动化部署系统
6. 确认系统安装结果

######自动化系统部署平台，装机示例

* IP:172.16.2.151
* 主机名：linux-node2.lichengbing.cn
* 掩码：255.255.255.0
* 网关：172.16.2.150
* DNS：172.16.2.150


<pre>
cobbler system add --name=linux-node2.lichengbing.cn --mac=00:0C:29:8E:A9:4F --profile=CentOS-7-x86_64 --ip-address=172.16.2.151 --subnet=255.255.255.0 --gateway=172.16.2.151 --interface=eth0 --static=1 --hostname=linux-node2.lichengbing.cn --name-servers="172.16.2.150" --kickstart=/var/lib/cobbler/kickstarts/CentOS-7-x86_64.cfg
</pre>

<table>
   <tr>
      <td>机房</td>
	  <td>资产号</td>
	  <td>IP</td>
	  <td>MAC</td>
	  <td>主机名</td>
	  <td>系统</td>
	  <td>上架日期</td>
	  <td>维护人</td>
   </tr>
   <tr>
      <td>IDC </td>
	  <td> </td>
	  <td> </td>
	  <td> </td>
	  <td> </td>
	  <td> </td>
	  <td> </td>
	  <td> </td>
   </tr>
</table>
## 操作系统安装规范
1. 当前我公司使用操作系统均为CentOS 6和CentOS 7，均使用x86_64位系统，需要使用公司cobbler进行自动化安装，禁止自定义设置
2. 版本选择，数据库统一使用cobbler上CentOS-7-DB专用的profile，其他web使用CentOS-7-web系统
3. 如有windows server服务器安装需求，需单独向系统安全部申请安装权限
4. 任何系统安装验收完成后，不得私自进行系统升降级操作
5. 重装操作需在工作平台提交流程审核，上级审核通过才允许操作

##系统初始化规范
###初始化操作
* 时间同步服务器IP：172.16.2.150
* 设置DNS（172.16.2.150）
* 安装Zabiix Agent、Zabbix Server 172.16.2.150
* 安装Saltstack Minion：Saltstack Server：172.16.2.150
* 创建sudo用户sdg，禁止root登陆
* 修改默认语言设置  
<pre>
echo "LANG=zh_CN.UTF-8">/etc/sysconfig/i18n
</pre> 
* history命令记录时间
<pre>
export HISTTIMEFORMAT="%F %T `whoami`"
</pre>
* 日志记录操作
<pre>
export PROMPT_COMMAND='{ msg=$(history 1 | { read x y;echo $y; });logger"[euid=$(whoami)]":$(who am i):[`pwd`]"$msg"; }'
</pre>
* 内核参数优化
<pre>
net.ipv4.tcp_fin_timeout=2
net.ipv4.tcp_tw_reuse=1
net.ipv4.tcp_tw_recycle=1
net.ipv4.tcp_syncookies=1
net.ipv4.tcp_keepalive_time=600
net.ipv4.ip_local_port_range = 4000  65000
net.ipv4.tcp_max_syn_backlog= 16384
net.ipv4.tcp_max_tw_buckets=36000
net.ipv4.route.gc_timeout=100
net.ipv4.tcp_syn_retries=1
net.ipv4.tcp_synack_retries=1
net.core.somaxconn=16384
net.core.netdev_max_backlog=16384
net.ipv4.tcp_max_orphans=16384
</pre>
* yum仓库
<pre>
cd /etc/yum.repos.d
mkdir -p repo_back && mv *repo repo_back/
vi new.repo
[new]
name=server
baseurl=http://10.0.0.101
enable=1
gpgcheck=0
yum clean all
yum list
</pre>

 

###目录规范
* 脚本放置目录：/opt/shell
* 脚本日志目录：/opt/shell/log
* 脚本锁文件目录：/opt/shell/lock
* 软件源码目录：~/tools


###服务安装规范
1. 源码安装路径：/usr/local/appname.version
2. 创建软链接： ln -s /usr/local/appname.version /usr/local/appname


###主机名规范
   **机房名-项目-角色-服务-集群-节点.域名**

    例如：
    idc01-xxshop-api-nginx-bj-node1.shop.com

###服务启动规范
	所有服务，统一使用www用户，uid为666，除负载均衡需要监听80端口使用root以外，其余服务一律使用www用户启动,并且使用大于1024端口

