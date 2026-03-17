# 测试纠错笔记
1、删目录命令
❌ rmdir -R /var/log/myapp    # 错误：rmdir无-R，只能删空目录
✅ rm -rf /var/log/myapp      # 正确：递归强制删除
⚠️ 危险警告：rm -rf / var/log（多了空格会删根目录，系统直接报废）
2、改用户组vs改文件组
❌ chgrp sudo devops          # 错误：这是改"devops文件"的组为sudo
✅ usermod -aG sudo devops   # 正确：把用户devops加入sudo组（课程可能未讲，先死记）
3、压缩命令
tar三模式（背）
压：tar -czvf 目标.tar.gz 源目录    # create + zip + verbose + file
看：tar -tzvf 目标.tar.gz           # table列表查看（不解压）
解：tar -xzvf 目标.tar.gz -C /路径   # extract解压到指定目录（-C可选）
## 记忆：c压、t看、x解，后面都是zvf
zip必加-r
❌ zip project.zip /home/project    # 只压空文件夹，里面文件全漏
✅ zip -rv project.zip /home/project # recursive递归 + verbose显示过程
## 用户与权限（易混点）
命令	作用对象	正确示例	常见错误
`useradd -m`	创建用户	`useradd -m devops`	缺`-m`导致无家目录
`chown`	改文件所有者	`chown root:admin file`	分开写chown+chgrp，应冒号合并
`chmod`	改文件权限	`chmod 754 file`	754算对了（rwx/r-x/r--）
## 进程与定时任务
进程查看
ps aux | grep nginx | grep -v grep   # 排除grep自身干扰
或：pgrep -a nginx                    # 最简洁，只看PID和命令
crontab时间格式
#分 时 日 月 周
30 2 * * * /usr/local/bin/backup.sh   # 每天凌晨2:30执行
## shell脚本语法
错误vs正确写法
❌ #！bin/bash                      ✅ #!/bin/bash
❌ String NAME = “Linux”           ✅ NAME="Linux"
❌ echo “Hello,NAME!Today is (date)” ✅ echo "Hello, $NAME! Today is $(date)"
Shell变量三铁律：
1.
等号两侧无空格： NAME="value" （不是 NAME = "value" ）
2.
用英文符号： ""   ''  ``  ()  不能用中文引号
3.
引用加$：变量前加 $ ，命令替换用 $() 包裹
标准模板：
#!/bin/bash
NAME="Linux"
echo "Hello, $NAME! Today is $(date)"
## 查找与查看（细节纠错）
查看文件尾部
tail -n 10 /etc/passwd    # 最后10行
head -n 5 /etc/passwd     # 前5行
tail -f /var/log/nginx.log # 实时追踪（-f = follow）
find精确查找
❌ find /home/user | grep data      # 匹配路径中包含data的所有内容
✅ find /home/user -name "*data*"   # 只匹配文件名



##最后手打三遍
mkdir test && cd test
touch 1.txt 2.txt
tar -czvf test.tar.gz test/    # 压
tar -tzvf test.tar.gz          # 看
tar -xzvf test.tar.gz -C /tmp  # 解
