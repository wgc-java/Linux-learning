# Linux监控命令学习笔记
## 1、top实时排序
- 按CPU排序：top进入后按大写P
- 按内存排序：top进入后按大写M
- 按PID排序：top进入后按大写N
## 2、netstat网络排查
- “netstat -an”：显示所有连接（数字形式）
- “netstat -anp”：显示进程名/PID（排查谁占用了端口）
## 3、精确匹配端口（防误伤）
- 错误：“grep 80”会匹配到IP里的80，如192.168.1.80
- 正确：“grep “：80””精确匹配端口
## 4、实战命令
- 查看MYSQL连接：“netstat -an | grep ":3306" | grep ESTABLISHED
- 统计22端口连接数："netstat -an | grep ":22" | wc -l"
 

# grep多条件匹配详解（grep的“或”关系）
## 刚才测试中写的  grep ESTABLISHED + TIME_WAIT  错在：shell里  +  不是"或者"的意思，grep会把它当成要搜索的字符 + 。
 正确写法（两种）：
### 方法1：扩展正则（最常用）
grep -E "条件1|条件2"
-E  = 启用扩展正则
|  = 管道符，在这里表示"或"

例子（查22端口，且状态是ESTABLISHED或TIME_WAIT）：
netstat -an | grep ":22" | grep -E "ESTABLISHED|TIME_WAIT"
### 方法2：多参数写法（不记，了解）
grep -e 条件1 -e 条件2
等价写法： netstat -an | grep ":22" | grep -e ESTABLISHED -e TIME_WAIT
### 常见错误！
grep "ESTABLISHED|TIME_WAIT"    # 错误！不加-E，|被当成普通字符
grep ESTABLISHED || TIME_WAIT   # 错误！这是shell的逻辑或，不是grep的
grep ESTABLISHED + TIME_WAIT    # 错误！+是正则量词，不是"或者"
### 简化思维：如果只是想统计所有22端口连接（不管状态），直接
netstat -an | grep ":22" | wc -l
不需要多条件，因为 grep ":22" 已经把22端口全抓出来了。
只有当要排除某些状态（比如不看LISTEN，只看连接）时才用多条件。