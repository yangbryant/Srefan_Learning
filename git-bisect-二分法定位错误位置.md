## git bisect 二分法定位错误

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
