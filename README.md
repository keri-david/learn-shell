- [Linux 基础](#linux-基础)
- [正则表达式和linux三剑客](#正则表达式和linux三剑客)
  - [正则的简单认识](#正则的简单认识)
    - [限定符](#限定符)
    - [运算符](#运算符)
    - [元字符](#元字符)
    - [贪婪性和懒惰性](#贪婪性和懒惰性)
  - [awk](#awk)
    - [awk的运算符](#awk的运算符)
    - [awk的内置变量](#awk的内置变量)
    - [awk使用的正则](#awk使用的正则)
    - [awk的判断循环和数组](#awk的判断循环和数组)
    - [awk函数](#awk函数)
    - [awk练习习](#awk练习习)
  - [sed](#sed)
    - [sed的定位](#sed的定位)
    - [sed的元字符](#sed的元字符)
    - [sed的参数](#sed的参数)
    - [sed的操作命令和替换标志](#sed的操作命令和替换标志)
    - [sed的一些例子](#sed的一些例子)
    - [sed练习](#sed练习)
  - [grep](#grep)
    - [grep 使用的元字符](#grep-使用的元字符)
    - [grep 的参数](#grep-的参数)
    - [grep 的一些例子](#grep-的一些例子)
    - [grep练习](#grep练习)
# Linux 基础

---
---
---

# 正则表达式和linux三剑客

## 正则的简单认识

### 限定符

| 符号 | 描述                              |
| ---- | --------------------------------- |
| ?    | 出现0或1次                        |
| *    | 出现0或多次                       |
| +    | 一次或多次                        |
| {}   | 限定出现几次，如{2,6}、{5,}、{,3} |

- 以上匹配单个字符，如果想要多个字符的连续出现，可以用括号

### 运算符

| 符号 | 描述                                             |
| ---- | ------------------------------------------------ |
| \|   | 或运算符，如`a (dog\|cat)`就是匹配a dog、a cat   |
| [ ]  | 字符类，如[a-zA-Z0-9]+就是匹配所有的字符和数字类 |
| ^    | 非操作，如[^0-9]就是匹配所有的非数字类           |

### 元字符

| 符号 | 描述                                               |
| ---- | -------------------------------------------------- |
| \d   | 与[0-9]相同                                        |
| \w   | 英文字符，数字、字母、下划线                       |
| \s   | 空格符，包含tab和换行符                            |
| \D   | 代表非数字字符                                     |
| \W   | 代表非单词字符                                     |
| \S   | 代表非空格字符                                     |
| `.*` | 代表任意字符，不包含换行符                         |
| ^    | 行首                                               |
| $    | 行尾                                               |
| \b   | 表示边界，防止有些字符只有部分符合，但也被匹配进来 |

### 贪婪性和懒惰性
- `*+{}`在匹配字符串会尽可能匹配到多的字符 

比如`<span>hello,world<span>`,如果用`<.+>`去匹配字符，就会把所有内容都匹配到，`？`代表出现0和1次，于是可以用`<.+?>`来正确匹配

- [.]就表示句点

## awk

### awk的运算符

| 符号                    | 描述                         |
| ----------------------- | ---------------------------- |
| = += -= *= /= %= ^= **= | 赋值运算                     |
| \|\|&& !                | 或和与和非                   |
| ~ !~                    | 匹配正则或不匹配             |
| < <= > >= != ==         | 关系运算                     |
| + - * / & ^ *** ++ --   | 算数运算                     |
| $ ?: In                 | 字段引用、三目运算、是否存在 |

- **举例子**
```sh
#三目运算
[root@yum tmp]# awk 'BEGIN{a="b";print a=="b"?"ok":"err"}'
ok
[root@yum tmp]# awk 'BEGIN{a="b";print a=="c"?"ok":"err"}'
err
#算数运算会将操作对象转为数值
[root@yum tmp]# awk 'BEGIN{a="b";print a++,++a}' 
0 2
[root@yum tmp]# awk 'BEGIN{a="20b4";print a++,++a}' 
20 22
#都是数值就比大小，如果是其他类型，即使对应ASCII码的比较
[root@yum tmp]# awk 'BEGIN{a="11";if(a>=9){print "ok"}}'
[root@yum tmp]# awk 'BEGIN{a=11;if(a>=9){print "ok"}}' 
ok
[root@yum tmp]# awk 'BEGIN{a;if(a>=b){print "ok"}}' 
ok
#正则的运算
[root@yum tmp]# awk 'BEGIN{a="100testaaa";if(a~/100/){print "ok"}}' 
ok
[root@yum tmp]# echo|awk 'BEGIN{a="100testaaa"}a~/100/{print "ok"}' 
ok
#逻辑运算
[root@yum tmp]# awk 'BEGIN{a=1;b=2;print (a>2&&b>1,a=1||b>1)}'
0 1
#赋值运算
[root@yum tmp]# awk 'BEGIN{a=5;a+=5;print a}' 
10
```

### awk的内置变量

| 变量名 | 属 性                                 |
| ------ | ------------------------------------- |
| $0     | 当前记录                              |
| $1~$n  | 当前记录的第n个字段                   |
| FS     | 输入字段分隔符，默认是空格            |
| RS     | 输入记录分割符，默认为换行符          |
| NF     | 当前记录中的字段个数，就是有多少列    |
| NR     | 已经读出的记录数，就是行号，从 1 开始 |
| OFS    | 输出字段分隔符 默认也是空格           |
| ORS    | 输出的记录分隔符 默认为换行符         |

- ***举例子***
```sh
#FS="\t" 一个或多个 Tab 分隔
[root@yum tmp]# cat tab.txt
ww      CC      IDD
[root@yum tmp]# awk 'BEGIN{FS="\t+"}{print $1,$2,$3}' tab.txt 
ww CC IDD
#FS="[" ":]+" 以一个或多个空格或：分隔
[root@yum tmp]# cat hello.txt 
root:x:0:0:root: /root:/bin/bash
[root@yum tmp]# awk -F [" ":]+ '{print $1,$2,$3}' hello.txt 
root x 0
#字段数量 NF
[root@yum tmp]# echo 'we are studing awk now!' | awk '{print NF}'
[root@yum tmp]# cat hello.txt 
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin:888
[root@yum tmp]# awk -F ":" 'NF==8{print $0}' hello.txt 
bin:x:1:1:bin:/bin:/sbin/nologin:888
#记录数量
[keri@Centos7 ~]# ip addr | awk 'NR==3{print $2}'
# OFS输出字段分隔符
[root@yum tmp]# cat hello.txt 
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin:888
[root@yum tmp]# awk 'BEGIN{FS=":"}{print $1"#"$2"#"$3}' hello.txt 
root#x#0
bin#x#1
[root@yum tmp]# awk 'BEGIN{FS=":";OFS="#"}{print $1,$2,$3}' hello.txt 
root#x#0
bin#x#1
```

### awk使用的正则

| 元字符            | 功能                              | 示例                                             | 解释                                                             |
| ----------------- | --------------------------------- | ------------------------------------------------ | ---------------------------------------------------------------- |
| ^                 | 行首定位符                        | /^root/                                          | 匹配所有以 root 开头的行                                         |
| $                 | 行尾定位符                        | /root$/                                          | 匹配所有以 root 结尾的行                                         |
| .                 | 匹配任意单个字符                  | /r..t/                                           | 匹配字母 r,然后两个任意字符，再以t结尾的行，比如 root,reat等     |
| *                 | 匹配 0 个或多个前导字符(包括回车) | /a*ool/                                          | 匹配 0 个或多个 a 之后紧跟着 ool 的行，比如 ool，aaaaool 等      |
| +                 | 匹配 1 个或多个前导字符           | /a+b/ 匹配 1 个或多个 a 加 b 的行，比如ab,aab 等 |
| ?                 | 匹配 0 个或 1 个前导字符          | /a?b/                                            | 匹配 b 或 ab 的行                                                |
| []                | 匹配指定字符组内的任意一个字符    | /^[abc]/ 匹配以字母 a 或 b 或 c 开头的行         |
| [^]               | 匹配不在指定字符组内任意一个字符  | /^[^abc]/                                        | 匹配不以字母 a 或 b 或 c 开头的行                                |
| ()                | 子表达式组合                      | /(rool)+/                                        | 表示一个或多个 rool 组合，当有一些字符需要组合时，使用括号括起来 |
| \|                | 或者的意思                        | /(root)                                          | B/                                                               | 匹配 root 或者 B 的行 |
| \                 | 转义字符                          | /a\/\//                                          | 匹配 a//                                                         |
| ~,!~              | 匹配，不匹配的条件语句            | $1~/root/ 匹配第一个字段包含字符 root 的所有记录 |
| x{m} x{m,} X{m,n} | x 重复 m、至少m、在m和n之间次     | /(root){3}/ 、/root{3}/                          | 表示rootrootroot、roottt                                         |

- ***举列子***
```sh
# 规则表达式
[root@yum tmp]# awk '/root/{print $0}' passwd 
root:x:0:0:root: /root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
[root@yum tmp]# awk -F : '$5~/root/{print $0}' passwd 
root:x:0:0:root: /root:/bin/bash
[root@yum tmp]# ifconfig eth0|awk 'BEGIN{FS="[[:space:]:]+"} NR==2{print $4}' #取出 ip，默认分隔符就是空格
192.168.68.33
#布尔表达式
[root@yum tmp]# awk -F: '$1=="root"{print $0}' passwd 
root:x:0:0:root: /root:/bin/bash
[root@yum tmp]# awk -F: '($1=="root")&&($5=="root"){print $0}' passwd 
root:x:0:0:root: /root:/bin/bash
```

### awk的判断循环和数组
- if例子
```sh
{
 if ( $1== "foo" ) {
 if ( $2== "foo" ) {
 print "uno"
 } else {
 print "one"
 }
 } elseif ($1== "bar" ) {
 print "two"
 } else {
 print "three"
 }
}
```
- do-while例子
```sh
{
 count=1
 do {
 print "I get printed at least once no matter what"
 } while ( count !=1 )
}
```
- for例子
```sh
for ( x=1;x<=4;x++ ) {
 print "iteration", x
}
```
- break例子
```sh
x=1
while(1) {
 print "iteration", x
 if ( x==10 ) {
 break
 }
 x++
}
```
- continue例子
```sh
for ( x=1;x<=21;x++ ) {
 if ( x==4 ) {
 continue
 }
 print "iteration", x
}
``

#### awk数组的应用
```sh
#查看服务器连接总数，并统计
netstat -an|awk '/^tcp/{++s[$NF]}END{for(a in s)print a,s[a]}' 
ESTABLISHED 1 
LISTEN 20
#统计 web 日志访问流量，要求输出访问次数，请求页面或图片，每个请求的总大小，总访问流量的大小汇总
awk '{a[$7]+=$10;++b[$7];total+=$10}END{for(x in a)print b[x],x,a[x]|"sort -rn -k1";print "total size is :"total}' /app/log/access_log 
total size is :172230 
21 /icons/poweredby.png 83076 
14 / 70546 
8 /icons/apache_pb.gif 18608
```

### awk函数

| 函数                                | 说明                                                                                                                                                                                                                                                                                            |
| ----------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| gsub( Ere, Repl, [ In ] )           | 除了正则表达式所有具体值被替代这点，它和 sub 函数完全一样地执行。                                                                                                                                                                                                                               |
| sub( Ere, Repl, [ In ] )            | 用 Repl 参数指定的字符串替换 In 参数指定的字符串中的由 Ere 参数指定的扩展正则表达式的第一个具体值。sub 函数返回替换的数量。出现在 Repl 参数指定的字符串中的 &（和符号）由 In 参数指定的与 Ere 参数的指定的扩展正则表达式匹配的字符串替换。如果未指定 In 参数，缺省值是整个记录（$0 记录变量）。 |
| index( String1, String2 )           | 在由 String1 参数指定的字符串（其中有出现 String2 指定的参数）中，返回位置，从 1 开始编号。如果 String2 参数不在 String1 参数中出现，则返回 0（零）。                                                                                                                                           |
| length [(String)]                   | 返回 String 参数指定的字符串的长度（字符形式）。如果未给出String 参数，则返回整个记录的长度（$0 记录变量）。                                                                                                                                                                                    |
| blength [(String)]                  | 返回 String 参数指定的字符串的长度（以字节为单位）。如果未给出String 参数，则返回整个记录的长度（$0 记录变量）。                                                                                                                                                                                |
| substr( String, M, [ N ] )          | 返回具有 N 参数指定的字符数量子串。子串从 String 参数指定的字符串取得，其字符以 M 参数指定的位置开始。M 参数指定为将String 参数中的第一个字符作为编号 1。如果未指定 N 参数，则子串的长度将是 M 参数指定的位置到 String 参数的末尾 的长度。                                                      |
| match( String, Ere )                | 在 String 参数指定的字符串（Ere 参数指定的扩展正则表达式出现在其中）中返回位置（字符形式），从 1 开始编号，或如果 Ere 参数不出现，则返回 0（零）。                                                                                                                                              |
| RSTART                              | 特殊变量设置为返回值。RLENGTH 特殊变量设置为匹配的字符串的长度，或如果未找到任何匹配，则设置为 -1（负一）。                                                                                                                                                                                     |  | split( String, A, [Ere] ) | 将 String 参数指定的参数分割为数组元素 A[1], A[2], . . ., A[n]，并返回 n 变量的值。此分隔可以通过 Ere 参数指定的扩展正则表达式进行，或用当前字段分隔符（FS 特殊变量）来进行（如果没有给出 Ere 参数）。除非上下文指明特定的元素还应具有一个数字值，否则 A 数组中的元素用字符串值来创建。 |
| tolower( String )                   | 返回 String 参数指定的字符串，字符串中每个大写字符将更改为小写。大写和小写的映射由当前语言环境的 LC_CTYPE 范畴定义。                                                                                                                                                                            |
| toupper( String )                   | 返回 String 参数指定的字符串，字符串中每个小写字符将更改为大写。大写和小写的映射由当前语言环境的 LC_CTYPE 范畴定义。                                                                                                                                                                            |
| sprintf(Format, Expr, Expr, . . . ) | 根据 Format 参数指定的 printf 子例程格式字符串来格式化 Expr 参数指定的表达式并返回最后生成的字符串。                                                                                                                                                                                            |

- ***举例子***
```sh
#替换
awk 'BEGIN{info="this is a test2010test!";gsub(/[0-9]+/,"!",info);print info}' this is a test!test! 
#在 info 中查找满足正则表达式，/[0-9]+/ 用”!”替换，并且替换后的值，赋值给 info 未给 info 值，默认是$0 
#查找
awk 'BEGIN{info="this is a test2010test!";print index(info,"test")?"ok":"no found";}' 
ok #未找到，返回 0 
#匹配查找
awk 'BEGIN{info="this is a test2010test!";print match(info,/[0-9]+/)?"ok":"no found";}' 
ok #如果查找到数字则匹配成功返回 ok，否则失败，返回未找到
#截取
awk 'BEGIN{info="this is a test2010test!";print substr(info,4,10);}' 
s is a tes #从第 4 个 字符开始，截取 10 个长度字符串
#分割
awk 'BEGIN{info="this is a test";split(info,tA," ");print length(tA);for(k in tA){print k,tA[k];}}' 4 
4 test 1 this 2 is 3 a 
#分割 info,动态创建数组 tA,awk for …in 循环，是一个无序的循环。 并不是从数组下标1…n 开始
```

### awk练习习
> 1. 提取以下字符串Yd12&_34UJ9ad)(10[]%&中所有的数字
```sh
echo "Yd12&_34UJ9ad)(10[]%&" | awk  'gsub(/[^[:digit:]]/,"",$0)'
echo "Yd12&_34UJ9ad)(10[]%&"|awk '{string=$0;len=length(string);for(i=0;i<=len;i++){tmp=substr(string,i,1);if(tmp~/[0-9]/){str=(str tmp)}}print str}'
echo "Yd12&_34UJ9ad)(10[]%&"|awk -F "[^[:digit:]]" '{for(k=1;k<=NF;k++){count[$k]}}END{for(i in count){printf "%s",i}printf   "\n"}'

```
>2. /etc/passwd 中shell的个数（最后一列）
```sh
awk -F ":" '{print $NF}' /etc/passwd|sort |uniq -c 
awk -F ":" '{fs[$NF]++}END{for (i in fs){print i,fs[i]}}' /etc/passwd
```
>3. 姓名,性别,科目,成绩
```text
chen min,man,history,123
liu hong,woman,english,143
jiang hao,man,english,113
chen wu,woman,math,133
li hua,man,history,134
lv fang,woman,math,67
gui quan,man,english,139
liu can,woman,math,59
```
默认文本导入a.txt
1. 显示所有男性的姓名
```sh
awk '/,man/{split($0,a,",");print a[1]}' a.txt
```
2. 统计每个科目的人数
```sh
awk '{split($0,a,",");fs[a[3]]++}END{for(i in fs){print i,fs[i]}}' a.txt
awk -F "," '{fs[$3]++} END {for(i in fs){print i,fs[i]}}' a.txt
awk -F "," '{print $3}' a.txt| sort| uniq -c
```
3. 显示li姓的考试分数
```sh
awk '/li /{FS=",";print $4}' a.txt
```
4. 显示考试得分最高的人姓名
```sh
awk '{split($0,a,",");fs[a[1]]=a[4]}END{maxname=0;max=0;for(i in fs){if(fs[i]>max){maxname=i;max=fs[i]}};print maxname,max}' a.txt
awk  'BEGIN{FS=","}{print $1" "$4}' a.txt |sort -n  -k  3|tail -1
awk -F ',' '{print $1,$4}' a.txt | sort -rnk3 | head -1
```
5. 显示数学考试得分最低的人姓名和分数
```sh
grep ',math,' a.txt |awk  'BEGIN{FS=","}{print $1" "$4}' a.txt |sort -n  -k  3|head -1
awk '/,math,/{split($0,a,",");fs[a[1]]=a[4]}END{minname=0;min=150;for(i in fs){if(fs[i]<min){minname=i;min=fs[i]}};print minname,min}' a.txt
awk -F ',' '/,math,/{print $1,$4}' a.txt|sort -rnk3|head -1
```
6. 显示不及格的人姓名和分数（小于等于90）
```sh
awk -F ',' '$4<=90{print $1,$3,$4}' a.txt
```

## sed
### sed的定位
1. sed -n 2~5p 含义：从第二行开始匹配，隔 5 行匹配一次，即 2,7,12.......。
2. $符表示匹配最后一行
3. `/REGEXP/`，这个是表示匹配正则那一行，通过//之间的正则来匹配。
4. `\cREGEXPc`，这个是表示匹配正则那一行，通过\c 和 c 之间的正则来匹配,c 可以是任一字符
5. addr1,addr2,两行之间的内容
6. addr1;addr2,分别区这两行的内容
7. addr1,+N,从add1开始匹配N行，一共N+1行

### sed的元字符

| 元字符            | 功能                              | 示例                                      | 解释                                                                                 |
| ----------------- | --------------------------------- | ----------------------------------------- | ------------------------------------------------------------------------------------ |
| ^                 | 行首定位符                        | /^root/                                   | 匹配所有以 root 开头的行                                                             |
| $                 | 行尾定位符                        | /root$/                                   | 匹配所有以 root 结尾的行                                                             |
| .                 | 匹配任意单个字符                  | /r..t/                                    | 匹配字母 r,然后两个任意字符，再以t结尾的行，比如 root,reat等                         |
| *                 | 匹配 0 个或多个前导字符(包括回车) | /a*ool/                                   | 匹配 0 个或多个 a 之后紧跟着 ool 的行，比如 ool，aaaaool 等                          |
| [^]               | 匹配不在指定字符组内任一字符      | /[^A-KM-Z]ove/                            | 匹配包含 ove，但 ove 之前的那个字符不在 A 至 K 或 M 至 Z 间的行                      |
| \\(..\\)          | 保存已匹配的字符                  |                                           |                                                                                      |
| &                 | 保存查找串以便在替换串中引用      | s/love/**&**/                             | 符号&代表查找串。字符串 love将替换前后各加了两个\*\*的引用，即 love 变成\*\*love\*\* |
| \\<               | 词首定位符                        | `/\<love/ 匹配包含以 love 开头的单词的行` |
| \\>               | 词尾定位符                        | `/love\>/ 匹配包含以 love 结尾的单词的行` |
| x{m} x{m,} X{m,n} | x 重复 m、至少m、在m和n之间次     | /(root){3}/ 、/root{3}/                   | 表示rootrootroot、roottt                                                             |
### sed的参数

| 选项 | 说明                                         |
| ---- | -------------------------------------------- |
| -n   | 使用安静模式加入-n 后只打印被sed特殊处理的行 |
| -e   | 多重编辑，且命令顺序会影响结果               |
| -f   | 指定一个 sed 脚本文件到命令行执行，          |
| -r   | Sed 使用扩展正则                             |
| -i   | 直接修改文档读取的内容，不在屏幕上输出       |

### sed的操作命令和替换标志

| 命 令 | 说 明                                                          |
| ----- | -------------------------------------------------------------- |
| a\    | 在当前行后添加一行或多行                                       |
| c\    | 用新文本修改（替换）当前行中的文本d 删除行                     |
| i\    | 在当前行之前插入文本                                           |
| h     | 把模式空间里的内容复制到暂存缓存区                             |
| H     | 把模式空间里的内容追加到暂存缓存区                             |
| g     | 取出暂存缓冲区里的内容，将其复制到模式空间，覆盖该处原有内容   |
| G     | 取出暂存缓冲区里的内容，将其复制到模式空间，追加在原有内容后面 |
| l     | 列出非打印字符                                                 |
| p     | 打印行                                                         |
| n     | 读入下一输入行，并从下一条命令而不是第一条命令开始处理         |
| q     | 结束或退出 sed                                                 |
| r     | 从文件中读取输入行                                             |
| ！    | 对所选行意外的所有行应用命令                                   |
| s     | 用一个字符串替换另一个                                         |

| 替换标志 | 解释                                                |
| -------- | --------------------------------------------------- |
| g        | 在行内进行全局替换                                  |
| p        | 打印行                                              |
| w        | 将行写入文件                                        |
| x        | 交换暂存缓冲区与模式空间的内容                      |
| y        | 将字符转换为另一字符（不能对正则表达式使用 y 命令） |

### sed的一些例子
```sh
#下面给出测试文件 ceshi.txt 作为输入文件。 
northwest NW Charles Main 3.0 .98 3 34 
western WE Sharon Gray 5.3 .97 5 23 
southwest SW Lewis Dalsass 2.7 .8 2 18 
southern SO Suan Chin 5.1 .95 4 15 
southeast SE Patricia Hemenway 4.0 .7 4 17 
eastern EA TB Savage 4.4 .84 5 20 
northeast NE AM Main Jr. 5.1 .94 3 13 
north NO Margot Weber 4.5 .89 5 9 
central CT Ann Stephens 5.7 .94 5 13
#打印目标行
[root@yum test]# sed -n '/north/p' sed.txt 
 northwest NW Charles Main 3.0 .98 3 34
 northeast NE AM Main Jr. 5.1 .94 3 13
 north NO Margot Weber 4.5 .89 5 9
 #删除目标行，打印剩余行
 [root@yum test]# sed '/north/d' sed.txt 
 western WE Sharon Gray 5.3 .97 5 23
 southwest SW Lewis Dalsass 2.7 .8 2 18
 southern SO Suan Chin 5.1 .95 4 15
 southeast SE Patricia Hemenway 4.0 .7 4 17
 eastern EA TB Savage 4.4 .84 5 20
 central CT Ann Stephens 5.7 .94 5 13
 ##只打印被替换的行
[root@yum test]# sed -n 's/Hemenway/Jones/gp' sed.txt 
southeast SE Patricia Jones 4.0 .7 4 17 
#当“与”符号（&）用在替换串中时，它代表在查找串中匹配到的内容时。这个示例中所有以 2 位数结尾的行后面都被加上.5。
[root@yum test]# sed 's/[0-9][0-9]$/&.5/' sed.txt 
northwest NW Charles Main 3.0 .98 3 34.5
western WE Sharon Gray 5.3 .97 5 23.5
southwest SW Lewis Dalsass 2.7 .8 2 18.5
southern SO Suan Chin 5.1 .95 4 15.5
southeast SE Patricia Hemenway 4.0 .7 4 17.5
eastern EA TB Savage 4.4 .84 5 20.5
northeast NE AM Main Jr. 5.1 .94 3 13.5
north NO Margot Weber 4.5 .89 5 9
central CT Ann Stephens 5.7 .94 5 13.5
#包含在圆括号里的模式 Mar 作为标签 1 保存在特定的寄存器中。替换串可以通过\1 来引用它。则 Margot 被替换为 Marlinane
[root@yum test]# sed 's/\(Mar\)got/\1linanne/p' sed.txt 
northwest NW Charles Main 3.0 .98 3 34
western WE Sharon Gray 5.3 .97 5 23
southwest SW Lewis Dalsass 2.7 .8 2 18
southern SO Suan Chin 5.1 .95 4 15
southeast SE Patricia Hemenway 4.0 .7 4 17
eastern EA TB Savage 4.4 .84 5 20
northeast NE AM Main Jr. 5.1 .94 3 13
north NO Marlinanne Weber 4.5 .89 5 9
north NO Marlinanne Weber 4.5 .89 5 9
central CT Ann Stephens 5.7 .94 5 13
#打印模式 west 和 east 之间所有的行。如果 west 出现在 east 之后的某一行，则打印的范围从 west 所在行开始，到下一个出现 east 的行或文件的末尾（如果前者未出现）。
[root@yum test]# sed -n '/west/,/east/p' sed.txt 
northwest NW Charles Main 3.0 .98 3 34
western WE Sharon Gray 5.3 .97 5 23
southwest SW Lewis Dalsass 2.7 .8 2 18
southern SO Suan Chin 5.1 .95 4 15
southeast SE Patricia Hemenway 4.0 .7 4 17
#打印从第 5 行开始第一个以 northeast 开头的行之间的所有行。
[root@yum test]# sed -n '5,/northeast/p' sed.txt 
southeast SE Patricia Hemenway 4.0 .7 4 17eastern EA TB Savage 4.4 .84 5 2northeast NE AM Main Jr. 5.1 .94 3 13
#修改从模式 wast 和 east 之间的所有行，将各行的行尾($)替换为字符串**VACA**。换行符被移到新的字符串后面
[root@yum test]# sed '/west/,/east/s/$/**VACA**/' sed.txt 
northwest NW Charles Main 3.0 .98 3 34**VACA**
western WE Sharon Gray 5.3 .97 5 23**VACA**
southwest SW Lewis Dalsass 2.7 .8 2 18**VACA**
southern SO Suan Chin 5.1 .95 4 15**VACA**
southeast SE Patricia Hemenway 4.0 .7 4 17**VACA**
eastern EA TB Savage 4.4 .84 5 20
northeast NE AM Main Jr. 5.1 .94 3 13
north NO Margot Weber 4.5 .89 5 9
central CT Ann Stephens 5.7 .94 5 13
#选项-e 用于进行多重编辑。第一重编辑编辑删除第 1~3 行。第二重编辑将Hemenway 替换为 Jones。因为是逐行进行这两行编辑（即这两个命令都在模式空间的当前行上执行），所以编辑命令的顺序会影响结果。例如，如果两条命令都执行的是替换，前一次替换会影响后一次替换。
[root@yum test]# sed -e '1,3d' -e 's/Hemenway/Jones/' sed.txt 
southern SO Suan Chin 5.1 .95 4 15
southeast SE Patricia Jones 4.0 .7 4 17
eastern EA TB Savage 4.4 .84 5 20
northeast NE AM Main Jr. 5.1 .94 3 13
north NO Margot Weber 4.5 .89 5 9
central CT Ann Stephens 5.7 .94 5 13
#命令 a 用于追加。字符串 Hello，World！被加在以 north 开头，north 后面跟一个空格的各行之后。如果要追加的内容超过一行，则除最后一行外，其他各行都必须以反斜杠结尾
[root@yum test]# sed '/^north /a Hello,World!' sed.txt 
northwest NW Charles Main 3.0 .98 3 34
western WE Sharon Gray 5.3 .97 5 23
southwest SW Lewis Dalsass 2.7 .8 2 18
southern SO Suan Chin 5.1 .95 4 15
southeast SE Patricia Hemenway 4.0 .7 4 17
eastern EA TB Savage 4.4 .84 5 20
northeast NE AM Main Jr. 5.1 .94 3 13
north NO Margot Weber 4.5 .89 5 9
Hello,World!
central CT Ann Stephens 5.7 .94 5 13
#命令 i 是插入命令。如果在某一行匹配到模式 eastern,i 命令就在该行的上方插入命令中插入反斜杠后面后的文本。除了最后一行。
[root@yum test]# sed '/eastern/i Hello,World! -----------------------------------' sed.txt
northwest NW Charles Main 3.0 .98 3 34
western WE Sharon Gray 5.3 .97 5 23
southwest SW Lewis Dalsass 2.7 .8 2 18
southern SO Suan Chin 5.1 .95 4 15
southeast SE Patricia Hemenway 4.0 .7 4 17
Hello,World! -----------------------------------
eastern EA TB Savage 4.4 .84 5 20
northeast NE AM Main Jr. 5.1 .94 3 13
north NO Margot Weber 4.5 .89 5 9
central CT Ann Stephens 5.7 .94 5 13
#c 命令是修改命令。该命令将完整地修改在模式缓冲区行的当前行。如果模式 eastern被匹配，c 命令将其后的文本替换包含 eastern 的行。
[root@yum test]# sed '/eastern/c Hello,World! -----------------------------------' sed.txt
northwest NW Charles Main 3.0 .98 3 34
western WE Sharon Gray 5.3 .97 5 23
southwest SW Lewis Dalsass 2.7 .8 2 18
southern SO Suan Chin 5.1 .95 4 15
southeast SE Patricia Hemenway 4.0 .7 4 17
Hello,World! -----------------------------------
northeast NE AM Main Jr. 5.1 .94 3 13
north NO Margot Weber 4.5 .89 5 9
central CT Ann Stephens 5.7 .94 5 13
#如果在某一行匹配到模式 eastern，n 命令就指示 sed 用下一个输入行（即包含 AM Main Jr 的那行）替换模式空间中的当前行，并用 Archie 替换 AM
[root@yum test]# sed '/eastern/{n;s/AM/Archie/;}' sed.txt 
northwest NW Charles Main 3.0 .98 3 34
western WE Sharon Gray 5.3 .97 5 23
southwest SW Lewis Dalsass 2.7 .8 2 18
southern SO Suan Chin 5.1 .95 4 15
southeast SE Patricia Hemenway 4.0 .7 4 17
eastern EA TB Savage 4.4 .84 5 20
northeast NE Archie Main Jr. 5.1 .94 3 13
north NO Margot Weber 4.5 .89 5 9
central CT Ann Stephens 5.7 .94 5 13
#y 命令把 1~3 行中所有的小写命令字母都转换成了大写。正则表达式元字符对 y 命令不起作用。与替分隔符一样，斜杠可以被替换成其他字符
[root@yum test]# sed 
'1,3y/abcdefghijklmnopqrstuvwxyz/ABCDEFGHIJKLMNOPQRSTUVWXYZ/' sed.txt 
NORTHWEST NW CHARLES MAIN 3.0 .98 3 34
WESTERN WE SHARON GRAY 5.3 .97 5 23
SOUTHWEST SW LEWIS DALSASS 2.7 .8 2 18
southern SO Suan Chin 5.1 .95 4
#在某行匹配到模式 Lewis 时，s 表示先用 Joseph 替换 Lewis，然后 q 命令让 sed 退出。
[root@yum test]# sed '/Lewis/{ s/Lewis/Joseph/;q; }' sed.txt 
northwest NW Charles Main 3.0 .98 3 34
western WE Sharon Gray 5.3 .97 5 23
southwest SW Joseph Dalsass 2.7 .8 2 18
```
```sh
#在修改配置文件的时候，有一些空格、空行、带“#”开头的注释都要删除或替换
sed 's/^[ ]*//' sed2.txt
sed 's/^[[:space:]]*//' sed2.txt
grep -viE "^#|^$" ssh_config
sed '/^#|^$/d' ssh_config
#从 Google 上下载下来的配置文件往往都带有数字，现在需要删除所有行的首数字。
sed 's/^[0-9][0-9]*//g' sed2.txt
```
### sed练习
>1. 取出/etc/passwd中第一列和最后一列
```sh
sed 's/:.*:/ /g' /etc/passwd
```
>2. 取出/etc/passwd中1到5行
```sh
sed -n '1,5p' /etc/passwd
```
>3. 取出/etc/passwd中1到5行中第一列
```sh
sed -n '1,5 s/:.*/ /gp' /etc/passwd
```
>4. 取出/etc/passwd第二列 
```sh
sed -E "s/^([^:]+):([^:]+):(.+)/\2/g" passwd
sed 's/^[^:]*:\([^:]*\):[^:]*:[^:]*:[^:]*:[^:]*:[^:]*/\1/g' passwd
```

## grep
### grep 使用的元字符

| 元字符            | 功能                              | 示例                                      | 解释                                                                                 |
| ----------------- | --------------------------------- | ----------------------------------------- | ------------------------------------------------------------------------------------ |
| ^                 | 行首定位符                        | /^root/                                   | 匹配所有以 root 开头的行                                                             |
| $                 | 行尾定位符                        | /root$/                                   | 匹配所有以 root 结尾的行                                                             |
| .                 | 匹配任意单个字符                  | /r..t/                                    | 匹配字母 r,然后两个任意字符，再以t结尾的行，比如 root,reat等                         |
| *                 | 匹配 0 个或多个前导字符(包括回车) | /a*ool/                                   | 匹配 0 个或多个 a 之后紧跟着 ool 的行，比如 ool，aaaaool 等                          |
| [^]               | 匹配不在指定字符组内任一字符      | /[^A-KM-Z]ove/                            | 匹配包含 ove，但 ove 之前的那个字符不在 A 至 K 或 M 至 Z 间的行                      |
| \\(..\\)          | 保存已匹配的字符                  |                                           |                                                                                      |
| &                 | 保存查找串以便在替换串中引用      | s/love/**&**/                             | 符号&代表查找串。字符串 love将替换前后各加了两个\*\*的引用，即 love 变成\*\*love\*\* |
| \\<               | 词首定位符                        | `/\<love/ 匹配包含以 love 开头的单词的行` |
| \\>               | 词尾定位符                        | `/love\>/ 匹配包含以 love 结尾的单词的行` |
| x{m} x{m,} X{m,n} | x 重复 m、至少m、在m和n之间次     | /(root){3}/ 、/root{3}/                   | 表示rootrootroot、roottt                                                             |

### grep 的参数

| 选 项 | 功 能                                                                                                                                         |
| ----- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| -E    | 如果加这个选项，那么后面的匹配模式就是扩展的正则表达式，也就是 grep -E = egrep                                                                |
| -i    | 比较字符时忽略大小写区别                                                                                                                      |
| -w    | 把表达式作为词来查找，相当于正则中的"`\<...\>`"(...表示你自定义的规则)                                                                        |
| -x    | 被匹配到的内容，正好是整个行，相当于正则"^...$"                                                                                               |
| -v    | 取反，也就是输出我们定义模式相反的内容                                                                                                        |
| -c    | count.统计，统计匹配结果的行数，主要不是匹配结果的次数，是行数。                                                                              |
| -m    | 只匹配规定的行数，之后的内容就不在匹配了                                                                                                      |
| -n    | 在输出的结果里显示行号，这里要清楚的是这里所谓的行号是该行内容在原文件中的行号，而不是在输出结果中行号                                        |
| -o    | 只显示匹配内容，grep 默认是显示满足匹配条件的一行，加上这个参数就只显示匹配结果，比如我们要匹配一个 ip 地址，就只需要结果，而不需要该行的内容 |
| -R    | 递归匹配。如果要在一个目录中多个文件或目录匹配内容，则需要这个参数                                                                            |
| -B    | 输出满足条件行的前几行，比如 grep -B 3 "aa" file 表示在 file 中输出有 aa 的行，同时还要输出 aa 的前 3 行                                      |
| -A    | 这个与-B 类似，输出满足条件行的后几行                                                                                                         |
| -C    | 这个相当于同时用-B -A，也就是前后都输出                                                                                                       |
### grep 的一些例子
```sh
#以下所有的例子都是用的 grep.txt 文件。
northwest NW Charles Main 3.0 .98 3 34
western WE Sharon Gray 5.3 .97 5 23
southwest SW Lewis Dalsass 2.7 .8 2 18
southern SO Suan Chin 5.1 .95 4 15
southeast SE Patricia Hemenway 4.0 .7 4 17
eastern EA TB Savage 4.4 .84 5 20
northeast NE AM Main Jr. 5.1 .94 3 13
north NO Margot Weber 4.5 .89 5 9
central CT Ann Stephens 5.7 .94 5 13
[root@yum test]# grep "4$" grep.txt 
northwest NW Charles Main 3.0 .98 3 34
#打印所有以数字 4 结尾的行。($)行尾定位符
[root@yum test]# grep '5\..' grep.txt 
western WE Sharon Gray 5.3 .97 5 23
southern SO Suan Chin 5.1 .95 4 15
northeast NE AM Main Jr. 5.1 .94 3 13
central CT Ann Stephens 5.7 .94 5 13
#打印所有包含数字 5，后面跟一个.号 再跟一个任意字符的行。 (.)号代表单个字符，被(\)转义后，只代表本身一个.号。
[root@yum test]# grep '^[we]' grep.txt
western WE Sharon Gray 5.3 .97 5 23
eastern EA TB Savage 4.4 .84 5 20
#打印所有字母 w 和 e 开头的行。[]表示任意一个字符都可以匹配。
[root@yum test]# grep '[A-Z][A-Z] [A-Z]' grep.txt 
eastern EA TB Savage 4.4 .84 5 20
northeast NE AM Main Jr. 5.1 .94 3 13
#打印了包含两个大写字符、后跟一个空格和一个大写字符的行，例如 TB Savage 和 MB Main
[root@yum test]# grep 'ss* ' grep.txt 
northwest NW Charles Main 3.0 .98 3 34
southwest SW Lewis Dalsass 2.7 .8 2 18
central CT Ann Stephens 5.7 .94 5 13
#打印包含一个 s、后跟 0 个或多个连着的 s 和一个空格的文本行。
[root@yum test]# grep '\(3\)\.[0-9].*\1 *\1' grep.txt 
northwest NW Charles Main 3.0 .98 3 34
#如果某一行包含一个 3 后面跟一个句点和一个数字，再任意多个字符(.*),然后跟一个 3个或任意多个制表符，再接一个 3，则打印该行(`\1`表示第一个被匹配的内容，就是3)。
[root@yum test]# grep '\<north\>' grep.txt 
north NO Margot Weber 4.5 .89 5 9
#打印所有包含单词 north 的行。“`\<”是词首定位符“\>”`是词尾定位符，等同于用`-w`精确匹配
```

### grep练习
> 1. 在/var/log/messages查找ip
```sh
grep -En "([0-9]{1,3}[\.]){3}[0-9]{1,3}" /var/log/messages
grep -P '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' messages
```
> 2. 在/etc/fstab 查找以/开头的行
```sh
grep '^/' /etc/fstab
```
> 3. 在取出磁盘分区利用率并排序
```sh
df -h | grep -Eo '[0-9]{1,2}%.*'|sort -rn
df -h | grep -o " [0-9]*%.*" |sort -rn
```
> 4. 在/etc/passwd 中查找nologin的用户并计数
```sh
grep "nologin" /etc/passwd && grep -c "nologin" /etc/passwd
cat /etc/passwd |grep nologin|wc -l
```
>5. 以下几行中过滤出手机号
> - 19205541664
> - 23406513341
> - 1234556234
```sh
echo -e '19205541664\n23406513341\n1234556234' | grep -E '^1[0-9]{10}'
```