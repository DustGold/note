### 初识shell

#### 1、概述

shell是一个命令行解释器，它接收应用程序/用户命令，然后调用操作系统内核。shell还是一个功能强大的编程语言，易编写、易调试、灵活性强

Linux提供的shell解释器：

```shell
[atguigu@centos7_base study]$ cat /etc/shells 
/bin/sh
/bin/bash
/sbin/nologin
/usr/bin/sh
/usr/bin/bash
/usr/sbin/nologin
/bin/tcsh
/bin/csh
```

Centos默认的解析器是bash

```shell
[atguigu@centos7_base study]$ echo $SHELL
/bin/bash
```

bash和sh的关系

```shell
[atguigu@centos7_base study]$ ll /bin/ | grep bash
-rwxr-xr-x. 1 root root     964544 4月  11 2018 bash
lrwxrwxrwx. 1 root root         10 7月  28 00:28 bashbug -> bashbug-64
-rwxr-xr-x. 1 root root       6964 4月  11 2018 bashbug-64
lrwxrwxrwx. 1 root root          4 7月  28 00:28 sh -> bash
```

#### 2、入门

注意：shell脚本后缀为.sh文件，脚本以`#!/bin/bash`开头，目的是为了指定解析器

```shell
#!/bin/bash

echo 'hello word!'
```

运行脚本的方式：

1. 第一种：采用bash或sh+脚本的相对路径或绝对路径（不用赋予脚本+x权限）

   ```powershell
   [atguigu@centos7_base study]$ sh ./helloword.sh 
   hello word!
   [atguigu@centos7_base study]$ sh /home/atguigu/study/helloword.sh 
   hello word!
   [atguigu@centos7_base study]$ bash ./helloword.sh 
   hello word!
   [atguigu@centos7_base study]$ bash /home/atguigu/study/helloword.sh 
   hello word!
   ```

2. 采用输入脚本的绝对路径或相对路径执行脚本（必须具有可执行权限+x）

   ```powershell
   [atguigu@centos7_base study]$ ll
   总用量 4
   -rw-rw-r--. 1 atguigu atguigu 32 8月   6 18:29 helloword.sh
   [atguigu@centos7_base study]$ chmod u+x helloword.sh 
   [atguigu@centos7_base study]$ ll
   总用量 4
   -rwxrw-r--. 1 atguigu atguigu 32 8月   6 18:29 helloword.sh
   [atguigu@centos7_base study]$ ./helloword.sh 
   hello word!
   [atguigu@centos7_base study]$ /home/atguigu/study/helloword.sh hello word!
   ```

### 变量

#### 1、系统变量

`$PATH`、`$HOME`、`$PWD`、`$SHELL`、`$USER`

显示shell中所有变量：set

说明：在终端输入sh进入到shell交互式界面，当然也可以直接在黑屏终端下操作测试，使用exit回退到终端

```shell
[atguigu@centos7_base study]$ echo $PATH
/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/atguigu/.local/bin:/home/atguigu/bin
[atguigu@centos7_base study]$ sh
sh-4.2$ echo $PATH
/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/atguigu/.local/bin:/home/atguigu/bin
sh-4.2$ exit
exit
[atguigu@centos7_base study]$sh
sh-4.2$ set
```

#### 2、自定义变量

定义变量：`变量=值`

撤销变量：`unset 变量`

声明静态变量：`readonly 变量`，静态变量不能unset

定义规则：

1. 变量名称可以由字母、数字和下划线组成，但是不能以数字开头，环境变量名建议大写
2. 等号两侧不能有空格
3. 在bash中，变量默认类型都是字符串类型，无法直接进行数值运算
4. 变量的值如果有空格，需要使用双引号或单引号括起来

单引号与双引号的区别：

1. 以单引号包围变量的值时，单引号里面是什么就输出什么，即使内容中有变量和命令（命令需要反引起来）也会把它们原样输出。这种方式比较适合定义显示纯字符串的情况，即不希望解析变量、命令等的场景
2. 以双引号包围变量的值时，输出时会先解析里面的变量和命令，而不是把双引号中的变量名和命令原样输出。这种方式比较适合字符串中附带有变量和命令并且想将其解析后再输出的变量定义

基本使用：

```sh
[atguigu@centos7_base study]$ sh
sh-4.2$ a = 1
sh: a: 未找到命令
sh-4.2$ a= 1
sh: 1: 未找到命令
sh-4.2$ a =1
sh: a: 未找到命令
sh-4.2$ a=1
sh-4.2$ echo $a
1
sh-4.2$ a=11
sh-4.2$ echo $a
11
sh-4.2$ unset a
sh-4.2$ echo $a

sh-4.2$ readonly b=2
sh-4.2$ echo $b
2
sh-4.2$ b=22
sh: b: 只读变量
sh-4.2$ unset b
sh: unset: b: 无法反设定: 只读 variable
sh-4.2$ c=1+2
sh-4.2$ echo $c
1+2
sh-4.2$ d=sunck is a good man
sh: is: 未找到命令
sh-4.2$ d="sunck is a good man"
sh-4.2$ echo $d
sunck is a good man
sh-4.2$ e='sunck is a good man'
sh-4.2$ echo $e
sunck is a good man
sh-4.2$ f=200
sh-4.2$ echo '变量f的值为：$f'
变量f的值为：$g
sh-4.2$ echo "变量f的值为：$f"
变量f的值为：200
```

将变量提升为全局变量：export

```sh
sh-4.2$ G=100
sh-4.2$ echo $G
100
sh-4.2$ vim helloword.sh 
sh-4.2$ cat helloword.sh 
#!/bin/bash

echo 'hello word!'
echo "变量G的值为：$G"
sh-4.2$ ./helloword.sh 
hello word!
变量G的值为：
sh-4.2$ export G
sh-4.2$ ./helloword.sh 
hello word!
变量G的值为：100
```

#### 3、特殊变量

**$n**

功能描述：n为数字，$0代表该脚本名称，$1-$9代表第一到第九个参数，十以上的参数需要用大括号包含，如${10}

```sh
sh-4.2$ vim parameter1.sh
sh-4.2$ cat parameter1.sh 
#!/bin/bash

echo "命令：$0"
echo "第1个参数：$1"
echo "第2个参数：$2"
echo "第3个参数：$3"
echo "第4个参数：$4"
echo "第5个参数：$5"
echo "第6个参数：$6"
echo "第7个参数：$7"
echo "第8个参数：$8"
echo "第9个参数：$9"
echo "第10个参数：${10}"
sh-4.2$ sh ./parameter1.sh 1 2 3 4 5 6 7 8 9 a
命令：./parameter1.sh
第1个参数：1
第2个参数：2
第3个参数：3
第4个参数：4
第5个参数：5
第6个参数：6
第7个参数：7
第8个参数：8
第9个参数：9
第10个参数：a
```

**$#**

功能描述：获取所有输入参数个数，常用于循环

```sh
sh-4.2$ vim parameter2.sh
sh-4.2$ cat parameter2.sh 
#!/bin/bash

echo "参数个数：$#"
sh-4.2$ sh ./parameter2.sh 
参数个数：0
sh-4.2$ sh ./parameter2.sh 1
参数个数：1
sh-4.2$ sh ./parameter2.sh 1 2 3
参数个数：3
```

**$*与$@**

$* 功能描述：这个变量代表命令行中所有的参数，$*把所有的参数看成一个整体

$@ 功能描述：这个变量也代表命令行中所有的参数，不过$@把每个参数区分对待

说明：如果想让$*和$@ 体现区别必须用双引号括起来，并使用循环变量才能看到效果

```sh
sh-4.2$ vim parameter3.sh
sh-4.2$ cat parameter3.sh 
#!/bin/bash

echo $*
echo $@
sh-4.2$ sh ./parameter3.sh 1 2 3
1 2 3
1 2 3
```

**$?**

功能描述：最后一次执行的命令的返回状态。如果这个变量的值为0，证明上一个命令正确执行；如果这个变量的值为非0（具体是哪个数，由命令自己来决定），则证明上一个命令执行不正确了

```sh
sh-4.2$ sh ./parameter3.sh 1 2 3
1 2 3
1 2 3
sh-4.2$ echo $?
0
```

#### 4、运算符

基本语法：`$((运算式))`或`$[运算式]`

```shell
sh-4.2$ h=$(((2+3)*4))
sh-4.2$ i=$[(2+3)*4]
sh-4.2$ echo $h $i
20 20
```

#### 5、控制台输入

基本语法：read (选项) (参数)

| 选项值 | 说明                         |
| ------ | ---------------------------- |
| -p     | 指定读取值时的提示符         |
| -t     | 指定读取值时等待的时间（秒） |

参数：一般为变量，指定读取值的变量名

```sh
sh-4.2$ vim read.sh
sh-4.2$ cat read.sh 
#!/bin/bash

read -p "请输入一个数字：" num1
echo "你第一次输入的数字为：$num1"

read -t 5 -p "请在5秒内输入一个数字：" num2
echo "你第二次输入的数字为：$num2"
sh-4.2$ sh ./read.sh 
请输入一个数字：1
你第一次输入的数字为：1
请在5秒内输入一个数字：2
你第二次输入的数字为：2
sh-4.2$ sh ./read.sh 
请输入一个数字：1
你第一次输入的数字为：1
请在5秒内输入一个数字：你第二次输入的数字为：
sh-4.2$
```

#### 6、条件判断

**基本语法**：

- `test condition`
- `[ condition ]`（注意condition前后要有空格）

注意：条件非空即为true，例如[ atguigu ]与[ 0 ]返回true，[] 返回false

**常用判断条件**：

- 字符串比较是否相等

  `==`

- 整数比较

  `-lt`：小于（less than）

  `-le`：小于等于（less equal）

  `-eq`：等于（equal）

  `-ne`：不等于（not equal）

  `-gt`：大于（greater than）

  `-ge`：大于等于（greater equal）

- 按照文件权限判断

  `-r`：有读的权限（read）

  `-w`：有写的权限（write）

  `-x`：-x 有执行的权限（execute）

- 按照文件类型判断

  `-f`：文件存在并且是一个常规的文件（file）

  `-d`：文件存在并是一个目录（directory）

  `-e`：文件存在（existence）

- 与或非

  `-a`、`-o`、`!`：在中括号内使用

  `&&`、`||`、`!`：在中括号外使用，计算多个中括号中的条件判断式

实操：

1. 判断两个字符相等

   ```sh
   sh-4.2$ [ "sunck" == "sunck" ]
   sh-4.2$ echo $?
   0
   ```

2. 2是否大于等于1

   ```sh
   sh-4.2$ [ 2 -gt 1 ]
   sh-4.2$ echo $?
   0
   ```

3. helloword.sh是否具有读权限

   ```sh
   sh-4.2$ [ -r ./helloword.sh ]
   sh-4.2$ echo $?
   0
   ```

4. /home/atguigu/study/a.txt文件是否存在

   ```sh
   sh-4.2$ [ -e ./a.txt ]
   sh-4.2$ echo $?
   1
   sh-4.2$ [ -e ./helloword.sh ]
   sh-4.2$ echo $?
   0
   ```

5. helloword.sh是普通文件并且可读

   ```sh
   sh-4.2$ [ -f ./helloword.sh -a -r ./helloword.sh ]
   sh-4.2$ echo $?
   0
   sh-4.2$ [ -d ./helloword.sh -a -r ./helloword.sh ]
   sh-4.2$ echo $?
   1
   ```

   ```sh
   sh-4.2$ [ -d ./helloword.sh ] && [ -r ./helloword.sh ]
   sh-4.2$ echo $?
   1
   sh-4.2$ [ -f ./helloword.sh ] && [ -r ./helloword.sh ]
   sh-4.2$ echo $?
   0
   ```

### 选择判断语句

#### 1、if 语句

```shell
if [ 条件判断式 ]
then
	语句
fi
```

逻辑：当程序执行到if语句时，首先计算”条件判断式“，如果”条件判断式“成立，则执行”语句“，否则结束整个if语句继续向下执行

需求：定义一个变量，如果变量的值大于10则输出“sunck is a good man”

```shell
#!/bin/bash

read -p "输入一个数字：" num
if [ $num -gt 10 ]
then
	echo "sunck is a good man"
fi
```

#### 2、if-else语句

```shell
if [ 条件判断式 ]
then
	语句1
else
	语句2
fi
```

逻辑：当程序执行到if语句时，首先计算”条件判断式“，如果”条件判断式“成立，则执行”语句1“，否则执行“语句2”

需求：定义一个变量，如果变量的值大于10则输出“sunck is a good man”，否则输出“sunck is a nice man”

```shell
#!/bin/bash

read -p "输入一个数字：" num
if [ $num -gt 10 ]
then
	echo "sunck is a good man"
else
	echo "sunck is a nice man"
fi
```

#### 3、if-elif-else语句

```shell
if [ 条件判断式1 ];then
	语句1
elif [ 条件判断式2 ];then
	语句2
elif [ 条件判断式3 ];then
	语句3
……
else
	语句e
fi
```

逻辑：当程序执行到if-elif-else语句时，首先计算“条件判断式1”，如果“条件判断式1”成立则执行“语句1”，否则计算“条件判断式2”。如果“条件判断式2”成立则执行“语句2”，否则计算“条件判断式3”，直到某个“条件判断式”成立为止。如果所有的“条件判断式”都不成立，且有else语句，则执行“语句e”，否则结束整个if-elif-else语句继续向下运行

需求：定义一个age变量，如果age的值小于等于0则输出“age有误”，如果age的值大于0小于等于3则输出“婴儿”，如果age的值大于3小于等于6则输出“幼儿”，如果age的值大于6小于等于12则输出“童年”，如果age的值大于12小于等于18则输出“少年”，如果age的值大于18小于等于30则输出“青年”，如果age的值大于30小于等于40则输出“壮年”，如果age的值大于40小于等于50则输出“中年”，如果age的值大于50小于等于150则输出“老年”，其余则输出“妖怪”

```shell
#!/bin/bash

read -p "输入一个数字：" age
if [ $age -le 0 ];then
	echo "age有误！"
elif [ $age -gt 0 ] && [ $age -le 3 ];then
	echo "婴儿"
elif [ $age -gt 3 ] && [ $age -le 6 ];then
	echo "幼儿"
elif [ $age -gt 6 ] && [ $age -le 12 ];then
	echo "童年"
elif [ $age -gt 12 ] && [ $age -le 18 ];then
	echo "少年"
elif [ $age -gt 18 ] && [ $age -le 30 ];then
	echo "青年"
elif [ $age -gt 30 ] && [ $age -le 40 ];then
	echo "壮年"
elif [ $age -gt 40 ] && [ $age -le 50 ];then
	echo "中年"
elif [ $age -gt 50 ] && [ $age -le 150 ];then
	echo "老年"
else
	echo "妖怪"
fi
```

精髓：每一个else都是对它上面所有表达式的否定

```shell
#!/bin/bash

read -p "输入一个数字：" age
if [ $age -le 0 ];then
	echo "age有误！"
elif [ $age -le 3 ];then
	echo "婴儿"
elif [ $age -le 6 ];then
	echo "幼儿"
elif [ $age -le 12 ];then
	echo "童年"
elif [ $age -le 18 ];then
	echo "少年"
elif [ $age -le 30 ];then
	echo "青年"
elif [ $age -le 40 ];then
	echo "壮年"
elif [ $age -le 50 ];then
	echo "中年"
elif [ $age -le 150 ];then
	echo "老年"
else
	echo "妖怪"
fi
```

#### 4、case语句

```shell
case $变量名 in
"值1")
	语句1
	;;
"值2")
	语句2
	;;
……
*)
	语句*
	;;
esac
```

逻辑：当程序执行到case语句时，匹配“变量”的值，匹配上哪个“值”就执行对应的“语句”，执行完“语句”后结束整个case变量。如果没有匹配的“值”则执行“语句*”

1. case行尾必须为单词“in”，每一个模式匹配必须以右括号“）”结束
2. 双分号“**;;**”表示命令序列结束，相当于java中的break
3. 最后的“*）”表示默认模式，相当于java中的default

需求：执行脚本时输入一个数字，输入数字1则打印“星期1”，输入数字2则打印“星期2”，输入数字3则打印“星期3”，输入数字4则打印“星期4”，输入数字5则打印“星期5”，输入数字6则打印“星期6”，输入数字7则打印“星期7”，其他输出“输入有误”

```shell
#!/bin/bash

read -p "请输入一个数字：" num

case $num in
"1")
	echo "星期1"
	;;
"2")
	echo "星期2"
	;;
"3")
	echo "星期3"
	;;
"4")
	echo "星期4"
	;;
"5")
	echo "星期5"
	;;
"6")
	echo "星期6"
	;;
"7")
	echo "星期7"
	;;
*)
	echo "输入有误"
	;;
esac
```

### 循环语句

#### 1、while循环

```shell
while [ 条件判断式 ]
do
	语句
done
```

逻辑：当程序执行到while语句时，首先计算“条件判断式”的值，如果“条件判断式”不成立则结束整个while语句继续向下执行，如果“条件判断式”成立则执行“语句”，执行完“语句”再去计算“条件判断式”的值。如果“条件判断式”不成立则结束整个while语句继续向下执行，如果“条件判断式”成立则执行“语句”，执行完“语句”再去计算“条件判断式”的值。如此循环往复，直到“条件判断式”不成立才终止while语句。

需求：计算1加到100的和

```shell
#!/bin/bash

ret=0
index=1

while [ $index -le 100 ]
do
	ret=$[$ret+$index]
	index=$[$index+1]
done

echo "1加到100的和为：$ret"
```

#### 2、for语句

```shell
for ((变量初始化;循环控制条件;变量变化))
do
	语句
done
```

逻辑：当程序执行到for语句时，首先初始化一个“变量”的值，再去计算"循环控制条件"，如果“循环控制条件”不成立则结束整个for语句继续向下运行，如果“循环控制条件”成立则执行“语句”，执行完“语句”在执行“变量变化”修改“变量的值”，再去计算“循环控制条件”。如此循环往复，直到“循环控制条件”不成立才停止整个for语句。

需求：计算1加到100的和

```shell
#!/bin/bash

ret=0
for ((i=0;i<=100;i++))
do
	ret=$[$ret+$i]
done
echo "1加到100的和为：$ret"
```

注意：只有在for里可以这么写

#### 3、for-in语句

```shell
for 变量 in 计算值1 计算值2 计算值3 ……
do
	语句
done
```

逻辑：按顺序便利in后面的所有“计算值”

```shell
#!/bin/bash

a=1
b=2

for i in 'good' 'cool' $a $b $[1+2]
do
	echo "i的值为：$i"
done
```

#### 4、$*与$@

$*和$@都表示传递给函数或脚本的所有参数，不被双引号“”包含时，都以$1 $2 …$n的形式输出所有参数。当它们被双引号“”包含时，“$*”会将所有的参数作为一个整体，以“$1 $2 …$n”的形式输出所有参数；“$@”会将各个参数分开，以“$1” “$2”…”$n”的形式输出所有参数

```shell
#!/bin/bash

echo '------------$*--------------'
for i in $*
do
	echo "i的值为：$i"
done

echo '------------$@--------------'
for j in $@
do
	echo "j的值为：$i"
done
```

```sh
sh-4.2$ sh ./test.sh 1 2 3
------------$*--------------
i的值为：1
i的值为：2
i的值为：3
------------$@--------------
j的值为：3
j的值为：3
j的值为：3
```

```sh
#!/bin/bash

echo '------------$*--------------'
for i in "$*"
do
	echo "i的值为：$i"
done

echo '------------$@--------------'
for j in "$@"
do
	echo "j的值为：$i"
done
```

```sh
sh-4.2$ sh ./test.sh 1 2 3
------------$*--------------
i的值为：1 2 3
------------$@--------------
j的值为：1 2 3
j的值为：1 2 3
j的值为：1 2 3
```

#### 5、break与continue

break：调出距离最近的那一层循环

continue：跳过距离最近的那一层循环，继续下一次循环继

```shell
#!/bin/bash


echo "----------------break-------------------"
for ((i=1;i<=10;i++))
do
	if [ $i -eq 5 ]
	then
		break
	fi
	echo "i的值为：$i"
done
echo "----------------continue-------------------"
for ((j=1;j<=10;j++))
do
	if [ $j -eq 5 ]
	then
		continue
	fi
	echo "j的值为：$j"
done
```

```sh
sh-4.2$ sh ./test.sh 
----------------break-------------------
i的值为：1
i的值为：2
i的值为：3
i的值为：4
----------------continue-------------------
j的值为：1
j的值为：2
j的值为：3
j的值为：4
j的值为：6
j的值为：7
j的值为：8
j的值为：9
j的值为：10
```

### 系统函数

#### 1、basename

基本语法：`basename [string/pathname] [suffix]`

功能描述：basename命令会删掉所有的前缀包括最后一个（‘/’）字符，然后将字符串显示出来

选项suffix：如果suffix被指定了，basename会将pathname或string中的suffix去掉

```powershell
[atguigu@centos7_base study]$ basename /a/b/c/d/e.txt
e.txt
[atguigu@centos7_base study]$ basename /a/b/c/d/e.txt txt
e.
[atguigu@centos7_base study]$ basename /a/b/c/d/e.txt .txt
e
```

#### 2、dirname

基本语法：`dirname 文件绝对路径`

功能描述：从给定的包含绝对路径的文件名中去除文件名（非目录的部分），然后返回剩下的路径（目录的部分）

```powershell
[atguigu@centos7_base study]$ dirname /a/b/c/d/e.txt
/a/b/c/d
```

### 自定义函数

#### 1、定义

```shell
[function] funname [()]
{
    action;

    [return int;]
}
```

| 结构       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| function   | 说明定义函数，可以省略                                       |
| funname    | 自定义函数名，遵循标识符规则                                 |
| ()         | 可以省略，和function不能同时省略                             |
| action     | 封装的功能                                                   |
| return int | 函数返回值，只能通过$?系统变量来获取，可以显示添加。如果不添加，将以最后一条命令运行的结果作为返回值。注意return后的数值在[0~255]之间，如果超过255，将返回该值与256的余数。 |
| ;          | 语句结束标志，可以不写                                       |

#### 2、调用

```shell
funname [……]
echo "函数的返回值为：$?"
```

| 结果    | 说明                             |
| ------- | -------------------------------- |
| funname | 要调用的函数名                   |
| ……      | 传递参数，如果无需传参即省略     |
| $?      | 获取函数的返回值                 |
| ""      | 需要使用双引号，可以使用特殊变量 |

注意：要在定义函数后再调用函数

#### 3、无参无返回值的函数

需求：打印“hello word”

```shell
#!/bin/bash

# 定义函数
function func()
{
	echo "hello word"
}

echo "-----函数开始执行------"
# 调用函数
func
# 获取函数的返回值的值
echo "返回值为：$?"
echo "-----函数执行完毕------"
```

#### 4、使用参数

需求：计算两个数据的和

```shell
#!/bin/bash

function func()
{
	# 使用传递过来的参数值
	ret=$[$1+$2]
	echo "$1+$2的结果为：$ret"
}

echo "-----函数开始执行------"
# 调用函数，传递参数
func 3 5
echo "返回值为：$?"
echo "-----函数执行完毕------"
```

#### 5、返回值

说明：return后的数值在[0~255]之间，如果超过255，将返回该值与256的余数

```shell
#!/bin/bash

function func()
{
	ret=$[$1+$2]
	return $ret
}

echo "-----函数开始执行------"
func 3 5
echo "返回值为：$?"
echo "-----函数执行完毕------"
```

- 需求：在执行脚本时传递两个数字求和

  ```shell
  #!/bin/bash
  
  function func()
  {
  	ret=$[$1+$2]
  	return $ret
  }
  
  echo "-----函数开始执行------"
  func $1 $2
  echo "返回值为：$?"
  echo "-----函数执行完毕------"
  ```

- 需求：在执行脚本后从控制台输入两个数字求和

  ```shell
  #!/bin/bash
  
  function func()
  {
  	ret=$[$1+$2]
  	return $ret
  }
  
  echo "-----函数开始执行------"
  read -p '请输入第一个数字：' num1
  read -p '请输入第二个数字：' num2
  func $num1 $num2
  echo "返回值为：$?"
  echo "-----函数执行完毕------"
  ```

- 返回值超过255，但是就想使用真实结果

  ```shell
  #!/bin/bash
  
  function func()
  {
  	ret=$[$1+$2]
  }
  
  echo "-----函数开始执行------"
  read -p '请输入第一个数字：' num1
  read -p '请输入第二个数字：' num2
  func $num1 $num2
  echo "结果为：$ret"
  echo "-----函数执行完毕------"
  ```

### 正则表达式

#### 1、常规匹配

一串不包含特殊字符的正则表达式匹配它自己

```shell
[atguigu@centos7_base study]$ cat test.txt 
sunck is a good man
kaige is a good man
sunyz is a niee man

sunck is a nice man!

kaishen is a cool man!
sunds is a handsome man
[atguigu@centos7_base study]$ cat test.txt | grep sunck
sunck is a good man
sunck is a nice man!
```

查看test.txt文件中包含"sunck"的所有的行，匹配规则就是"sunck"

问题：匹配规则单一

#### 2、匹配单个字符与数字

| 匹配         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| .            | 匹配任意字符                                                 |
| []           | 里面是字符集合，匹配[]里任意一个字符                         |
| [0123456789] | 匹配任意一个数字字符                                         |
| [0-9]        | 匹配任意一个数字字符                                         |
| [a-z]        | 匹配任意一个小写英文字母字符                                 |
| [A-Z]        | 匹配任意一个大写英文字母字符                                 |
| [A-Za-z]     | 匹配任意一个英文字母字符                                     |
| [A-Za-z0-9]  | 匹配任意一个数字或英文字母字符                               |
| [^sunck]     | []里的^称为脱字符，表示非，匹配不在[]内的任意一个字符        |
| \d           | 匹配任意一个数字字符，相当于[0-9]                            |
| \D           | 匹配任意一个非数字字符，相当于`[^0-9]`                       |
| \w           | 匹配字母、下划线、数字中的任意一个字符，相当于[0-9A-Za-z_]   |
| \W           | 匹配非字母、下划线、数字中的任意一个字符，相当于`[^0-9A-Za-z_]` |
| \s           | 匹配空白符(空格、换页、换行、回车、制表)，相当于[ \f\n\r\t]  |
| \S           | 匹配非空白符(空格、换页、换行、回车、制表)，相当于`[^ \f\n\r\t]` |

注意：在使用\w、\W、\s、\S时需要使用引号包裹，最好使用单引号

```shell
echo "sunck.www.wang.wwwqqww1fww\nffwweeeww3ssssww gege" | grep ww.

echo "sunck.www.wang.wwwqqww1fww\nffwweeeww3ssssww gege" | grep ww[^0-9]

echo "sunck.www.wang.wwwqqww1fww\nffwweeeww3ssssww gege" | grep 'ww\w'

echo "sunck.www.wang.wwwqqww1fww\nffwweeeww3ssssww gege" | grep 'ww\s'
```

#### 3、匹配锚字符

| 匹配 | 说明                            |
| ---- | ------------------------------- |
| ^    | 行首匹配，和[]里的^不是一个意思 |
| $    | 行尾匹配                        |

```shell
[atguigu@centos7_base study]$ cat test.txt 
sunck is a good man
kaige is a good man
sunyz is a niee man

sunck is a nice man!

kaishen is a cool man!
sunds is a handsome man
[atguigu@centos7_base study]$ cat test.txt | grep ^sun
sunck is a good man
sunyz is a niee man
sunck is a nice man!
sunds is a handsome man
[atguigu@centos7_base study]$ cat test.txt | grep ^good
[atguigu@centos7_base study]$ cat test.txt | grep man$
sunck is a good man
kaige is a good man
sunyz is a niee man
sunds is a handsome man
[atguigu@centos7_base study]$ cat test.txt | grep ^$


[atguigu@centos7_base study]$
```

#### 4、匹配边界字符

| 匹配 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| \b   | 匹配一个单词的边界，指单词和空格的位置                       |
| \B   | 匹配非单词边界                                               |

```shell
[atguigu@centos7_base study]$ cat test.txt 
sunck is a good man!
ack is a nice ma
sfeck,nckab
aaackge
bbbck geg
[atguigu@centos7_base study]$ cat test.txt | grep 'ck\b'
sunck is a good man!
ack is a nice ma
sfeck,nckab
bbbck geg
[atguigu@centos7_base study]$ cat test.txt | grep 'ck\B'
sfeck,nckab
aaackge
```

#### 5、匹配多个字符

| 匹配   | 说明                                |
| ------ | ----------------------------------- |
| (xyz)  | 匹配括号内的xyz，作为一个整体去匹配 |
| x?     | 匹配0个或者1个x，非贪婪匹配         |
| x*     | 匹配0个或任意多个x                  |
| x+     | 匹配至少一个x                       |
| x{n}   | 确定匹配n个x，n是非负数             |
| x{n,}  | 至少匹配n个x                        |
| x{n,m} | 匹配至少n个最多m个x                 |
| x\|y   | \|表示或的意思，匹配x或y            |

```shell
echo "11111a2222aa3333aaa44444aaaa55555" | grep 'a*'

echo "12340567085465046567" | grep '[0-9]*0'
```

#### 6、转义字符

`\ `表示转义，并不会单独使用。由于所有特殊字符都有其特定匹配模式，当我们想匹配某一特殊字符本身时（例如，我想找出所有包含 '$' 的行），就会碰到困难。此时我们就要将转义字符和特殊字符连用，来表示特殊字符本身

```shell
[atguigu@centos7_base study]$ cat test.txt 
sunck is a go$od man
kaige is a goaod man
sunyz is a nice mano
sunck is a nice man!
kaishen is a co$ol man!
sunds is a handsome man
[atguigu@centos7_base study]$ cat test.txt | grep o$
sunyz is a nice mano
[atguigu@centos7_base study]$ cat test.txt | grep o\$
sunyz is a nice mano
[atguigu@centos7_base study]$ cat test.txt | grep "o\$"
sunyz is a nice mano
[atguigu@centos7_base study]$ cat test.txt | grep 'o\$'
sunck is a go$od man
kaishen is a co$ol man!
[atguigu@centos7_base study]$ cat test.txt | grep 'o$'
sunyz is a nice mano
[atguigu@centos7_base study]$
```

#### 7、常用正则表达式

详见《正则匹配示例.docx》

### shell工具

ETL工程师做的一些数据清洗的工作，我们可以使用cut、awk、sort来对一些基本数据进行处理

#### 1、cut

cut的工作就是“剪”，具体的说就是在文件中负责剪切数据用的。cut 命令从文件的每一行剪切字节、字符和字段并将这些字节、字符和字段输出

格式：cut [选项参数]  filename

说明：默认分隔符是制表符

| 选项参数 | 功能                         |
| -------- | ---------------------------- |
| -f       | 列号，提取第几列             |
| -d       | 分隔符，按照指定分隔符分割列 |
| -c       | 指定具体的字符               |

```
de hua good man
hui mei nice women
xue you cool man
ruo tong good women
dao lang cool man
```

- 切割cut.txt文件第一列

  `cut -d " " -f 1 cut.txt`

- 切割cut.txt文件第二、三列

  `cut -d " " -f 2,3 cut.txt`

- 切割cut.txt文件第二到四列

  `cut -d " " -f 2,3,4 cut.txt`

  `cut -d " " -f 2-4 cut.txt`

- 切割cut.txt文件第二到最后列

  `cut -d " " -f 2- cut.txt`

- 在cut.txt文件中切割出hua

  `cat cut.txt | grep hua | cut -d " " -f 2`

- 选取系统PATH变量值，第二个":"开始后的所有路径

  `echo $PATH | cut -d : -f 3-`

- 切割ifconfig后打印的IP地址

  `ifconfig | grep "netmask" | cut -d i -f 2- | cut -d " " -f 2`

#### 2、awk

作用：一个强大的文本分析工具，把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行分析处理

语法：awk [选项参数] ‘pattern1{action1} pattern2{action2}...’ filename

| 结构     | 说明                                    |
| -------- | --------------------------------------- |
| pattern  | 表示AWK在数据中查找的内容，就是匹配模式 |
| action   | 在找到匹配内容时所执行的一系列命令      |
| filename | 文件路径                                |

| 选项参数 | 功能                 |
| -------- | -------------------- |
| -F       | 指定输入文件折分隔符 |
| -v       | 赋值一个用户定义变量 |

- 搜索passwd文件以root关键字开头的所有行，并输出该行的第7列
  `awk -F : '/^root/{print $7}' passwd.txt`

- 搜索passwd文件以root关键字开头的所有行，并输出该行的第1列和第7列，中间以“，”号分割

  `awk -F : '/^root/{print $1 "," $7}' passwd.txt`

- 只显示/etc/passwd的第一列和第七列，以逗号分割，且在所有行前面添加列名user，shell在最后一行添加"dahaige，/bin/zuishuai"

  `awk -F : 'BEGIN{print "user,shell"}{print $1 "," $7}END{print "sunck,/bin/bash"}' passwd.txt`

- 将passwd文件中的用户id增加数值1并输出

  `awk -v i=1 -F : '{print $3+i "," $4}' passwd.txt`

| awk内置变量 | 说明                                   |
| ----------- | -------------------------------------- |
| FILENAME    | 文件名                                 |
| NR          | 已读的记录数（行数）                   |
| NF          | 浏览记录的域的个数（切割后，列的个数） |

- 统计passwd文件名，每行的行号，每行的列数

  `awk -F : '{print "filename:" FILENAME ",linenumber:" NR ",columns:" NF}' passwd.txt`

- 切割IP

  `ifconfig | grep netmask | awk -F "inet " '{print $2}' | awk -F " " '{print $1}'`

- 查询cut.txt中空行所在的行号

  `awk '/^$/{print NR}' cut.txt`

#### 3、sort

作用：sort命令是在Linux里非常有用，它将文件进行排序，并将排序结果标准输出

格式：sort (选项) (参数)

| 选项 | 说明                     |
| ---- | ------------------------ |
| -n   | 依照数值的大小排序       |
| -r   | 以相反的顺序来排序       |
| -t   | 设置排序时所用的分隔字符 |
| -k   | 指定需要排序的列         |

参数：指定待排序的文件列表

```
[atguigu@centos7_base study]$ cat sort.txt 
bb:40:5.4
bd:20:4.2
xz:50:2.3
cls:10:3.5
ss:30:1.6
```

需求：按照“：”分割后的第三列倒序排序

`sort -t : -nrk 3 sort.txt`

<font color="red">k要放在最后</font>