Linux最近学的易忘记的命令
# 网络排查
- netstat -tunlp        # 看监听端口（数字+进程）
- netstat -an | grep :80 # 查80端口
- ss -tunlp             # netstat新版替代

# 定时任务（crontab）
- crontab -e            # 编辑
- crontab -l            # 查看
 
## 语法
分 时 日 月 周 命令
0 2 * * * /backup.sh       # 每天2点
*/5 * * * * /check.sh      # 每5分钟
0 2 1,15 * * /backup.sh    # 每月1号15号
0 2 * * 1 /weekly.sh       # 每周一

# 高频面试命令
```bash
查大文件（>100M）
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null | sort -k5 -rh | head -5

清空日志（不删文件）
> /var/log/big.log

查目录大小
du -sh /var/log
du -h --max-depth=1 /var | sort -rh

实时看日志
tail -100f /var/log/nginx/access.log
```
