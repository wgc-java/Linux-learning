# Linux基础测试题详解 - 2026年3月26日
## 测试范围：韩顺平Linux教程（除66-69集NAT外全覆蓋）

---

## 模块一：进程与性能监控（第78-85集）

### 第1题 【top命令交互操作】
在top命令运行界面中，如何按**内存使用率**从高到低排序？  
A. 按 `P` 键  
B. 按 `M` 键  
C. 按 `N` 键  
D. 按 `T` 键

**正确答案：B**

**详细解析：**
- **A. P键**：按**CPU使用率**排序（Process），这是默认排序方式
- **B. M键**：按**内存使用率**排序（Memory），正确选项
- **C. N键**：按**进程PID号**排序（PID Number）
- **D. T键**：按**运行时间**排序（Time）

**记忆技巧**：P=CPU(Process)，M=Memory(M)，按首字母对应。

---

### 第2题 【netstat参数组合】
查看当前所有网络连接，并以数字形式显示地址和端口，同时显示对应进程名，应使用哪个命令？  
A. `netstat -an`  
B. `netstat -tunlp`  
C. `netstat -anp`  
D. `netstat -tuln`

**正确答案：B**

**详细解析：**
- **A. netstat -an**：仅显示所有连接并以数字形式显示，但**不显示进程名**（缺少-p参数）
- **B. netstat -tunlp**：**正确组合**
- `-t`：TCP协议
- `-u`：UDP协议
- `-n`：数字形式显示IP和端口（不做DNS解析，加快显示速度）
- `-l`：仅显示监听（Listening）状态的端口
- `-p`：显示进程名/PID（需要root权限才能看到所有进程）
- **C. netstat -anp**：显示了所有连接和进程，但**包含非监听状态**（如ESTABLISHED），且未限定TCP/UDP
- **D. netstat -tuln**：虽然显示了监听的TCP/UDP端口，但**缺少-p参数，看不到进程名**

**应用场景**：查看"哪个程序占用了80端口"必须用`-p`参数。

---

### 第3题 【网络连接状态识别】
执行`netstat -an`后看到大量`TIME_WAIT`状态的连接，这表示什么？  
A. 连接已建立，正在传输数据  
B. 连接刚断开，进入超时等待状态  
C. 正在尝试建立连接（三次握手中）  
D. 端口被占用，连接被拒绝

**正确答案：B**

**详细解析：**
- **A. ESTABLISHED**：才是连接已建立状态
- **B. TIME_WAIT**：**正确**。主动关闭方在发送最后一个ACK后进入的状态，等待2MSL（最大报文段生存时间）确保对方收到ACK，**防止旧连接数据包干扰新连接**
- **C. SYN_SENT**：正在发送SYN包尝试建立连接
- **D. CLOSE_WAIT或REFUSED**：端口占用或拒绝连接

**运维意义**：TIME_WAIT过多会占用端口资源，高并发场景可能需要调整`tcp_tw_reuse`参数。

---

### 第4题 【netstat过滤技巧】
如何查看当前所有ESTABLISHED（已建立）状态的SSH连接？  
A. `netstat -an | grep SSH | grep ESTABLISHED`  
B. `netstat -anp | grep 22`  
C. `netstat -tunlp | grep sshd`  
D. `ss -t state established '( dport = :22 or sport = :22 )'`

**正确答案：A**

**详细解析：**
- **A**：**正确但需注意大小写**。`netstat -an`输出中协议名是大写（TCP/UDP），但进程名可能是小写。更准确的是`netstat -an | grep :22 | grep ESTABLISHED`或`netstat -an | grep -i ssh`
- **B**：能看到22端口，但**包含LISTEN状态**（监听），不限于ESTABLISHED
- **C**：仅显示监听状态的sshd，**不显示已建立的连接**
- **D**：`ss`命令是新一代工具，语法更强大，但题目问的是netstat用法

**纠正**：在实际生产中，更准确的命令是`netstat -anpt | grep sshd | grep EST`或`ss -p | grep ssh`。

---

### 第5题 【vmstat系统监控】
`vmstat 2 5`命令的输出中，**procs**列的`r`和`b`分别表示什么？  
A. r=运行进程数，b=阻塞进程数  
B. r=读IO等待，b=写IO等待  
C. r=内存使用率，b=交换分区使用率  
D. r=僵尸进程数，b=孤儿进程数

**正确答案：A**

**详细解析：**
- **r（running）**：**运行队列中的进程数**（正在运行或等待CPU时间片），数值持续大于CPU核数说明CPU瓶颈
- **b（blocked）**：**不可中断睡眠状态的进程数**（通常等待IO，如磁盘读写），数值高说明IO瓶颈
- **其他选项**：IO等待在vmstat的`wa`（wait）列显示；内存在`memory`和`swap`区域显示；僵尸进程是`Z`状态，需用`ps`查看

**使用场景**：`vmstat 2 5`表示每2秒采样一次，共采样5次，用于判断系统是CPU密集型还是IO密集型负载。

---

### 第6题 【iostat磁盘监控】
`iostat -x 1`输出中，**%util**指标接近100%表示什么？  
A. CPU利用率达到100%  
B. 磁盘IO队列过长，存在IO瓶颈  
C. 内存交换频繁  
D. 网络带宽占满

**正确答案：B**

**详细解析：**
- **%util**：表示**磁盘设备的利用率**（每秒内设备忙于处理IO请求的时间百分比）
- **接近100%**：说明磁盘几乎一直在处理IO请求，**存在IO瓶颈**，可能导致应用响应慢
- **A**：CPU利用率看`top`的`id`（空闲）列或`us`（用户态）+`sy`（系统态）
- **C**：内存交换看`vmstat`的`si`（换入）和`so`（换出）列，或`free`命令
- **D**：网络带宽看`iftop`、`nload`或`sar -n DEV`

**注意**：%util高不一定代表磁盘到了物理极限，SSD和多盘RAID可以承受更高的util。

---

### 第7题 【sar命令综合】
使用`sar -u 1 3`命令主要查看什么信息？  
A. CPU利用率（每1秒采样，共3次）  
B. 内存使用情况  
C. 磁盘IO统计  
D. 网络吞吐量

**正确答案：A**

**详细解析：**
- **sar（System Activity Reporter）**：系统活动报告，可以查看历史或实时性能数据
- **-u**：**CPU利用率**（user/system/iowait/idle等）
- **1 3**：采样间隔1秒，采样3次
- **其他选项**：
- 内存：`-r`（内存使用率）或`-S`（交换分区）
- 磁盘IO：`-b`（IO传输率）或`-d`（块设备详细）
- 网络：`-n DEV`（网卡流量）

**优势**：sar可以保存历史数据（在/var/log/sa/），能查看昨天或上周的性能数据，这是top/vmstat做不到的。

---

### 第8题 【lsof实用场景】
查找哪个进程占用了`/var/log/nginx/access.log`文件，应使用？  
A. `ps aux | grep nginx`  
B. `lsof /var/log/nginx/access.log`  
C. `fuser -v /var/log/nginx/access.log`  
D. `file /var/log/nginx/access.log`

**正确答案：B**

**详细解析：**
- **B. lsof**：**List Open Files**，列出打开的文件及占用进程。Linux中一切皆文件，网络端口也是文件
- 示例输出：`nginx 1234 root 3w REG 253,0 12345 /var/log/nginx/access.log`
- 其中`3w`表示文件描述符3，w表示写模式打开
- **A. ps aux**：只能看到进程名，**无法精确定位到具体文件**
- **C. fuser**：也能查看，但参数较复杂，不如lsof直观，且默认可能未安装
- **D. file**：仅查看文件类型，不看进程占用

**扩展应用**：`lsof -i :80`查看占用80端口的进程；`lsof +D /var/log`查看所有打开/var/log下文件的进程。

---

## 模块二：磁盘与文件系统管理

### 第9题 【lsblk与分区查看】
如何查看系统中所有磁盘的分区结构和挂载点？  
A. `df -h`  
B. `fdisk -l`  
C. `lsblk`  
D. `mount`

**正确答案：C**

**详细解析：**
- **A. df -h**：查看**已挂载**文件系统的磁盘使用率，**看不到未挂载的分区和磁盘**
- **B. fdisk -l**：查看分区表详细信息（包括未挂载），但**需要root权限**，且输出冗长
- **C. lsblk**：**List Block Devices**，树形结构显示**块设备（磁盘/分区）及其挂载点**，最直观
- 输出示例：`sda ├─sda1 /boot └─sda2 /`
- **D. mount**：查看当前已挂载的文件系统，但**格式混乱，不适合快速查看结构**

**对比**：lsblk不显示使用量（用df），也不显示分区大小细节（用fdisk），但**结构最清晰**。

---
### 第10题 【查找大文件】
在`/var/log`目录下查找**大于100MB**的文件，应使用？  
A. `du -sh /var/log/*`  
B. `find /var/log -size +100M -type f`  
C. `ls -lh /var/log | grep M`  
D. `df -h /var/log`

**正确答案：B**

**详细解析：**
- **A. du -sh**：查看目录总大小，但**不会筛选大于100M的**，且*可能匹配目录导致递归统计慢
- **B. find /var/log -size +100M -type f**：**正确**
- `-size +100M`：大于100MB（+表示大于，-表示小于，无符号表示等于）
- `-type f`：仅查找**文件**（File），排除目录
- 可配合`-exec ls -lh {} \;`显示详情
- **C. ls -lh**：只能看到文件列表，**不会递归子目录**，且"M"可能出现在文件名中误匹配
- **D. df -h**：查看**文件系统整体**使用率，不显示单个文件

**生产技巧**：`find / -size +1G -type f 2>/dev/null`查找大于1G的文件，2>/dev/null屏蔽权限错误。

---

### 第11题 【日志清理实战】
`/var/log/app.log`文件已占用10GB空间，但应用正在写入该文件，如何**安全清空**且**不重启应用**？  
A. `rm -rf /var/log/app.log`  
B. `echo "" > /var/log/app.log`  
C. `> /var/log/app.log`  
D. `truncate -s 0 /var/log/app.log`

**正确答案：C 或 D（多选正确）**

**详细解析：**
- **A. rm -rf**：**危险！**虽然能删除文件，但**应用进程仍持有文件句柄**，磁盘空间不会释放（df看还是满的），且应用无法继续写入（可能报错）
- **B. echo "" >**：效果同C，但**多了写入空字符串的操作**，虽然结果一样但不够优雅
- **C. > /var/log/app.log**：**重定向清空**，将空内容写入文件，**立即释放空间**，应用可继续写入
- **D. truncate -s 0**：**专业命令**，将文件大小截断为0字节，**最安全高效**，是运维首选

**原理**：Linux中文件删除是删除目录项（dentry），但文件内容（inode）只有当所有进程关闭文件句柄后才真正释放。

---

## 模块三：定时任务调度（crontab）

### 第12题 【crontab语法基础】
`crontab -e`中，格式`30 2 * * * /backup.sh`表示什么？  
A. 每2小时30分执行一次  
B. 每天凌晨2:30执行  
C. 每月2号30分执行  
D. 每30分钟执行一次

**正确答案：B**

**详细解析：**  
Crontab格式：**分 时 日 月 周**（Minute Hour Day Month Weekday）
- `30`：第30分钟
- `2`：凌晨2点
- `*`：每天
- `*`：每月
- `*`：每周的每一天

**常见错误**：
- 写成`2 30 * * *`变成每天30点2分（无效时间）
- 与`*/30`混淆（每30分钟）

---

### 第13题 【crontab高级语法】
要求**每月1号和15号**的**凌晨2:30**执行备份脚本，正确写法是？  
A. `30 2 1-15 * * /backup.sh`  
B. `30 2 1,15 * * /backup.sh`  
C. `30 2 */15 * * /backup.sh`  
D. `30 2 1|15 * * /backup.sh`

**正确答案：B**

**详细解析：**
- **A. 1-15**：表示**1号到15号每天**执行（连续范围）
- **B. 1,15**：**逗号表示列表**，1号和15号执行，**正确**
- **C. */15**：**步长语法**，表示每15天执行一次（1号、16号、31号）
- **D. 1|15**：管道符在crontab中**无效**，这是shell语法

**扩展**：
- `*/5 * * * *`：每5分钟
- `0 0 * * 1`：每周一凌晨
- `0 0 1 * *`：每月1号凌晨

---

### 第14题 【crontab环境变量】
为什么crontab中执行脚本时经常出现"command not found"错误？  
A. crontab有独立的PATH环境变量，通常只包含/usr/bin:/bin  
B. 脚本权限不足  
C. 必须使用绝对路径写脚本  
D. crontab不支持执行shell脚本

**正确答案：A**

**详细解析：**
- **A**：**根本原因**。crontab的PATH通常只有`/usr/bin:/bin`，而用户安装的软件（如Java、Python、Docker）通常在`/usr/local/bin`或其他路径
- **B**：权限不足会报"Permission denied"，不是"command not found"
- **C**：推荐用绝对路径，但这不是"command not found"的原因
- **D**：crontab完全可以执行脚本

**解决方案**：
1. 脚本开头声明完整PATH：`PATH=/usr/local/bin:/usr/bin:/bin`
2. 命令使用绝对路径：`/usr/local/bin/python3 /path/to/script.py`
3. 在crontab顶部定义变量：`SHELL=/bin/bash; PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin`

---

## 模块四：压缩与归档（tar）

### 第15题 【tar打包与压缩】
将`/etc/nginx`目录打包并用gzip压缩，生成带日期的备份文件`nginx-config-20250326.tar.gz`，正确命令是？  
A. `tar -czvf nginx-config-20250326.tar.gz /etc/nginx`  
B. `tar -xzvf nginx-config-20250326.tar.gz /etc/nginx`  
C. `gzip -r /etc/nginx > nginx-config-20250326.tar.gz`  
D. `zip -r nginx-config-20250326.tar.gz /etc/nginx`

**正确答案：A**

**详细解析：**
- **tar参数**：
- `-c`：**create**创建新包
- `-z`：用**gzip**压缩（生成.tar.gz或.tgz）
- `-v`：**verbose**显示过程
- `-f`：**file**指定文件名（必须放在参数最后或紧接文件名）
- **B. -x**：**extract**解压，不是打包
- **C. gzip**：只能压缩单个文件，**不能打包目录**，且`-r`是递归，不生成tar包
- **D. zip**：虽然能压缩目录，但生成的是.zip格式，**不是tar.gz**

**注意**：tar打包会保留文件权限和软链接，这是zip做不到的，因此Linux备份首选tar。

---

### 第16题 【tar解压与指定目录】
将`nginx-config-20250326.tar.gz`解压到`/tmp/test`目录（该目录已存在），正确命令是？  
A. `tar -xzvf nginx-config-20250326.tar.gz /tmp/test`  
B. `tar -xzvf nginx-config-20250326.tar.gz -C /tmp/test`  
C. `tar -czvf nginx-config-20250326.tar.gz -C /tmp/test`  
D. `cd /tmp/test && tar -xzvf ~/nginx-config-20250326.tar.gz`

**正确答案：B 或 D（多选）**

**详细解析：**
- **A**：**错误！**`/tmp/test`会被当成要解压的文件名（因为tar认为第一个非参数是要解压的包），如果该文件不存在会报错
- **B**：**-C参数**（Change directory），先切换到指定目录再解压，**最标准写法**
- **C**：-c是创建，不是解压
- **D**：先cd到目标目录再解压，**效果等同B**，但B更专业（不需要切换当前shell目录）

**进阶**：
- 只查看内容不解压：`tar -tzvf nginx-config-20250326.tar.gz`（t=list）
- 只解压特定文件：`tar -xzvf nginx-config-20250326.tar.gz etc/nginx/nginx.conf`

---

## 模块五：软件包管理（yum vs rpm）

### 第17题 【yum与rpm核心区别】
关于yum和rpm的描述，正确的是？  
A. rpm可以自动解决软件包依赖关系，yum不行  
B. yum基于rpm，但能自动下载并安装依赖包  
C. yum只能安装，不能卸载软件  
D. rpm是红帽专用，其他Linux发行版不能用

**正确答案：B**

**详细解析：**
- **A**：**完全相反**。rpm是底层工具，**不解决依赖**（安装A时如果依赖B和C，rpm会报错退出）；yum是上层工具，**自动解决依赖**（会一并下载B和C）
- **B**：**正确**。yum（Yellowdog Updater Modified）基于rpm包格式，但连接软件仓库（repository），自动处理依赖关系
- **C**：yum可以卸载：`yum remove 包名` 或 `yum erase 包名`
- **D**：rpm是**Linux Standard Base（LSB）标准**，几乎所有发行版都支持（SUSE、CentOS、Fedora等），只是包管理器不同（Ubuntu用dpkg/apt，但也能转换安装rpm）

**类比**：rpm像手动安装.exe（缺DLL就报错），yum像应用商店（自动下载依赖）。

---

### 第18题 【rpm查询命令】
如何查询系统中是否安装了`nginx`以及安装了哪个版本？  
A. `rpm -qa | grep nginx`  
B. `rpm -qi nginx`  
C. `rpm -ql nginx`  
D. `rpm -qf nginx`

**正确答案：A 或 B（不同场景）**

**详细解析：**
- **A. rpm -qa | grep nginx**：
- `-q`：query查询
- `-a`：all所有已安装的包
- **用于模糊查询**：不确定包名是nginx还是nginx-1.20.1时用
- 输出示例：`nginx-1.20.1-1.el7.x86_64`

- **B. rpm -qi nginx**：
- `-i`：info详细信息
- **用于查看具体包的版本、描述、安装时间、签名等**
- 需要知道准确包名（可以用A查到的全名）

- **C. -ql**：list，列出该包**安装的所有文件列表**（如配置文件、二进制文件位置）
- **D. -qf**：file，查询**某个文件属于哪个包**（如`rpm -qf /usr/bin/nginx`）

**常用组合**：
- `rpm -qc nginx`：只看配置文件（config）
- `rpm -qd nginx`：只看文档（document）
- `rpm -qR nginx`：查看依赖（Require）

---

### 第19题 【yum仓库与缓存】
执行`yum install nginx`时，系统首先会做什么？  
A. 直接从网上下载nginx并安装  
B. 检查本地/var/cache/yum缓存，然后查询配置的repository（仓库）元数据  
C. 检查crontab中是否有相关定时任务  
D. 询问root密码确认权限

**正确答案：B**

**详细解析：**  
**yum工作流程**：
1. **解析依赖**：读取本地已安装包数据库（/var/lib/rpm/）
2. **更新元数据**：检查`/var/cache/yum/`中的仓库缓存是否过期（默认可能几小时到几天）
3. **查询仓库**：连接`/etc/yum.repos.d/`中配置的base/epel等仓库
4. **下载**：找到nginx及其依赖的rpm包，下载到`/var/cache/yum/.../packages/`
5. **安装**：调用rpm命令进行本地安装

- **A**：不会"直接"下载，先查缓存和元数据
- **C**：与crontab无关
- **D**：sudo在命令执行前已验证，yum本身不询问密码

**实用命令**：
- `yum clean all`：清空缓存（解决metadata错误）
- `yum makecache`：立即更新元数据
- `yum repolist`：查看可用仓库列表

---

### 第20题 【综合：文件权限排障】
执行`ls -l /data/file.txt`显示：  
`-rw-r--r-- 1 root developers 1024 Mar 26 10:00 /data/file.txt`  
用户`tom`（属于developers组）无法写入该文件，最可能的原因是？  
A. 文件权限中组权限是r--（只读），需要改为rw-  
B. 目录/data没有写权限  
C. 文件被chattr锁定  
D. 磁盘只读挂载

**正确答案：A**

**详细解析：**  
**权限位分析**：`-rw-r--r--`
- 第1位`-`：普通文件
- 第2-4位`rw-`：属主（root）可读可写
- 第5-7位`r--`：**属组（developers）只读** ← 问题在这里
- 第8-10位`r--`：其他人只读

- **A**：**正确**。组权限只有r（4），没有w（2），需要`chmod 664 file.txt`或`chmod g+w file.txt`
- **B**：如果目录无写权限，报错会是"Permission denied"在创建/删除文件时，但修改已有文件是文件本身的权限控制
- **C. chattr +i`：会导致连root都无法修改，但题目没提这个特殊属性
- **D. 只读挂载**：会导致整个分区无法写入，不只是单个文件

**修改命令**：
- `sudo chmod 664 /data/file.txt`（rw-rw-r--）
- 或 `sudo chown tom:developers /data/file.txt`（改属主为tom）

---

## 易错点总结与记忆口诀

### 1. Crontab五字段顺序
**口诀**：`分 时 日 月 周`（**Minute Hour Day Month Week**）  
**联想**："分时日月周"像"分日月年周"的时间顺序，注意第一个是**分钟**不是秒

### 2. tar参数成对记忆
- **压缩打包**：`c`reate + `z`ip = **-czvf**
- **解压**：e`x`tract + `z`ip = **-xzvf**
- **指定目录**：大写 **-C**（Change directory）

### 3. yum vs rpm选择
- **装软件选yum**（自动解决依赖）
- **查信息用rpm**（查询已安装包详情）
- **离线环境用rpm**（先下载好rpm包再手动安装）

### 4. 性能监控工具分工
- **top**：实时看进程（交互式，按P/M排序）
- **vmstat**：看系统整体（CPU/内存/IO/进程状态）
- **iostat**：专看磁盘IO（%util指标关键）
- **sar**：看历史数据（保存到/var/log/sa/）
- **netstat/ss**：看网络连接（端口占用、TCP状态）

### 5. 日志清理安全操作
**危险**：`rm`正在写入的日志（空间不释放）  
**安全**：`> file` 或 `truncate -s 0 file`（清空内容，释放空间，保持文件句柄）

---

## 附：今日答题情况速查
- **第15题（权限）**：文件所属组权限不足（r--需改为rw-）
- **第16题（磁盘）**：lsblk✅ | 找大文件(find -size)❌ | 清空日志(> file)❌
- **第17题（crontab）**：语法正确`30 2 1,15 * *`，注意环境变量PATH问题
- **第18-19题（tar）**：压缩(-czvf)✅ | 解压到指定目录(-C)✅

**建议加强**：find命令的-size参数、日志清理机制、crontab环境变量调试。