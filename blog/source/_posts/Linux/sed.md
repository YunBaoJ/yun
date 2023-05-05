---
title: Linux之sed的使用方法
categories:
- linux
tags: 
---
### 行编辑器

> vi/vim称为全文本编辑器，sed和awk称为行编辑器
>
> * 交互式和非交互式
> * 文件操作模式和行操作模式

#### 一、sed

##### 1. sed的基本工作方式：

* 将文件以行为单位读取到内存中
*  使用sed的每个命令对该行进行操作
* 处理完成后输出该行

##### 2. 替换命令 s

```shell
sed 's/old/new/' filepath
sed -e 's/old/new/' -e 's/old/new/' filepath # 多条指令使用参数-e
sed -i 's/old/new' filepath # 将替换的结果写回到原文件使用-i
sed 's/正则表达式/new/' filepath # 使用正则表达式使用
sed -r 's/扩展正则表达式/new/' filepath # 使用扩展正则表达式使用-r
```

案例

```shell
# 准备一个实验文件
head -5 /etc/passwd > afile

sed 's/root/chendong/' afile

sed 's:/:~:' afile # 可以使用其他符号代替/做命令的分隔符

sed -e 's/x/aa/' -e 's/aa/bb/' afile
# 上面命令可以简写为
sed 's/x/aa/;s/aa/bb/' afile

sed -i 's/x/bb/' afile

sed 's/..//' afile #可以使用元字符，不写替换内容即为空
sed 's/s*bin//' afile
sed -r 's/s+bin//' afile
sed 's/^root//' afile
```

##### 3. 替换命令的更多选项

* 全局替换

```shell
sed 's/old/new/g' filepath # 用于替换所有出现的地方

# 案例
sed 's/root/!!!!/g' afile
sed 's/root/!!!!/2' afile # 替换匹配到的第2次
```

* 寻址

```shell
sed '/正则表达式/s/old/new/g' filepath # 匹配了正则表达式的行才执行替换
sed '行号s/old/new/g' filepath # 指定行号
sed '行号1,行号2s/old/new/g' filepath # 指定从行号1到行号2的行

# 案例
sed '1,3s/adm/!!!!/g' afile
sed '1,$s/adm/!!!!/g' afile # 指定从第1行到最后一行
sed '/^root/s/bash/!!!/' afile # 通过正则表达式匹配root开头的行
```

##### 4. 其他命令

* 删除
  * d命令后的其他命令不会被执行
  * 不会更改原文件

```shell
sed '[寻址]d' filepath 

# 案例
sed '/root/d' afile # 匹配到root行都删除
```

* 追加、插入、更改

```shell
sed '/root/a hello' afile # a 在匹配的行的下一行插入指定内容

sed '/root/i hello' afile # i 在匹配的行的上一行插入指定内容

sed '/root/c hello' afile # c 将匹配的行更改为指定内容
```

##### 

#### 二、awk

##### 1. awk流程控制

* 输入数据前 BEGIN{}
* 主输入循环  {}
* 所有文件读取完成 END{}
* 不需要写所有流程

##### 2. awk的字段

* 每行称为awk的记录
* 使用空格、制表符分隔开的单词称作字段
* 可以自己指定分隔的字段

```shell
awk '{print $1, $2, $3}' filepath # $1, $2, $3 ... $n 表示每一个字段 \t

awk -F ':' '{print $1, $2, $3}' filepath # 使用-F选项可以改变字段分隔符
```

案例：

```shell
awk -F "'" '/^menu/{ print $2 }' /boot/grub2/grub.cfg

awk -F "'" '/^menu/{ print x++,$2 }' /boot/grub2/grub.cfg # 输出行号
```

##### 3. awk表达式

* 赋值操作符

```shell
var1 = "name"
var2 = "hello" "world" # 字符串连接
var3 = $1 # 第一个字段
```

* 算数操作符

```shell
+ - * / % ^
```

* 系统变量
  * FS 字段分隔符，OFS 输出字段分隔符
  * RS 记录分隔符
  * NR 行数
  * NF 字段数量，最后一个字段内容可以使用NF取出

```shell
awk 'BEGIN{FS=":"}{print $1}' afile # 等同于 awk -F ":" '{print $1}' afile

awk 'BEGIN{RS=":"}{print $0}' afile

awk '{print NR, $0}' afile 

awk '{print NF}' afile 
```

* 关系操作符

```shell
< > <= >= == != 
# 匹配字符、不匹配字符
~ !~
```

* 布尔操作符

```shell
&& || !
```

##### 4. 判断和循环

* 条件语句

```c
if(表达式) {
  awk语句
}
else {
  awk语句2
}
```

案例：

```shell
# 准备实验文件 scores.txt
user1 80 90 67
user2 60 67 90
user3 82 87 83
user4 100 75 68

awk '{if($2>=80) print $1}' scores.txt
awk '{if($2>=80) {print $1; print $2}}' scores.txt
```

* 循环语句

sum=0 ; for(i=2;i<=NF;i++)  sum+=$i ; print sum

```c
while(表达式) {
  awk语句
}

do {
  awk语句
}while(表达式)
  
for(初始值;循环判断条件;累加) {
  awk语句
}
```

* 同样也支持break，continue

案例：

```shell
awk '{sum=0 ; for(i=2;i<=NF;i++) sum+=$i ; print sum, sum/(NF-1) }' scores.txt
```

##### 5. awk数组

* 数组的定义
  * 下标可以使用字符串
* 数组的遍历

```shell
awk '{sum=0 ; for(i=2;i<=NF;i++) sum+=$i ; avg[$1]=sum/(NF-1) }END{for(user in avg) sum2+=avg[user]; print sum2/NR}' scores.txt
```

sum=0 

for(i=2;i<=NF;i++) 

​      sum+=$i

avg[$1]=sum/(NF-1)



for(user in avg) 

​     sum2+=avg[user]

print sum2/NR
