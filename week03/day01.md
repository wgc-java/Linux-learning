Linux 服务管理（Daemon Management）
# 一、核心概念

服务（Service）：后台运行的程序，也叫守护进程（daemon），如 sshd、mysqld、nginx

运行级别（Runlevel）：CentOS 6概念，7+版本已弱化，改用targets

systemd：CentOS 7+ 的系统和服务管理器，替代了旧的 SysV init
# 二、systemctl 命令详解（CentOS 7+ 重点）
## 1. 服务状态管理
启动服务
systemctl start 服务名

停止服务
systemctl stop 服务名

重启服务
systemctl restart 服务名

查看状态（重要，看Active: active/inactive）
systemctl status 服务名

重新加载配置（不中断服务）
systemctl reload 服务名
## 2.开机自启管理
设置开机自启
systemctl enable 服务名

取消开机自启
systemctl disable 服务名

查看是否开机自启
systemctl is-enabled 服务名
## 3.查看所有服务
查看所有已启动的服务
systemctl list-units --type=service --state=running

查看所有服务（包括未启动）
systemctl list-unit-files --type=service
# 三、常见服务名对照
服务名	功能	常用场景
`sshd`	SSH远程连接	远程管理服务器
`firewalld`	防火墙（CentOS 7+）	端口管理
`iptables`	防火墙（旧版）	旧系统维护
`mysqld` / `mariadb`	数据库	数据存储
`nginx` / `httpd`	Web服务器	网站部署
`crond`	定时任务	定时脚本执行
`network`	网络服务	网卡管理
# 四、防火墙基础
firewalld
查看防火墙状态
systemctl status firewalld

启动/停止/重启防火墙
systemctl start firewalld
systemctl stop firewalld  # 临时关闭，重启失效

禁止开机自启（危险，生产环境慎用）
systemctl disable firewalld

查看开放端口
firewall-cmd --list-ports

开放80端口（永久生效）
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --reload  # 重载配置
### 与iptables关系
CentOS 7+ 默认使用  firewalld ，但底层仍调用  iptables

二者不能同时运行，会产生冲突
# 五、实战练习（今晚在家IDEA实操）
练习1：管理SSH服务
systemctl status sshd          # 查看SSH状态
sudo systemctl restart sshd    # 重启SSH（需sudo）
systemctl is-enabled sshd      # 查看是否开机自启

练习2：模拟Web服务器部署流程
sudo systemctl start httpd     # 启动Apache（需先安装）
sudo systemctl enable httpd    # 设置开机自启
curl localhost                 # 测试是否成功

练习3：查看所有运行中的服务
systemctl list-units --type=service --state=running | grep running
# 六、面试高频考点
### Q：如何设置服务开机自启？
CentOS 7+ 答案
systemctl enable 服务名

CentOS 6 答案
chkconfig 服务名 on
### Q：如何查看某个服务是否运行？
systemctl status 服务名
或看进程
ps aux | grep 服务名
### Q：service和systemctl的区别？

service ：旧命令，立即生效但不持久，兼容脚本

systemctl ：新命令，统一管理系统，支持依赖关系
# 七、注意事项
### 1.
权限：管理服务通常需要  sudo  或 root 权限
### 2.
服务名后缀：Linux服务名通常带  d （daemon），如  sshd  不是  ssh
### 3.
防火墙谨慎：生产环境不要随意  stop firewalld ，应开放特定端口而非全关
