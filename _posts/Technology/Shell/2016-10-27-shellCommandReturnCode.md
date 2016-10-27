---

layout: post
title: Shell中if条件判断的一些问题
category: 技术
tags: Shell
keywords: shell 条件判断

---

## 前言

最近工作在做HA（High Available），在写一个Monitor监控服务状态的脚本，因为有一些判断文件或者变量是否为空，从而判断状态的语句，碰到一个很有意思的问题，首先是[ -z ${var} ] 能够靠谱的
判断一个变量或者字符串是否为空的问题，其次就是if COMMANDS; then COMMANDS, 这样的条件判断语句if的判断条件什么时候为真。

## -z能够靠谱的判断变量或者字符串为空的问题

网络上有一种说法，-z不能靠谱的判断，因为在不同的情况下判断的结果不准确，推荐了一种更安全的判断方式，如下：
  if [ "X${VAR}" = "X"]; then COMMANDS
这种判断方式确实很普遍，而且能够有效的避免VAR为空时可能的错误。

但是，用[ -z ${VAR} ]这种方式也能准确的判断变量或者字符串是否为空，只是if条件判断时if成立的判断条件跟普通高级语言不同罢了，在下面会详细介绍。

## if COMMANDS; then COMMANDS 判断条件

首先，shell中if判断成立的条件是exit code 是0， 否则都会执行then后面的语句。所以我们如果有一个函数，函数中会return一个状态值，那么怎么去准确的判断函数返回值呢？
先看下面一段程序：

` \#!/bin/bash `

`APACHE_NAME=httpd`

`isApacheRunning(){ `

    `` PROC=`/usr/bin/pgrep -x "$APACHE_NAME"` ``
    if [ -z "$PROC" ] ; then
        return 1 #false

    else
        return 0 #true
    fi
}

`hello(){`
`if isApacheRunning`
`then`
`  echo "$APACHE_NAME already running"`
`fi`
`}`

`echo "start"`
`hello`
`echo "end"`

在isApacheRunning里，判断httpd进程存在，return 0表示true,return 1表示false。在if isApacheRunning判断中，当isApacheRunning返回值是0是，执行then后面的语句。
所以，在shell中，用0表示true,1表示false。

stackoverflow上有两篇详细的讲解了这一点：


[bash functions: return boolean to be used in if](http://stackoverflow.com/questions/5431909/bash-functions-return-boolean-to-be-used-in-if)

[What is proper way to test a bash function return value?](http://stackoverflow.com/questions/6241256/what-is-proper-way-to-test-a-bash-function-return-value)

### 另一个例子

if COMMANDS;then COMMANDS这种形式是我们最常遇到的判断语句，其中COMMANDS既可以代表函数调用，又可以代表可执行命令，上面讲到了函数调用，下面例子是执行命令。

例如：
if ps -p ${pid} |grep -v PID > /dev/null; then
    apacheRet=0 #执行成功返回值是0
else
    apacheRet=1
fi

命令执行成功，返回值为0

[root@adf-cr bin]# ps -p 871 | grep -v PID
  871 ?        00:00:00 cr-wrapper
[root@adf-cr bin]# echo $?
0

命令执行失败，返回值为1

[root@adf-cr bin]# ps -p 32643 | grep -v PID > /dev/null
[root@adf-cr bin]# echo $?
1

## 总结

shell 函数中如果判断结果是真，return 0, 如果是假，return 1。 在if COMMANDS;then COMMANDS 条件判断语句中，COMMANDS 的exits code为0 判断为真，执行then后面语句；为1判断为假,
执行else语句。