# 模块	关键技能	生产环境关联
权限管理	chmod数字/字母模式、chown更改所有者、chgrp更改所属组	生产服务器文件安全管控
磁盘管理	df -h查看挂载、du -sh查看目录大小、lsblk查看分区	磁盘空间不足报警排查
进程管理	ps aux / top / htop、kill信号机制、&后台运行、nohup挂起	服务宕机排查、资源占用分析
定时任务	crontab -e编辑、时间格式理解、&>/dev/null 2>&1日志丢弃	自动化备份、日志清理脚本


# 🔑 关键概念速记（面试常问）
1.
权限数字计算：r=4, w=2, x=1 →  chmod 755 file  = rwxr-xr-x
2.
进程状态：R(运行), S(睡眠), T(停止), Z(僵尸进程)
3.
crontab格式： 分 时 日 月 周 （0 2 * * * = 每天凌晨2点）
4.
磁盘vs内存：df看文件系统挂载点，free看内存交换分区

##二、复习  查看权限
ls -l 文件或目录

# 输出示例：
-rwxr-xr-- 1 root dev 1234 Mar 11 10:00 script.sh


解读：

第1位  - ：文件类型（-文件，d目录，l链接）

第2-4位  rwx ：所有者权限（读+写+执行）

第5-7位  r-x ：所属组权限（读+执行，无写）

第8-10位  r-- ：其他人权限（仅读）


# 三、修改权限 chmod（重点）
方法1：数字法（面试/生产常用）
公式： chmod 数字 文件
# 常见组合（必须背熟）：
chmod 777 file    # rwxrwxrwx 全开（危险，生产环境慎用）
chmod 755 file    # rwxr-xr-x 所有者全权限，其他人只读+执行（脚本标准权限）
chmod 644 file    # rw-r--r-- 所有者读写，其他人只读（配置文件常用）
chmod 700 file    # rwx------ 只有所有者能操作（密钥文件标准）

# 目录需要加上x权限才能进入：
chmod 755 dir     # 目录标准权限
# 2、字母法
chmod u+x script.sh      # 给所有者添加执行权限
chmod go-w file.txt      # 移除组和其他人的写权限
chmod a+r log.txt        # 给所有人添加读权限
chmod u=rwx,g=rx,o=r file # 分别设置不同权限
# 四、 修改所有者和所有组
1、chown更改所有者（root权限）
# 基本语法：
chown 新所有者 文件

# 示例：
chown wgc file.txt           # 将file.txt所有者改为wgc
chown wgc:wgc file.txt       # 同时更改所有者和所属组为wgc
chown -R wgc dir/            # 递归更改目录及其下所有文件（重要！）
2、chgrp更改所属组（root）权限
# 基本语法：
chgrp 新组名 文件

# 示例：
chgrp developers file.txt    # 将文件所属组改为developers
chgrp -R developers dir/     # 递归更改
# 注意！！普通用户不能chown/chgrp，只能root操作或sudo

# 五、生产环境实战场景
1、网站目录标准权限
 Web服务器目录通常这样设置：
chown -R www-data:www-data /var/www/html    # 改所有者和组为web服务用户
chmod -R 755 /var/www/html                  # 目录755，文件644

 上传目录需要写权限：
chmod 775 /var/www/html/uploads

2、脚本权限问题排查
 执行脚本提示Permission denied？
ls -l script.sh
 如果是：-rw-r--r-- (没有x权限)

chmod u+x script.sh    # 添加执行权限
./script.sh            # 现在可以执行了

3、共享目录设置
让目录下新建的文件自动继承组权限：
chmod 2775 /shared      # 2是SGID位，775是权限
 结果：drwxrwsr-x (s表示SGID生效)

# 六、面试高频考点
Q：chmod 755和644的区别？
A：755用于可执行文件或目录（rwxr-xr-x），644用于普通文本/配置文件（rw-r--r--）。目录必须有x权限才能cd进入。
Q：如何递归修改目录及子文件权限？
A：使用-R参数： chmod -R 755 dirname  或  chown -R user:group dirname
Q：为什么上传的文件显示Permission denied？
A：检查三步：1）文件权限是否有r；2）目录权限是否有x（进入权限）；3）文件所有者是否匹配当前用户。
Q：数字权限777代表什么？为什么不推荐？
A：rwxrwxrwx，所有人有全部权限。生产环境极度危险，任何用户（包括攻击者）都能修改/删除文件。


