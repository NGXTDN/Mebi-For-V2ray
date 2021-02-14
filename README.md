# Mebi-For-V2ray：对接SSPANEL，免费V2ray后端

用于对接sspanel，免费v2ray后端仅供学习测试使用。免费后端不做优化、不修改v2ray原程序、不做一键脚本。

后端简介
本站写的用于对接sspanel的免费v2后端，不做优化、不修改v2ray原程序、不做一键脚本。对接方式只提供数据库对接，请开放面板数据库相应端口，数据库默认端口 3306。

后端支持
节点服务器信息上报
节点服务器流量上报
节点服务器流量限制
用户流量上报
用户动态增删
用户在线ip数上报
支持用户分组
支持用户限速
支持用户ip数限制
支持获取用户真实ip
支持审计
后端不支持
封禁ip
不提供VLESS
一键脚本
vless协议仍处于公测阶段，所以就不提供接入了。

搭建
安装v2ray
使用官方提供的一键脚本

bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)
安装后v2ray默认路径：/usr/local/bin/v2ray

安装后端
mkdir -p /usr/local/mebi /etc/mebi
wget --no-check-certificate -O /usr/local/mebi/mebi.tar.gz https://www.mebi.me/room/software/v2ray/mebi.tar.gz
tar -xvf /usr/local/mebi/mebi.tar.gz -C /usr/local/mebi/
mv /usr/local/mebi/mebi.conf /etc/mebi/mebi.conf
配置文件默认在 /etc/mebi/mebi.conf

# 数据库对接
db_host=www.mebi.me                           # 数据库地址
db_port=3306                                 # 数据库端口
db_name=root                               # 数据库名
db_user=root                                 # 数据库用户名
db_password=abcd1234                        # 数据库密码

node_id=385                                    # 节点id
interval=60                                   # 监控周期
speed_punish_time=3                          # 超过限速惩罚时间
connector_punish_time=60                      # 超过连接数惩罚时间

certificateFile=                              # 公钥证书
keyFile=                                      # 私钥证书
安装流量统计
节点服务器流量统计依靠vnstat，优点是服务器重启不会丢失流量信息，能够以任一一天按月统计流量。

# CentOS
yum install vnstat -y
# Debian/Ubuntu
apt-get install vnstat -y
vnstat默认 1号为每月的统计日。你还可以设定统计日为10号，那么按月统计为2月10号——3月9号（包含9号全天），这就很方便流量重置日就是账单日的服务器。修改统计日方法：打开 /etc/vnstat.conf

MonthRotate 1
修改1为你想要的流量充值重置日，然后重启vnstat

service vnstat restart
安装iptables
vmess是无状态协议，加上不能修改v2ray程序，所以限速、限设备数只能借助第三方工具。本后端需要iptables实现功能，请安装iptables。Centos7等系统默认安装Firewalld，请先将其卸载再安装iptables。

# CentOS
yum install iptables -y
# Debian/Ubuntu
apt-get install iptables -y
基本逻辑：每隔一段时间（interval）用户达到预设限速值，会对该用户下所有ip封禁一定时间（speed_punish_time）并自动恢复。每隔一段时间（interval）用户达到预设设备数值，会对该用户超过设备数部分的、较早时间的ip进行封禁一定时间（connector_punish_tim）并自动恢复。

配置
地址写法
tcp

ip;端口;alertid;tcp;;server=demo.mebi.me
server是用户连接用的地址，可以不写。

ws

ip;端口;alertid;ws;;path=/path|server=demo.mebi.me
ws必须要有path，必须以"/"开头，根路径可以写：path=/。server同上。

h2+tls

ip;端口;alertid;h2;tls;path=/|server=demo.mebi.me
h2可以写成http。h2必须带tls加密，此为h2特性。path同上。server必写。

tcp+tls

ip;端口;alertid;tcp;tls;server=demo.mebi.me
参考同上。server必写。

ws+tls

ip;端口;alertid;ws;tls;path=/path|server=demo.mebi.me
参考同上。server必写。

中转

ip;端口;alertid;ws;tls;path=/path|server=zz.mebi.me|outside_port=10000
此中转适用上面所有配置，只需在原地址上加上 outside_port。outside_port为中转端口，server改为中转域名。

配置ssl

然后将 fullchain.cert 和 域名.key 路径分别填入配置文件里的certificateFile和keyFile。

启动mebi后端
/usr/local/mebi/mebi -config /etc/mebi/mebi.conf
config参数后接配置文件。

将后端加入系统环境
编辑 /etc/profile，在文末添加如下

export MEBIDIR=/usr/local/mebi
export PATH=$PATH:MEBIDIR
执行如下命令生效

source /etc/profile
现在可以直接调用mebi

mebi -config /etc/mebi/mebi.conf
