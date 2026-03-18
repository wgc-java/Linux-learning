# 三月18日 学习状况总结
## 测试错误详细分析
### 致命混淆区
错误命令	错误原因	正确命令	记忆法
`mkdir -r`	把删除参数 `-r` 用在创建上	`mkdir -p`	parents，创建多级目录
`cp 源>目标`	混淆重定向符号 `>` 和路径	`cp 源 目标`	cp 是 copy，直接跟路径，没有 `>`
`chmod o+x`	`o` 是 others，不是 all	`chmod +x` 或 `chmod 755`	不给字母就是所有人，或写 `a+x`
`chmod user:group 文件`	chmod 不能改归属	`chown user:group 文件`	own 是拥有者，change owner
`chgrp sudo devuser`	chgrp 改的是文件的组，不是用户的组	`usermod -aG sudo devuser`	user modify 才是改用户属性，`-aG` 追加到附属组
`-su devuser`	`-` 是 su 的选项，不是前缀	`su - devuser`	`su -` 是登录式切换，带环境变量
### 语法盲区
题号	错误答案	正确答案	要点
7	`ps aux 文件`	`ls -l 文件`	看文件权限用 `ls`，`ps` 是看进程
9	`systemctl sudo...`	`sudo systemctl...`	sudo 永远在最前面
14	不知道 `/etc/group`	`cat /etc/group`	系统用户组信息存在 /etc/group
16	不知道 `top`	`top`	实时交互式看进程，按 CPU 排序
17	`\| -v grep`	`\| grep -v grep`	`-v` 是 grep 的参数，不是独立命令
19	`tar ...(*.log)`	`tar -czvf 包名 路径/*.log`	通配符直接写，不要括号；`-f` 后紧跟包名
20	`find app.log`	`grep "关键词" app.log`	`find` 找文件名，`grep` 找文件内容
## 关键概念对照表
### 1、权限四兄弟
chmod 755 file      ← 改文件的 rwx 权限数字（谁都能读，所有者能写，所有人能执行）
chown user:group file  ← 改文件的所有者和所属组（冒号分隔）
chgrp group file    ← 只改文件的所属组（单改组时用）
usermod -aG group user ← 给"用户"添加附属组（操作对象是人，不是文件！）
### 2、压缩命令区分
gzip file.txt       → 生成 file.txt.gz，原文件消失（只能压单文件）
gunzip file.txt.gz  → 解压回 file.txt

zip -r dir.zip dir/ → 压缩目录，原目录保留（跨平台友好）
unzip dir.zip       → 解压

tar -czvf a.tar.gz *.log  → 打包+压缩（c创建 z压缩 v显示 f文件名）
tar -xzvf a.tar.gz -C /tmp → 解压到指定目录（x提取 C指定路径）
### 3、 文本与进程工具
ps aux | grep nginx      → 查看 nginx 进程（静态快照）
top                      → 实时查看进程（交互式，按 q 退出）
kill -9 PID              → 强制杀死进程（-9 是 SIGKILL）
grep -ni "error" file    → 在文件内容里搜 error（-n 行号 -i 忽略大小写）
find / -name "*.log"     → 在系统中找 .log 文件（搜文件名，不是内容）

