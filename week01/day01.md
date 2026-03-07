# Day01 - Linux环境搭建与基础命令

**日期**：2025-03-07（周六）  
**时间**：18:00-22:00（4小时）  
**视频**：韩顺平Linux第1-15集（跳过第10集vmtools）

## 今日重点

### 1. 环境搭建（第1-9集）
- **CentOS安装**：最小化安装 vs GNOME桌面（选了桌面版方便入门）
- **网络模式**：NAT模式（共享主机IP），桥接模式（独立IP）
- **分区**：自动分区，/boot 500MB，swap 2GB，剩余给/

### 2. 基础命令（第11-15集）
```bash
# 查看当前位置
pwd                    # Print Working Directory

# 列出文件
ls                     # 简单列出
ls -l                  # 详细列表（权限、大小、时间）
ls -la                 # 显示隐藏文件（.开头的）

# 切换目录
cd /                   # 去根目录
cd ~                   # 回家目录（/root）
cd ..                  # 上级目录
cd /etc/sysconfig      # 绝对路径

# 创建目录
mkdir test             # 创建单个目录
mkdir -p a/b/c         # 递归创建（p=parents）