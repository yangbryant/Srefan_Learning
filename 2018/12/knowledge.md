## 二.git bisect 二分法定位错误

* 设置代码提交范围, 终点 `=` **最近提交**, 启动 `=` **很久之前没有问题的版本**
* 执行命令后, 代码就会切换到设定范围正中间的那次提交

```shell
// git bisect start [终点] [起点]
git bisect start HEAD 4d83cf
```

1. 验证如果此版本没有问题, 意味着错误是后半段代码引入的, 设置为 `good`, 自动切换到后半段的中段.

```shell
git bisect good
```

2. 验证如果此版本有问题, 意味着错误是前半段代码引入的, 设置为 `bad`, 自动切换到前半段的中段.

```shell
git bisect bad
```

* 不断重复这个过程, 直到找到问题提交. 给如下提示.

```shell
3a97bd is the first bad commit
```

* 找到问题, 检查代码, 确定具体的错误.
* 然后使用 `reset` 命令, 退出查错, 回到最近一次提交.

```shell
git bisect reset
```

## 一.awk

#### 基本用法

```shell
awk '{print $0}' demo.txt
```

> `demo.txt` 是 `awk` 所要处理的文本文件. 
> 前面单引号内部有一个大括号, 里面就是每一行的处理动作 `print $0`. 
> 其中, `print` 是打印命令, `$0` 代表当前行.
> 因此上面命令的执行结果, 就是把每一行原样打印出来.

* `$0` **代表当前行**
*  `$1`, `$2`, `$3` **代表第一个字段、第二个字段、第三个字段等等**

```shell
awk -F ':' '{ print $1 }' demo.txt
```

> `-F` 参数 **指定分隔符**. 这个文件的分隔符是冒号(`:`).

#### 变量

* `$ + 数字` 标识某个字段
* `NF` 表示当前行有多少个字段, 因此 `$NF` 代表最后一个字段, `$(NF-1)`代表倒数第二个字段.
* `NR` 表示当前行是第几行

```shell
$ awk -F ':' '{print $1, $(NF-1)}' demo.txt
root /root
daemon /usr/sbin
bin /bin
sys /dev
sync /bin

$ awk -F ':' '{print NR ") " $1}' demo.txt
1) root
2) daemon
3) bin
4) sys
5) sync
```

> **注意**: `print` 命令如果原样输出字符,要放在双引号(`""`)里面.

* `FILENAME`: 当前文件名
* `FS`: 字段分隔符,默认是空格和制表符
* `RS`: 行分隔符,用于分割每一行,默认是换行符
* `OFS`: 输出字段的分隔符,用于打印时分割字段,默认是空格
* `ORS`: 输出记录的分隔符,用于打印时分隔记录,默认是换行符
* `OFMT`: 数字输出格式, 默认是`%.6g`

#### 函数

`awk` 内置函数, 方便处理原始数据, [查看手册点击这里.](https://www.gnu.org/software/gawk/manual/html_node/Built_002din.html#Built_002din).

* `toupper()` 字符转为大写.

```shell
awk -F: '{ print toupper($1) }' demo.txt
ROOT
DAEMON
BIN
SYS
SYNC
```

* `tolower()` 字符转为小写.
* `length()` 返回字符串长度.
* `substr()` 返回子字符串.
* `sin()` 正弦.
* `cos()` 余弦.
* `sqrt()` 平方根.
* `rand()` 随机数.

#### 条件

`awk` 允许指定输出条件, 只输出符合条件的行, **输出条件写在动作之前**.

```shell
awk -F: '/usr/ {print $1}' demo.txt
```

* `print` 命令前面是正则表达式, 只输出包含 `usr` 的行.

```shell
// 只输出奇数行
awk -F: 'NR % 2 == 1 {print $1}' demo.txt

// 输出第三行以后的行
awk -F: 'NR > 3 {print $1}' demo.txt

// 输出第一个字段等于指定值的行
awk -F ':' '$1 == "root" || $1 == "bin" {print $1}' demo.txt
```

#### if 语句

`awk` 提供了 `if` 结构, 用于编写复杂的条件.

```shell
// 输出第一个字段的第一个字符大于m的行
awk -F: '{if ($1 > "m") print $1}' demo.txt
```

`if` 结构还可以指定 `else` 部分.

```shell
awk -F ':' '{if ($1 > "m") print $1; else print "---"}' demo.txt
```

#### 参考链接

[阮一峰的网络日志](http://www.ruanyifeng.com/blog/2018/11/awk.html)