【091】Shell编程快速入门
1. 脚本格式
   #!/bin/bash
   #注释
   echo "hello world"

2. 执行方式
    - 给权限执行：chmod u+x test.sh  →  ./test.sh
    - 直接解释器执行：sh/bash test.sh（不需要执行权限）
    - 绝对路径执行：/root/shell/test.sh

3. 注意事项
    - 必须以#!/bin/bash开头（指定解释器）
    - 要有执行权限才能./执行，或者直接用bash命令调用

【092】Shell变量
1. 定义变量
   变量名=值              # 注意：=两边不能有空格！
   A=100
   name="zhangsan"       # 字符串可以不加引号，有空格必须加

2. 变量规则
    - 可由字母、数字、下划线组成，不能以数字开头
    - 等号两侧不能有空格
    - 变量名一般大写（规范）

3. 撤销变量
   unset A               # 删除变量A

4. 静态变量（只读）
   readonly A=100        # 不能unset，不能修改，生命周期随shell结束

5. 命令结果赋值给变量
   A=`date`              # 反引号（Tab键上面）
   A=$(date)             # 等价写法，推荐这种

6. 输出变量
   echo "A=$A"  或 echo "A=${A}"

【093】设置环境变量（重点）
1. 基本语法
   export 变量名=变量值      # 将shell变量输出为环境变量/全局变量
   source 配置文件           # 让修改后的配置立即生效
   echo $变量名              # 查询环境变量的值

2. 实战步骤（以TOMCAT_HOME为例）
   ① 在/etc/profile中定义
   export TOMCAT_HOME=/opt/tomcat

   ② 立即生效
   source /etc/profile

   ③ 查看
   echo $TOMCAT_HOME

   ④ 在另一个shell脚本中使用（子shell也能访问）

3. 原理图解（93集那个图）
   /etc/profile 是"爸爸"
   ├── a.sh（子shell）能访问
   └── b.sh（子shell）能访问
   所以环境变量是全局的，所有shell子进程都能用

4. 常见环境变量
   $PATH        # 命令搜索路径
   $HOME        # 用户家目录
   $SHELL       # 当前shell类型
   $PWD         # 当前目录

5. 注意事项
    - 修改/etc/profile后必须source，否则只对新打开的终端生效
    - export就是把局部变量变成"传给子进程"的变量