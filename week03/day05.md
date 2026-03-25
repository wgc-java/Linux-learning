rpm vs yum
# 核心区别
维度 	    rpm	yum
本质	        本地包管理器（红帽系）	前端工具，底层调用rpm
依赖处理     	不解决依赖（装A需要B，你得先手动装B）	自动解决依赖（一键搞定所有依赖包）
使用场景	    查询已装软件、安装本地rpm包	在线安装、卸载、更新（日常使用首选）
命令复杂度	参数多，难记	简单，人性化
## 面试话术
"rpm是底层包管理器，主要用于查询已安装软件信息；yum是上层工具，优势是自动解决依赖关系。实际运维中，在线安装用yum，本地安装用rpm。"
# rpm命令（重点在查询，安装为辅）
## 1、查询类
```bash
# 查某个软件是否已安装（最常用）
rpm -qa | grep nginx

# 查软件详细信息（版本、释放时间等）
rpm -qi nginx

# 查软件包安装了哪些文件（路径）
rpm -ql nginx

# 查某个文件属于哪个软件包（反向查询）
rpm -qf /usr/bin/nginx
```
### 记忆口诀：q-query查询，a-all所有，i-info信息，l-list文件列表，f-file文件反查

## 2、安装/卸载（本地包才使用）
```bash
# 安装本地rpm包（不解决依赖，很少用）
rpm -ivh xxx.rpm
# i-install, v-verbose显示过程, h-hash显示进度条

# 卸载（慎用，不检查依赖）
rpm -e nginx
```
## 面试坑点：
问："rpm安装时提示缺少依赖怎么办？"
答："用 rpm -ivh --nodeps 强制安装（不推荐），或者改用yum自动解决依赖。"
#  yum 命令（日常使用核心）
## 1. 安装/卸载/更新（最常用 trio）
```bash
yum install nginx -y      # 安装（-y自动确认）
yum remove nginx -y       # 卸载（同时删依赖）
yum update -y             # 更新所有包（生产环境慎用！）
yum update nginx -y       # 只更新nginx
```
## 2、查询类
```bash
yum list | grep nginx           # 列出可安装的包
yum info nginx                  # 查看软件包信息
yum search nginx                # 搜索软件包（关键词模糊匹配）
yum provides /usr/bin/nginx     # 查文件属于哪个包（类似rpm -qf，但在线查）
```
## 3、缓存管理
```bash
yum clean all           # 清除所有缓存（解决yum源问题时常用）
yum makecache           # 重新生成缓存（换源后执行）
```
# ️ 面试常问场景题
## Q1：生产环境安装软件用什么？
"优先用yum，因为自动解决依赖。如果内网环境没有外网，就配置本地yum源（createrepo制作），或者提前下载好rpm包及依赖用rpm安装。"
## Q2：yum install和rpm -ivh什么区别？
"yum是在线仓库安装，自动下载并解决依赖；rpm是本地安装，不解决依赖。yum更适合在线环境，rpm适合离线安装特定包。"
## Q3：怎么查看系统装了哪些软件？
"用 rpm -qa 列出所有已安装包，配合grep过滤。或者用 yum list installed 。"
## Q4：卸载软件用什么？
"用 yum remove ，因为它会同时卸载不再需要的依赖包。rpm -e只会卸载单个包，容易留下孤儿依赖。"
# 红线提醒（防坑）
## 1.
yum update -y 别乱用（生产环境大忌）
会更新内核（kernel），可能导致重启后系统起不来
面试时说："生产环境更新会用 yum update --exclude=kernel* ，排除内核更新"
## 2.
rpm -e 慎用 
卸载A可能破坏B的依赖（B依赖A），导致其他软件崩溃
面试时说："卸载前先用 rpm -q --whatrequires A 查一下谁依赖A"
## 3.
本地yum源配置（如果被问内网环境）
知道流程：挂载镜像 →  createrepo 生成索引 → 配置 .repo 文件 →  yum clean all && yum makecache
不用背具体配置，知道概念即可
#  面试话术模板（被问到软件包管理）
"Linux软件包管理主要用rpm和yum。rpm用于查询已安装软件信息和本地包安装，命令如 rpm -qa 查询、 rpm -ql 查文件路径；yum用于在线安装和依赖管理，日常使用 yum install/remove/update 。生产环境注意yum update会升级内核，需要加 --exclude=kernel* 排除。"
现在记住3个命令： rpm -qa | grep xx 、 yum install xx -y 、 yum remove xx -y ，面试够用了。