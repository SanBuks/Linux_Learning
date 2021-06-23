# 第十章 BASH
1. Shell
    - Shell 广义上来说是运行相应程序的软件 , 狭义上来说是命令行软件(本文所指)
    - Shell 通过接受用户指令指挥内核,让内核操作硬件完成用户的命令
    - 与图形界面不同,优点是远程管理方便,可以方便编写脚本管理,所有发行版本都所具备
    - 种类有 sh( 被Bash替代), C shell(被TCSH替代), K shell 
    - 所有可用种类的shell文件存放在/bin文件中
    - /etc/passwd 中每行最后会记录每个用户所用的不同权限的shell (login shell/non-login shell)
2. Bash 的功能简介
    - 历史记录,命令补全,命令别名,任务管理,前后台控制,脚本编写,正则表达式
    - ^ u / ^ k 从光标处向前/向后删除命令行
    - $ type [-tpa] command_name 查看命令是否为Bash外置命令
        - -t 查看command类型
        - -p 外部命令时显示路径
        - -a 将PATH中所有命令列出来 后接command可以显示执行顺序
    - 在bash中输入名令后查找执行顺序
        - 直接用绝对路径执行命令
        - alias -> builtin -> $PATH中找到的第一个命令
3. Shell 变量
    1. 设置变量
        - 使用变量  $varname / ${varname} (避免混淆情况)
        - 设置变量名字
            - = 设置 两边没有空格 如 myname=sanbuks
            - 变量名称由字母和数字构成,首字母非数字
            - 系统内定的变量用大写字母 自定义变量用小写字母
        - 设置变量内容
            - 双引号和单引号组成字符串,区别是双引号中$var 显示变量内容,而单引号只显示原本字符
            - \ 将特殊符号 回车,$,空格,单引号等变成一般字符
            - `uname -r` 和 $(uname -r) 引用其他命令的结果 (执行过程中先执行引用的命令,结果作为输入信息) 如$ ls -ld `locate crontab`
            - varname=${varname}xxxx / "varname"xxxx累加内容
        - export varname 将变量设置为环境变量
        - unset varname 取消变量
    2. 环境变量
        - 列出变量
          - $ env 列出shell下的环境变量
          - $ set 列出所有变量
          - $ declare 列出所有变量
          - $ export 列出所有环境变量
        - 一些环境变量的功能
          - HOME 用户根目录
          - PATH 执行文件查找路径
          - RANDOM 设定随机数范围
          - PS1 设定命令提示符 具体符号含义略
          - $ 目前shell的PID
          - ? 上一个命令的返回值 0 成功 非零 失败
          - LANG detemine the native language which can be used by applications to determine the language to use
          - LC_ALL determines the values for all locale categories (LC_* and LANG also need ' $ export ' to set )   
        - $ export varname 将自定义变量设定成环境变量让子进程(比如bash下运行的命令程序)调用 
          - 实际上OS分配内存给shell ,其中的全局变量可让shell子程序使用
          - export 局部变量时 , 局部变量被加载到该内存中
          - 启动子程序,子程序将全局变量加载到自己的内存中
    3. 对于变量的操作
        - $ read [-pt] varname 读取键盘输入作为变量内容
          - -p '...'  显示提示字符
          - -t n 设置n秒之后结束输入
        - $ declare / typeset [-aixrp] varname 声明变量类型和属性
          - -a 设定为数组
          - -i 设定为整型 ('+'取消)
          - -x =export ('+'取消)
          - -r 设定位只读不能修改不能unset
          - -p 显示该变量的信息
    4. 限制程序操作文件系统
        - $ ulimit [SHa...] size 限制shell对系统资源的占用
          - -S 警告设置 -H 严格设置
          - -a 显示限制信息 , 可以从中找到限制选项  
    5. 改变变量的内容
        - $ {var#keyword} 从前删除最短的匹配
        - $ {var##keyword} 从前删除最长的匹配
        - $ {var%keyword} 从后删除最短的匹配
        - $ {var%%keyword}从后删除最长的匹配
        - $ {var/keyword/newword} 从前替换一个匹配
        - $ {var//keyword/newword} 替换所有匹配
    6. 变量赋值时的测试
        - var=$ {str-expr} str 未设置 var=expr 否则 var=$str
        - var=$ {str:-expr} str 未设置或为空串 var=expr 否则 var=$str
        - var=$ {str=expr} str 未设置 var=expr, str=expr 否则 var=$str
        - var=$ {str:=expr} str未设置或为空串 var=expr, str=expr 否则 var=$str
        - var=$ {str?expr} str未设置 expr输出至stderr 否则 var=$str
        - var=$ {str:?expr}str未设置或为空串 expr输出至stderr 否则 var=$str
        - var=$ {str+expr}str未设置var="" 否则var=$str
        - var=$ {str:+expr}str未设置或为空串 var="" 否则var=$str
4. 命令别名与历史命令
    - $ alias another_command="command" 设置命令别名
        - $ alias 列出所有别名
        - $ unalias another_command 删除命令别名
    - $ history [n] [-c] / [-raw] histfiles
        - $ history 显示所有历史记录
        - $ history n 显示第n条命令
        - $ history -c 清除记录
        - -r 读 入histfiles文件 默认文件是~/.bash_history -w 写入 -a 追加
        - 多用户登陆时最后一个注销的history文件才有效
    - $ ! [n] [command] [!]
        - !n 执行第n条命令
        - !command 向上查找开头为command的命令并执行
        - !! 执行上一条命令
5.   终端环境设置
    - $ stty [-a] key_function key 设置或显示按键信息
        - -a 列出所有按键信息
        - $ stty erase ^h 设置ctrl + h 为向后删除
    - $ set [-uvx ...hHmBC略] 设置shell (+取消相关设置) 
        - -u 提示使用未设置变量产生的错误信息 
        - -x 执行之前显示命令内容
        - -v 显示额外信息
    - 常见的按键信息
        - ^ C = 暂停屏幕输出 ^ Q = 恢复屏幕输出
        - ^ D = EOF     ^ M = Enter
        - ^ Z = 暂停命令
6. 通配符与特殊符号
    - 通配符  
      - \* 0到无穷多个字符
      - ? 一个任意字符
      - [ abcd] 一个括号内的字符
      - [0-9] 指定在编码顺序内的所有字符
      - [^abcd] 一个不在括号内的字符  ^反向选择
    - 特殊字符
      - \# 注释符号
      - | 管道命令
      - & 将命令编程后台命令
      - \> , >> 输出定向 替换 , 累加
      - \< , << 输入定向
      - ( ) 子shell 起始结束
      - { } 命令区块组合
7. 数据流重定向与命令表达式
    - 执行命令时文件的数据流作为输入,执行命令后的结果作为输出,一般输出为屏幕 如 $ echo "123" 实质为 "123">> screen
    - 流种类
      - 标准输入 stdin (0)< 或 (0)<<
      - 标准输出 stdout (1)> 或 (1)>> 
      - 标准错误输出 stderr 2> 或 2>>
    - 重定向
      - 建立文件 $ ls -al > ./stdinfile (\> 覆盖 >> 追加)
      - 将错误信息和正常结果分开输出 $ find / -name test >  ./findfile 2> ./testerr_file
      - 将错误信息和正常结果输出到同一个文件中 $ find / -name test > ./findfile 2>&1 / $ find / -name test 2> ./findfile 1>&2
      - 键盘输入 $ cat >  catfile "typein"
      - 文件输入 $ cat > catfile < typeinfile (<<"eof" 以eof作为输入的结束标志)
    - 命令表达式(命令行中的表达式都是左结合)
      - command1 ; command2 顺序执行
      - command1 && command2 与逻辑执行 $?为0 则 command2 执行
      - command1 || command2 或逻辑执行 $?为!0 则command2 执行
8. 管道
    - 管道命令"|"将前一个命令的标准输出作为标准输入给与后一个命令,通过多个"|"链接,将数据流像是在管道中进行流动处理
    - 选取命令:
      - $ cut -d 'divide_char' -f n / -c n1-n2 分割选取
        - 以divide_char 为分割字符 取得第 n 个区段(从1开始) 
        - -c n1-n2 取n1到n2个字符
      - $ grep [-acinv]'keyword' filename 匹配选取
        - -a 将二进制文件以文本方式查找
        - -c 统计次数
        - -i 忽略大小写
        - -n 输出行号
        - -v 反向选择
    - 排序命令:
      - $ sort [-fbMnrutk] file 排序结果
        - -f 忽略大小写
        - -b 忽略前面空格
        - -n 以数字排序
        - -t 'divide_char' -k n 作用同上,以这个区段来进行排序
        - -r 反向选择
        - -M 以月份名字排序
        - -u 忽略重复的结果
      - $ uniq [-ic] 将排序号的结果去重
        - -i 忽略大小写
        - -c 进行计数
      - $ wc [-lwm] 统计 [行 字数 字符]
        - -l 仅列出多少行
        - -w 仅列出多少字数
        - -m 仅列出多少字符
    - 双重定向 $ tee [-a] file 如 cat plan.txt | tee ./test | > test2 将数据列额外备份到了./test中
    - 字符转换命令
      - $ tr [-ds] SET1 (SET2)  删除或替换字符
        - -d 删除匹配的字符 如 $... | tr -d '\r' (\r = ^M) 
        - -s 替换匹配的字符 如 $... | tr -s '[a-z]' '[A-Z]' 
      - $ col -x 将tab变成空格 (^I -> )
      - $expand [-t n] file 将tab间隔变成n个空格
      - $ join [-t 'divide_char'] [-i12] file1 file2 合并有相同数据的两行
        - 以'divide_char'为分隔字符默认为空格,将两个文件内容分割
        - -1 n1 -2 n2 取第一个文件的n1区段和第二个文件的n2区段为参照对象(前提已经排列好),合并相同区段的两行,将相同区段的一列调至最左侧且只保留一列,按文件顺序依次追加剩余内容
        - -i 忽略大小写
      - $paste [-d 'divide_char'] file1 file2 以divide_char为分隔字符将file1和file2每行依文件顺序追加在一起
    - $split [-bl] file PREFIX  将大文件分割成若干个小文件PREFIXaa, PREFIXab...
      - -b n[bkm] 按文件大小划分
      - -l n 按行数划分
      - $ cat PREFIX* >> file 合并文件
    - $ xargs [-0epn] command  读入stdin,以空格回车为分隔符产生多个参数作为输入,多次调用command(可以不是管道命令)
      - -n command每次要用几个参数
      - -p 每次调用command前询问
      - -e 'eof_char' 分析到eof_char时结束
      - -0 将特殊字符还原成一般字符
    - "-" 可以替代 stdin 和 stdout 如 $ tar -cvf - /home | tar -xvf - -C /tmp/homeback
