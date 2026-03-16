# PS指令详解
 查看所有进程（最常用）
ps aux

 查看进程树（查父子关系）
ps -ef

 组合grep查找特定进程
ps aux | grep nginx

 按CPU占用排序（排查卡顿）
ps aux --sort=-%cpu | head -5

# 关键字段含义（面试）
字段	含义	运维场景
PID	进程ID	`kill -9 PID` 杀进程
%CPU	CPU占用率	排查哪个进程吃CPU
%MEM	内存占用	排查内存泄漏
STAT	状态	S(睡眠) R(运行) Z(僵尸进程)
COMMAND	启动命令	看是不是病毒/挖矿程序

# shell脚本实战：cpu监控
文件名：check_process.sh
vim check_process.sh
编辑：#!/bin/bash
（# 监控脚本：找出CPU占用前5的进程
echo "$(date) 系统高CPU进程：" > /tmp/ps_log.txt
ps aux --sort=-%cpu | head -6 | grep -v "PID" >> /tmp/ps_log.txt
cat /tmp/ps_log.txt）
然后赋予该文件执行权限
chmod +x check_process.sh
然后运行
./check_process.sh
然后查看日志输出
cat /tmp/ps_log.txt
后续可拓展：加入crontab定时任务：每五分钟自动执行
增加磁盘监控版本
加入邮件报警功能（CPU>80%时发送）