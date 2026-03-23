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