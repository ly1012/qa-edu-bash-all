
# Bash 实战

## 使用不同的关键字搜索并返回查询结果数

**脚本**

```bash
# 将关键字写入文件，每行代表每次搜索的关键字
echo -e "Java\nPython\nBash\nAndroid" > baidu.keyword
# 发起搜索请求并过滤响应正文
while read k;\
do \
    curl -s "https://www.baidu.com/s?wd=$k" \
    -H 'User-Agent: Chrome/83.0.4103.116'  \
    -H 'Cookie: BAIDUID=D00F687284B30B47ACFCAB2B52E471D6:FG=1; ' \
; \
done < baidu.keyword | grep -o '结果约[0-9,]*' | awk -F '约' '{print $2}'
```

**输出**

```
100,000,000
74,900,000
68,400,000
100,000,000
```

## 查看 bash 系统手册的文件说明部分

- `man bash` 将 man 命令的系统手册说明输出到标准输出。
- `cat -n` 从标准输入接收数据，并进行行编号。
- `tail -n +4480` 读取第 4480 行及之后的行。
- `head -n 25` 读取前 25 行。

```bash
:/test$ man bash | cat -n | tail -n +4480 | head -n 25
  4480         emacs(1), vi(1)
  4481         readline(3)
  4482
  4483  FILES
  4484         /bin/bash
  4485                The bash executable
  4486         /etc/profile
  4487                The systemwide initialization file, executed for login shells
  4488         /etc/bash.bashrc
  4489                The systemwide per-interactive-shell startup file
  4490         /etc/bash.bash.logout
  4491                The systemwide login shell cleanup file, executed when a login shell exits
  4492         ~/.bash_profile
  4493                The personal initialization file, executed for login shells
  4494         ~/.bashrc
  4495                The individual per-interactive-shell startup file
  4496         ~/.bash_logout
  4497                The individual login shell cleanup file, executed when a login shell exits
  4498         ~/.inputrc
  4499                Individual readline initialization file
  4500
  4501  AUTHORS
  4502         Brian Fox, Free Software Foundation
  4503         bfox@gnu.org
  4504
```

## 找出英文小说中出现频率最高的 10 个单词

- `grep -oE "\b[[:alpha:]]+\b" The_Count_of_Monte_Cristo.txt` 输出所有英文单词，每个单词一行。
- `awk '{word[$1]++} END {for(i in word){print i,word[i]}}'` 输出单词及出现次数，第一列为单词，第二列为单词出现次数。
- `sort -rn -k2` 按单词出现次数从大到小排序。
- `head -n 10` 取出出现次数前十高的单词及其出现次数。
- `awk '{print $1}'` 取出第一列，即单词。

```bash
:/test/practice$ grep -oE "\b[[:alpha:]]+\b" The_Count_of_Monte_Cristo.txt | awk '{word[$1]++} END {for(i in word){print i,word[i]}}' | sort -rn -k2 | head -n 10
the 24601
to 12039
of 12010
and 10917
a 8451
I 7813
you 6967
in 5894
he 5642
his 5409

:/test/practice$ grep -oE "\b[[:alpha:]]+\b" The_Count_of_Monte_Cristo.txt | awk '{word[$1]++} END {for(i in word){print i,word[i]}}' | sort -rn -k2 | head -n 10 | awk '{print $1}'
the
to
of
and
a
I
you
in
he
his

# 或者
:/test/practice$ grep -oE '\b[[:alpha:]]+\b' The_Count_of_Monte_Cristo.txt | sort | uniq -c | sort -nr | head -n 10
  24601 the
  12039 to
  12010 of
  10917 and
   8451 a
   7813 I
   6967 you
   5894 in
   5642 he
   5409 his
```

## 统计 Nginx 日志中 HTTP 状态码出现次数

```bash
[root@izj6c66dst6ergsqe2wynxz wwwlogs]# less access.log | awk -F '"' '{print $3}' | awk '{print $1}' | sort | uniq -c | sort -n -r
 683734 200
  56730 404
  35217 400
   1050 405
    497 403
    349 499
    219 301
    129 304
     19 408
      7 416
      2 206
      1 500
      1 302
```

## 找出 URL 返回文本中的 URL

一个简单版的实现：

```bash
# `curl -s` 静默模式，不输出进度条和错误信息。
# `grep -i` 不区分大小写。
# `grep -o` 只输出匹配内容，每个匹配内容为一行。
# `grep -E` 使用 ERE（扩展正则表达式）。
# `wc -l` 统计文件行数。
# `head -n3` 输出前三行。
:/test/practice$ a=$(curl -s https://qadoc.org | grep -ioE 'https?://[^" >]+'); wc -l <<< "$a"; head -n3 <<< "$a"
22
https://www.qadoc.org/
https://cdn.jsdelivr.net/npm/bootstrap@4.4.1/dist/css/bootstrap.min.css
https://hm.baidu.com/hm.js?2f2c198df727904a87109583c9bb612a
```

## 简单的系统资源监控

**脚本**

```bash
#! /usr/bin/env bash

samplenum=0
starttime=$(date +%s)
title="------------------- Cpu --------------------    -------------------- Mem ------------------------   -------------------- Swap ---------------------"
header=$(awk 'BEGIN{RS=" "} {if(FNR<=8) printf "%-6s",$0; else printf "%-13s",$0;}' <<< 'us sy ni id wa hi si st total free used buff/cache total free used availMem' | sed -e 's/ *$//g')
echo -e "$title\n$header" > stat.txt

while true
do
        data=$(top -n 1 | sed -n 3,5p)
        # echo "$data" >> stat.txt
        formatdata=$(grep -oE '[ ,]?[0-9.]+[ |,]' <<< "$data" | sed -e 's/^ *//g' -e 's/ *$//g' -e 's/,//g' | sed ':label;N;s/\n/ /g;t label' | sed 's/[ ]\{1,\}/ /g' | awk '{for(i=1; i<=NF; i++) if(i<=8) printf "%-6.1f",$i; else printf "%-13.1f",$i;printf "\n"}')
        echo "$formatdata" >> stat.txt
        clear
        ((samplenum++))
        currenttime=$(date +%F-%T)
        endtime=$(date +%s)
        staytime=$(($endtime-$starttime))
        echo -e "当前时间：$currenttime\t已启动：$staytime 秒\t第 $samplenum 次采样\n$title\n$header\n$formatdata"
        sleep 2
done
```

**效果**

```bash
# 屏幕动态刷新效果
当前时间：2020-07-31-16:58:53   已启动：15 秒   第 8 次采样
------------------- Cpu --------------------    -------------------- Mem ------------------------   -------------------- Swap ---------------------
us    sy    ni    id    wa    hi    si    st    total        free         used         buff/cache   total        free         used         availMem
3.0   3.0   0.0   93.9  0.0   0.0   0.0   0.0   3880244.0    342904.0     1064196.0    2473144.0    0.0          0.0          0.0          2471480.0

# 文件内容效果
[root@izj6c66dst6ergsqe2wynxz practice]# cat stat.txt
------------------- Cpu --------------------    -------------------- Mem ------------------------   -------------------- Swap ---------------------
us    sy    ni    id    wa    hi    si    st    total        free         used         buff/cache   total        free         used         availMem
0.0   3.2   0.0   96.8  0.0   0.0   0.0   0.0   3880244.0    343564.0     1063540.0    2473140.0    0.0          0.0          0.0          2472136.0
0.0   3.1   0.0   81.2  15.6  0.0   0.0   0.0   3880244.0    343372.0     1063732.0    2473140.0    0.0          0.0          0.0          2471944.0
3.2   0.0   0.0   96.8  0.0   0.0   0.0   0.0   3880244.0    343372.0     1063732.0    2473140.0    0.0          0.0          0.0          2471944.0
0.0   3.1   0.0   96.9  0.0   0.0   0.0   0.0   3880244.0    343372.0     1063732.0    2473140.0    0.0          0.0          0.0          2471944.0
3.1   3.1   0.0   93.8  0.0   0.0   0.0   0.0   3880244.0    342780.0     1064320.0    2473144.0    0.0          0.0          0.0          2471356.0
0.0   3.1   0.0   96.9  0.0   0.0   0.0   0.0   3880244.0    342980.0     1064120.0    2473144.0    0.0          0.0          0.0          2471556.0
0.0   0.0   0.0   96.8  3.2   0.0   0.0   0.0   3880244.0    343152.0     1063948.0    2473144.0    0.0          0.0          0.0          2471728.0
3.0   3.0   0.0   93.9  0.0   0.0   0.0   0.0   3880244.0    342904.0     1064196.0    2473144.0    0.0          0.0          0.0          2471480.0
```