# Linux系统监控三剑客笔记
**日期**: 2026-03-24  
**来源**: 韩顺平Linux教程第78集补漏  
**适用**: CentOS 7/8，运维排查必备

---

## 1. vmstat - 系统整体体检表（Virtual Memory Statistics）

### 作用
查看进程、内存、交换分区、磁盘IO、CPU活动的整体情况，判断系统瓶颈类型。

### 核心命令
```bash
vmstat [采样间隔] [采样次数]

# 实战常用
vmstat 1 3        # 每秒1次，采样3次（看趋势）
vmstat 1          # 每秒刷新，按Ctrl+C停止（实时监控）

procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
列名	含义	危险阈值	处理建议	
r	运行队列长度（等待CPU的进程数）	CPU核数	CPU瓶颈，需优化进程或扩容	
b	阻塞进程数（等待IO）	0	磁盘IO瓶颈，查硬盘性能	
si/so	每秒从磁盘交换到内存/从内存交换到磁盘	0	内存不足，需加内存或杀进程	
bi/bo	每秒读/写磁盘块数	突增异常	可能有大量日志写入或备份任务	
wa	CPU等待IO完成的时间占比	20%	磁盘太慢，CPU在干等，考虑SSD
# 实战排查流程
# 用户说"服务器好卡"
vmstat 1 5

# 分析路径：
r高 → CPU忙 → 用top/ps找耗CPU进程
b高/wa高 → 磁盘卡 → 用iostat定位具体硬盘
si/so高 → 内存爆 → 用free -m确认，考虑加内存

```

## 2. iostat - 磁盘IO专家（IO Statistics）
### 作用
   精确定位磁盘性能问题，看哪个硬盘在拖后腿（比vmstat更细）。
### 核心命令
基础版（看基本读写）
iostat 1 3

运维必用（看扩展指标，定位瓶颈）
iostat -x 1 3

只看特定硬盘（如sda）
iostat -x sda 1 3

### 输出字段详解（重点看-x扩展模式）
Device    r/s    w/s    rkB/s    wkB/s  await  %util
sda      2.00   5.00    32.00    80.00   4.50  12.00
### 表格
Device    r/s    w/s    rkB/s    wkB/s  await  %util
sda      2.00   5.00    32.00    80.00   4.50  12.00
### 实战场景
数据库查询变慢，怀疑磁盘问题
iostat -x 1 5

看到sdb的%util=99%，await=120ms
结论：sdb硬盘扛不住了，可能是：
1. 数据盘没做RAID  2. 日志和数据同盘  3. 硬盘老化

## 3. sar - 历史数据侦探（System Activity Reporter）
### 作用
   查看过去的系统状态（其他命令只能看现在）。分析已发生的性能故障（如昨晚半夜服务器卡顿）。
### 安装（最小化系统需手动装）
yum install sysstat -y
systemctl enable sysstat   # 开机自启收集

### 核心命令速查
#### CPU历史（最常用）
sar -u 1 5              # 实时看CPU（每秒1次，5次）
sar -u -f /var/log/sa/sa24   # 看24号的历史数据
%user: 用户进程占用

%system: 系统内核占用

%iowait: 重点看，CPU等磁盘的时间，>30%说明磁盘拖后腿

### 磁盘IO历史
sar -d 1 3              # 看所有磁盘
sar -d -p 1 3           # -p显示友好设备名（如sda而非dev8-0）
* tps: 每秒传输次数（Transactions Per Second）
### 内存历史
kbmemfree: 空闲内存（KB）

kbmemused: 已用内存（含缓存）

%memused: 使用百分比，>90%危险
### 网络流量（扩展）
sar -n DEV 1 3          # 看网卡流量（rxkB/s收，txkB/s发）
### 历史数据文件位置

路径:  /var/log/sa/

sa24: 24号的二进制原始数据（用sar -f读取）

sar24: 24号的文本报告（cat直接看）
### 实战场景：复盘昨晚故障
```bash
# 老板问：昨晚3点为什么报警？
# 看23号（假设昨晚是23号）凌晨3点数据

sar -u -f /var/log/sa/sa23 | grep "03:00"   # 看CPU
sar -d -f /var/log/sa/sa23 | grep "03:00"   # 看磁盘

# 如果看到03:00时%iowait=95%，同时磁盘tps飙升
# 结论：凌晨3点有定时任务（如备份）导致IO阻塞
```

# 📋 运维排查速查表（建议背诵）
```bash
现象	先用	再看	确认
服务器现在很卡	`vmstat 1 3`	`iostat -x 1 3`	`top`找具体进程
磁盘报警	`iostat -x 1 3`	`%util`是否>80%	`df -h`看空间
昨晚/上周卡过	`sar -f /var/log/sa/saXX`	对应时间点%iowait	查cron定时任务
内存不足怀疑	`vmstat 1 3`看si/so	`sar -r`看历史	`free -m`确认
```
# ⚠️ 常见面试题
```bash
1.
vmstat中r和b的区别？

r：等CPU的进程（CPU瓶颈）

b：等IO的进程（磁盘瓶颈）
2.
iostat -x中%util达到100%意味着什么？

磁盘饱和，请求队列堆积，必须扩容或优化
3.
如何排查昨晚2点的性能问题？

用 sar -u -f /var/log/sa/sa[日期] 查看历史CPU数据，重点看%iowait
4.
wa（wait）很高但iostat看磁盘不忙？

可能是网络IO卡（NFS挂载卡了），用 vmstat 结合 nfsstat 查
```

