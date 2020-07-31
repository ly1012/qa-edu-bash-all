
# 代码片段

## 单行多行互换

**使用 sed 命令**

> 优点：简洁。

```bash
# 单行变多行
# `s/regexp/replacement/` 修改，将每行中第一个 regexp 匹配项替换为 replacement。
# `g` Copy hold space to pattern space。一般配合 s/regexp/replacement/ 使用，实现行内全局替换。
# 综合解释：
# 1. 读取一行到模式空间，替换分号为换行符，输出模式空间数据
# 2. 重复上一步，直到读取到 EOF。
# 3. 总共输出了 3 次模式空间数据。
:/test/practice$ sed 's/:/\n/g' <<< "$PATH" | head -n3
/usr/local/sbin
/usr/local/bin
/usr/sbin
# 多行变单行
# `:label` 标签（为了给 t 或 b 使用） 。
# `N` 追加下一行到模式空间。
# `t label` 跳转到 label 位置。
# 综合解释：
# 1. 读取第一行到模式空间。
# 2. [标签位置]追加下一行到模式空间，现在模式空间为原来的行+刚追加的行，替换模式空间中的换行符为分号
# 3. 跳转到标签位置继续，直到读取到 EOF，输出模式空间数据。
# 4. 总共输出了 1 次模式空间数据。
:/test/practice$ sed 's/:/\n/g' <<< "$PATH" | head -n3 | sed ":label;N;s/\n/:/g;t label"
/usr/local/sbin:/usr/local/bin:/usr/sbin
```

**使用 awk 命令**

> 优点：简单，容易理解。

- `RS` records seperate，记录分隔符。
- `ORS` output records seperate，记录输出分隔符。

```bash
# 单行变多行
:/test/practice$ echo $PATH | awk 'BEGIN {RS=":"} {print $0}' | head -n 3
/usr/local/sbin
/usr/local/bin
/usr/sbin
# 多行变单行
:/test/practice$ echo $PATH | awk 'BEGIN {RS=":"} {print $0}' | head -n 3 | awk 'BEGIN {ORS=":"} {print $0} '
/usr/local/sbin:/usr/local/bin:/usr/sbin:
```

## 大小写转换

**变量扩展**

```bash
# 全部转为小写
[root@izj6c66dst6ergsqe2wynxz ~]# k='Python Java Bash'; echo ${k,,}
python java bash
[root@izj6c66dst6ergsqe2wynxz ~]# echo 'Python Java Bash' | while read k; do echo ${k,,}; done
python java bash
# 全部转为大写
[root@izj6c66dst6ergsqe2wynxz ~]# k='Python Java Bash'; echo ${k^^}
PYTHON JAVA BASH
[root@izj6c66dst6ergsqe2wynxz ~]# echo 'Python Java Bash' | while read k; do echo ${k^^}; done
PYTHON JAVA BASH
```

**tr 命令**

```bash
# 全部转为小写
$ echo 'Python Java Bash' | tr 'A-Z' 'a-z'
python java bash
# 全部转为大写
$ echo 'Python Java Bash' | tr 'a-z' 'A-Z'
PYTHON JAVA BASH
```

**awk 命令**

```bash
# 全部转为小写
$ echo 'Python Java Bash' | awk '{print tolower($0)}'
python java bash
# 全部转为大写
$ echo 'Python Java Bash' | awk '{print toupper($0)}'
PYTHON JAVA BASH
```

**sed 命令**

```bash
# 全部转为小写
$ echo 'Python Java Bash' | sed 'y/ABCDEFGHIJKLMNOPQRSTUVWXYZ/abcdefghijklmnopqrstuvwxyz/'
python java bash
# 全部转为大写
$ echo 'Python Java Bash' | sed 'y/abcdefghijklmnopqrstuvwxyz/ABCDEFGHIJKLMNOPQRSTUVWXYZ/'
PYTHON JAVA BASH
```

