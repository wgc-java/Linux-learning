# Linux 学习笔记 - Day 3（3月9日周一）

**学习范围**：第25-30集  
**当前进度**：Linux 30集 / Java 80集  
**环境**：VMware + CentOS 7

---

## 一、用户管理进阶（25集）

### useradd 选项
```bash
# 完整创建用户（带家目录+指定shell）
useradd -m -s /bin/bash username    # -m 自动创建/home目录（CentOS有时需要显式指定）
useradd -d /home/web webuser        # -d 指定自定义家目录
useradd -g developers dev01         # -g 指定主要组（需提前存在）

# 查看创建结果
id username                         # 显示UID、GID、所属组
cat /etc/passwd | grep username     # 查看用户信息记录
# passwd与密码管理
passwd username                     #  root给用户设密码
passwd                              #  普通用户修改自己密码（需输入旧密码）

# 密码文件
cat /etc/shadow                     #  加密后的密码（权限000，只有root能看）
# 切换用户与环境变量
su - username                       #  完全切换（加载用户环境变量，推荐）
su username                         #  半切换（保留当前shell环境，PATH可能不对）
## 二、切换用户与环境变量

# 示例：从root切到普通用户
su - wgc                            #  提示符变 $，家目录自动到 /home/wgc
pwd                                 #  显示 /home/wgc

su wgc                              #  提示符可能还是 #，目录还是 /root（坑！）
##环境变量加载区别（面试点）
su - username    # 加载 /etc/profile + ~/.bash_profile + ~/.bashrc（完整环境）
su username      # 只加载 ~/.bashrc（可能找不到自定义命令）
运维场景：切换用户执行程序必须用  su - ，否则可能报 "command not found"
## 三、用户组管理
groupadd developers                 #  创建组
groupadd -g 1005 testers          #  -g 指定GID（自定义编号）

cat /etc/group                      #  查看所有组
# 格式：组名:密码占位符:GID:成员列表

groupdel developers                 #  删除组（组内不能有用户，需先移出）
# usermod修改用户属性
# 修改用户主组（初始组）
usermod -g developers dev01

# 添加附属组（一个用户可属于多个组）
usermod -G wheel,wgc dev01        #  -G 覆盖附属组（注意会清空原有附加组！）
usermod -aG docker dev01          #  -aG 追加附属组（安全，推荐）

# 修改用户名
usermod -l newname oldname        #  -l 小写L，login name
关键区别：
 
主组（Primary Group）：创建文件时默认归属的组（只能有一个）
 
附属组（Secondary Group）：额外权限，如访问docker socket

##四 chmod修改权限
1、数字法
chmod 755 file.txt        # rwxr-xr-x
chmod 644 file.txt        # rw-r--r--
chmod 700 ~/.ssh          # rwx------（私钥目录标准权限）

# 计算规则：4+2+1
# 7 = 4(r) + 2(w) + 1(x)
# 6 = 4(r) + 2(w) + 0(-)
# 5 = 4(r) + 0(-) + 1(x)
2、符号法（精准调整）
chmod u+x script.sh       # 给属主(user)加执行权限
chmod g-w file.txt        # 给属组(group)去掉写权限
chmod o=r file.txt        # 设置其他人(other)只有读权限
chmod a+x script.sh       # 给所有人(all)加执行权限

# 组合
chmod u=rwx,g=rx,o-rwx file.txt    # 属主全权限，组读执行，其他无权限
递归修改
chmod -R 755 /var/www/html/        # -R 大写，目录及子文件全部修改

# 危险！不要对系统目录乱用-R
chmod -R 777 /                      # 作死行为，系统会崩溃
##运维标准权限：
 
脚本文件： 755 （可执行）
 
配置文件： 644 （只读，只有owner可写）
 
日志目录： 755  或  775 
 
私钥文件： 600 （如  ~/.ssh/id_rsa ）
##五、chown修改属主/属组
基础用法
# 修改属主
chown wgc file.txt                  # 把文件给wgc用户

# 修改属组
chown :developers file.txt          # 只改组（注意冒号）
chgrp developers file.txt           # 同上，chgrp专门改组

# 同时修改属主和属组（常用）
chown wgc:developers file.txt       # 主人:wgc，组:developers
chown wgc.wgc file.txt              # 点号也能用，但不推荐（容易和文件名混淆）
递归修改
chown -R nginx:nginx /var/www/html/    # 把网站目录改成nginx用户所有
运维场景：Web服务器报错 "Permission denied"，经常是上传文件后属主是root，
而服务以nginx用户运行，需  chown nginx:nginx  修正。
##六、文件查找命令入门
#which/whereis
which python3                       # 找命令位置（看$PATH）
which -a python3                    # 显示所有路径（可能有多个版本）

whereis python3                     # 找命令+源码+man手册
whereis -b ls                       # 只找二进制文件
#locate快速查找
locate nginx.conf                   # 从数据库查（秒出结果）
locate -i mysql                     # 忽略大小写
sudo updatedb                       # 更新数据库（每天自动更新，手动执行可搜新文件）

# 原理：查 /var/lib/mlocate/mlocate.db，非实时但极快
find实时查找
find / -name "nginx.conf"           # 全局查找（慢，遍历磁盘）
find /etc -name "*.conf"            # 限定目录范围
find /var/log -size +100M           # 按大小找（清理大日志用）
