# Shell脚本

shell 脚本设计内容：

引号、变量、运算符、各种括号、输出、流程控制、输出输入重定向、sed命令、grep命令、正则

## 变量

变量名：字母、下划线

赋值=前后不能有空格

```
PRICE=10
echo  $PRICE
echo "this price is ${PRICE}rmb" #使用花括号避免和其他文字混淆
```

变量默认都是字符串

```
PRICE=10
PRICE=$PRICE+1
echo $PRICE # 输出101
```

此时需要中括号 或者 bash支持的方式

```
echo $[$PRICE+1]
PRICE=`expr $PRICE + 1` # bash支持，expr表示整数运算
let "PRICE+=1"   # bash支持，let表示数学运算
```



##  三种引号的含义

```
反斜杠（\）：使反斜杠后面的一个变量变为单纯的字符串。
单引号（''）：转义其中所有的变量为单纯的字符串。
双引号（""）：保留其中的变量属性，不进行转义处理。
反引号（``）：把其中的命令执行后返回结果。

echo `pwd` 输出执行命令的结果
PRICE=5  定义变量
echo '$PRICE'   输出当前字符串
echo "$PRICE" 输出变量值
echo "\$$PRICE" 输出$5
```



## 第一个脚本&流程控制语句

**第一个脚本**

编写一个example.sh文件，如下：（第一行表示脚本编译器的位置）

```
#!/bin/bash
pwd
```

然后执行`bash example.sh`或者`./example.sh`，通常后者需要root分配权限

**接受用户的参数**

首先编写example.sh文件

```
#!/bin/bash
echo "当前脚本名称为$0"
echo "总共有$#个参数，分别是$*。"
echo "第1个参数为$1，第3个为$3。"
```

执行命令`bash example.sh one two three`，后面三个表示参数。

**条件测试语句** 

```
语法格式：[ 测试条件 ]，注意前后必须有空格
```

* 文件测试语句；`[ -e example.sh ]` 判断文件是否存在，该语句返回真或假，但是没有输出
* 逻辑测试语句；`[ -e example.sh ] && echo "Exist"` 前面若为真，则输出
* 整数值比较语句；`[ 10 -eq 10 ]` 
* 字符串比较语句。`[ $SHELL = "/bin/bash" ] && echo "true"` 

**if条件测试语句 ** 

编写example.sh文件

```
#!/bin/bash
DIR="./test_hehe"
if [ ! -e $DIR ] 
then
 mkdir -p $DIR
fi
```

**while条件循环** 

编写example.sh文件，猜数字游戏，用read读取用户输入数据

```
while true
do
read -p "请输入数字：" INT
if [ "$INT" -eq "$PRICE" ] ; then
echo "恭喜你答对了！"
exit 0
elif [ "$INT" -gt "$PRICE" ] ; then
echo "太高了！"
else
echo "太低了！"
fi
done
```

**for条件循环**

`&>` 表示输出或错误重定向

`$?` 表示上一条命令的执行结果

`ping -C 3 -i 0.2 -W 3 www.xxx.com`，其中-C表示次数，-i表示间隔，-W表示等待超时时间

`/dev/null` 被称为linux黑洞的文件，接受的数据相当于删除了数据

下面写一个测试主机是否在线的脚本。

先写一个ipadds.txt文件

```
192.168.10.10
192.168.10.11
192.168.10.12
```

然后写一个脚本example.txt，并执行

```
#!/bin/bash
HOSTS=$(cat ./ipadds.txt)
for IP in $HOSTS
do
ping -c 3 $IP &> /dev/null
if [ $? -eq 0 ] ; then
echo "Host $IP is On-line."
else
echo "Host $IP is Off-line."
fi
done
```

**case条件测试语句**

判断输入的是什么类型的数据

```
#!/bin/bash
read -p "请输入一个字符：" KEY
case "$KEY" in
[a-z]|[A-Z])
 echo "字母"
;;
[0-9])
 echo "数字"
;;
*)
 echo "其他"
esac
```



## 数组

```
arr=("aba" "def")
arr[3]="hehe"
echo ${arr[0]} # 输出第一个元素
echo ${arr[@]} # 输出所有元素
echo ${#arr[0]} # 第一个元素长度
echo ${#arr[@]} # 数组长度
```



## 计算数值的四种方式

let几乎支持所有的运算符

```
price=1
let "price+=1"
echo $price
```

(())的使用方法和let完全相同

```
price=1
((price+=1))
echo $price
```

$[]将中括号内的表达式作为数学运算

```
price=1
price=$[$price+1]
echo $price
```

expr

```
price=1
price=`expr $price + 1`
echo $price
```

浮点数计算也有命令，先不介绍

## if的条件

字符串判断

```
[ -n string ] 如果长度不为0为真
[ -z string ] 如果长度为0为真
[ "$a" = "$b" ] 判断两个变量是否相等
[ "$a" != "$b" ] 判断两个变量是否不相等
```

数值判断

```
[ $a -eq 15 ] 相等
[ $a -ne 15 ] 不相等
-gt 大于
-lt 小于
-ge 大于等于
-le 小于等于
```

测试文件状态

```
[ -f "file_name" ] 判断是否是一个文件
[ -r "file_name" ] 判断文件是否可读
-a 文件存在
-d 文件存在并且是目录
-f 文件存在并且是文件
```

逻辑判断

```
[ ! 表达式 ]
[ ! 表达式 -a 表达式 ]
[ ! 表达式 -o 表达式 ]
```

## 函数

1. 函数返回值，可以显示增加return语句，return要返回数值；
2. 返回返回值用，$?来获得；
3. 函数可以加function，也可以不加
4. 函数参数和shell一样。

```
#!/bin/bash

function hello(){
    echo "haha"
    echo "参数：$*"
}
hello "Zhangsan"
echo "hello: $?"
```

# Shell常用的Unix命令

```
wc -l file 计算文件行数
wc -w file 计算文件单词数
wc -c file 计算文件中的字符数
cut -b5-10 file 输出每行第5-10个字符
sort file 对文件中的行进行排序
uniq  删除文本文件中重复出现的行列
 -u 只显示不重复的行
 -d 只显示重复的行
 -c 统计重复/不重复的行的数量
tee outfile 将数据输出到标准输出设备和文件
basename file 文件名（没有路径）
dirname file 文件路径（没有文件名）
find 查找文件
grep 'zhangsan' file 在文件中搜索字符串
sed 对数据查找和替换
awk 把文件分成列
split  对文件分割（分成小份）
```

# 启动脚本

启动

```
#!/bin/sh

PORT=8300
nohup java xxx.jar --port=$PORT > /dev/null 2>&1 &
echo "$!" > pid
```

停止

```
#!/bin/sh

kill -9 `cat pid`
```

