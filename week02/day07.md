# 3月19日 自我测试错误总结
## 关键问题诊断
# 1. 命令格式不严谨

ls-lt  → 报错（缺少空格）

chmod 640  → 忘写文件名

根因：对Linux"命令+参数+对象"的三段式结构理解不深
# 2. 参数记忆混淆

mkdir -p （多级目录）记成 -r

tar 打包和解压参数混用， -C 指定目录完全没概念
# 3. Shell脚本空白

运维面试必问"写过什么脚本"，现在答不出

if判断、变量引用、 $1 参数传递完全不会


## 手敲10遍：
tar -czvf 包名 文件
tar -xzvf 包名 -C 目录/

## 背诵if语句模板
#!/bin/bash
if [ -f "$1" ]; then
echo "File found: $1"
else
echo "Error: $1 not found"
fi

## 题目1：查看Java进程，排除grep本身，只显示pid和命令名
### 方案1：awk提取（更灵活，运维常用）
ps aux | grep java | grep -v grep | awk '{print $2 "\t" $11}'

解释：
awk '{print $2}'  →  打印第2列（PID）
$11               →  第11列（命令名/路径）
\t                →  制表符对齐，美观
### 方案2：ps自定义输出（更专业，推荐）
ps -eo pid,cmd | grep java | grep -v grep

#解释：
#-e                →  显示所有进程
#-o pid,cmd        →  只输出PID和Command两列（自定义输出）
实战答方案二更专业

## 题目2： 找出/var/log下7天前的.log文件，打包到/backup/old_logs.tar.gz
障碍： 没学过 -mtime ，不知道find怎么和tar配合
详细拆解：
分步理解：
第1步：找出7天前的文件
find /var/log -name "*.log" -mtime +7

#参数详解：
-name "*.log"     →  文件名匹配，*是通配符
-mtime +7         →  修改时间大于7天（+7：7天前，-7：7天内）
-mtime -1         →  1天内修改过的（昨天到今天）

第2步：打包（关键难点：如何把find的结果传给tar）
find /var/log -name "*.log" -mtime +7 -exec tar -czf /backup/old_logs.tar.gz {} +

参数详解：
-exec ... {} +    →  对找到的每个文件执行...命令
{}                →  占位符，代表find找到的每一个文件
+                 →  把所有文件一次性传给tar（效率高）
-czf              →  c=create创建, z=gzip压缩, f=file指定文件名
 ### 替代方案 （用xargs，更高级）
find /var/log -name "*.log" -mtime +7 | xargs tar -czf /backup/old_logs.tar.gz
## 题目3：tar解压，（致命错误：参数位置）
我的错误：  tar -xzvf /tmp/extract /backup/old_logs.tar.gz
错误本质： 你把压缩包路径和目录路径写反了，且没用 -C
### 正确逻辑：
语法结构：tar 动作 压缩包 目标位置
tar -xzvf /backup/old_logs.tar.gz -C /tmp/extract/

参数详解：
-x                →  extract解压（与-c创建对应）
-z                →  用gzip解压（看文件后缀.tar.gz）
-v                →  verbose显示过程（可选）
-f                →  file指定文件（必须放最后或紧跟文件名）
-C /tmp/extract/  →  Change directory，先切换到这个目录再解压
### 记忆口诀：
-C 像"搬到"（Change to），搬家时先拿东西（压缩包），再搬到新家（-C目录）

### 常见组合矩阵：
场景	               命令	                               记忆
压缩目录	    `tar -czvf 包名.tar.gz 目录名/`	        create 创建
解压到当前	`tar -xzvf 包名.tar.gz`	                extract 解压
解压到指定	`tar -xzvf 包名.tar.gz -C 目标目录/`  	Change 切换目录
查看内容	    `tar -tzvf 包名.tar.gz`	                table of contents 列表


##  题目3：shell脚本
从零开始：
### 1、脚本骨架（固定格式，必须背熟）
#!/bin/bash
上面这行叫shebang，告诉系统用bash解释器，必须第一行
脚本内容从这里开始
### 2、变量与参数（理解$1）
#!/bin/bash

$1 代表脚本的第一个参数
比如你运行：./test.sh /etc/passwd
那么 $1 就等于 /etc/passwd
echo "你输入的文件是：$1"
### 3、if判断结构（重点）
#!/bin/bash

结构模板：
if [ 条件 ]; then
   条件成立执行的代码
else
    条件不成立执行的代码
 fi

实际应用（判断文件是否存在）：
if [ -f "$1" ]; then
 -f = file，判断是否是普通文件
 "$1" 加引号是防止文件名有空格出错

    SIZE=$(du -h "$1" | cut -f1)
    # du -h = disk usage human readable，查看文件大小
    # cut -f1 = 截取第一列（只保留大小数字，去掉文件名）
    
    echo "File found: $1, Size: $SIZE"
else
echo "Error: $1 not found"
fi
### 关键语法点：

[ -f "$1" ] ：中括号两边必须有空格， $1 必须加引号

then ：可以跟在同一行（ ; then ），也可以换行

fi ：if倒着写，表示结束（配对用）
### 4、执行方法
第1步：创建文件
vim check.sh

第2步：粘贴上面的代码，保存

第3步：给执行权限
chmod +x check.sh

第4步：运行（后面跟你要检查的文件）
./check.sh /etc/passwd
./check.sh /tmp/notexist.txt

3月20日必须能独立写出这个脚本，背也要背下来

## 场景：你在公司，只有VS Code，不能创建.sh文件，但想测试一段脚本逻辑
### 解决方案：

#### 方法1：bash -c（把字符串当脚本执行）
bash -c 'if [ -f /etc/passwd ]; then echo "存在"; else echo "不存在"; fi'
注意：外层用单引号，内层用双引号，防止变量提前解析

#### 方法2：管道传给bash
echo 'echo "Hello"; ls -l /tmp' | bash
#### 方法3：Here Document（多行脚本）
bash << 'EOF'
#!/bin/bash
echo "Line 1"
echo "Line 2"
ls /
EOF
注意最后的EOF必须顶格写，前面不能有空格



