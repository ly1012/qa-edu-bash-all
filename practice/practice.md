
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
```