# 一、 权限管理快速回顾
1、权限三连击命令对比
命令	作用对象	功能	示例
`chmod`	文件/目录	修改rwx权限	`chmod 755 file` 或 `chmod u+x file`
`chown`	文件/目录	修改所有者	`chown user:group file`
`chgrp`	文件/目录	修改所属组	`chgrp group file`
# 二、关键概念澄清
所有者 vs 用户：所有者是用户的一种属性（谁创建的文件）

组的作用：方便批量管理权限（如开发组、运维组）

sudo的本质：临时获得root权限执行命令

家目录（/home/xxx）：用户登录后的默认位置，与权限无关



====================================================
新内容：
# 一、crond定时任务
1. 什么是crond？

守护进程：后台自动运行的程序（类似Windows的计划任务）

作用：定时执行脚本或命令（如每天凌晨备份、每小时清理日志）
2. 核心命令
查看当前用户的定时任务
crontab -l

编辑定时任务（进入vim编辑模式）
crontab -e

删除所有定时任务（慎用）
crontab -r

查看crond服务状态（CentOS 7+）
systemctl status crond

启动/重启服务
systemctl start crond
systemctl restart crond
3、时间格式
*****command
分，时，日，月，周
4、常见时间示例
 每分钟执行
* * * * * /home/wgc/backup.sh

 每小时的第30分钟执行
30 * * * * command

 每天凌晨2点执行
0 2 * * * command

 每周日凌晨3点执行
0 3 * * 0 command

 每月1号早上8点执行
0 8 1 * * command

 每隔5分钟执行
*/5 * * * * command

 工作日（周一到周五）每小时执行
0 * * * 1-5 command
5、实际应用示例
# 示例1：每天凌晨3点备份数据库（假设脚本已写好）
0 3 * * * /home/wgc/scripts/db_backup.sh >> /var/log/db_backup.log 2>&1

# 示例2：每周清理一次临时文件（周日0点）
0 0 * * 0 rm -rf /tmp/old_files/*

# 示例3：每分钟检查磁盘使用率（超80%报警）
* * * * * df -h | grep '/dev/sda1' | awk '{print $5}' | cut -d'%' -f1 | while read use;
do [ $use -gt 80 ] && echo "Disk full" | mail -s "Alert" admin@example.com; done
6. 注意事项（易错点）

环境变量：crond执行时环境变量很少，建议脚本中使用绝对路径（如  /usr/bin/java  而非  java ）

权限：脚本需要有可执行权限（ chmod +x script.sh ）

日志：建议将输出重定向到日志文件（ >> /var/log/xxx.log 2>&1 ）

邮件：默认会发邮件给用户，不需要的话可以在文件开头加  MAILTO=""