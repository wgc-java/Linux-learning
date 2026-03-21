重复训练shell语句


# 默写
开始
#!/bin/bash
if [ -f "$1" ]; then
SIZE=$(du -k "$1" | cut -f1)
echo "文件存在: $1 大小: ${SIZE}K"
else
echo "错误: $1 文件不存在"
fi
echo "检查完成"
正确
还要加深肌肉记忆
然后运行shell文件的命令：
确保文件存在：
ls -l /tmp/test.sh
然后给执行权限：
chmod +x /tmp/test.sh
验证权限是否加上：
ls -l /tmp/test.sh
会看到rwxr-xr-x
然后运行脚本（两种方法）
方法A：
/tmp/test.sh /etc/passwd
方法B：
cd /tmp
./test.sh /etc/passwd
### ./表示”当前目录“，不能省略，直接写test.sh 会报错。  /etc/passwd 是传给$1的参数（测试存在的文件）
测试验证（必须做）
测试1：传入存在的文件
bash /tmp/test.sh /etc/passwd
预期输出：
文件存在： /etc/passwd,大小：4K
检查完成
然后传入不存在的文件
bash /tmp/test.sh /tmp/不存在的文件txt
输出。。。。。