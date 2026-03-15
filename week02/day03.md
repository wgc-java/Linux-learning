# 配置静态IP
## 步骤1：找到网卡配置文件
 查看网卡名称（通常是ens33或eth0）
ls /etc/sysconfig/network-scripts/
 目标文件：ifcfg-ens33
## 步骤2： 修改配置文件
 查看网卡名称（通常是ens33或eth0）
ls /etc/sysconfig/network-scripts/
 目标文件：ifcfg-ens33
## 修改以下行：
 将BOOTPROTO从dhcp改为static
BOOTPROTO=static

 将ONBOOT从no改为yes（开机自启）
ONBOOT=yes

 添加以下配置（IP根据你的VMware网段调整）
IPADDR=192.168.44.130      # IP地址（44是你的网段，130随意填2-254）
NETMASK=255.255.255.0      # 子网掩码（固定）
GATEWAY=192.168.44.2       # 网关（VMware NAT模式固定是.2）
DNS1=8.8.8.8              # 谷歌DNS
DNS2=114.114.114.114      # 国内DNS（备用）
## 步骤3  重启网络服务
 CentOS 7/8
sudo systemctl restart network

 或者（旧版）
sudo service network restart
## 步骤4 验证
ip addr
ping www.baidu.com

# 设置主机名和hosts映射
 临时生效（重启失效）
hostname wgc-server

 永久生效（写入配置文件）
sudo vim /etc/hostname
 写入：wgc-server
 重启后生效：sudo reboot
## hosts映射 （本地DNS）
sudo vim /etc/hosts
添加：
192.168.xx.130  wgc-server
192.168.xx.1 host-machine
测试：
ping wgc-server 


# 5. 今日跳过待回补（重要）

063-66集 NAT网络原理图

跳过原因：前置知识不足（需计算机网络基础）

计划回看时间：2026年4月10日-15日

前置条件：学完湖科大微课堂第1-20集

当前掌握：知道"VMware选NAT模式就能上网"即可
