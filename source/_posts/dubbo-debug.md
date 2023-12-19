---
title: dubbo 接口调试
tags:
  - dubbo
  - java
  - python
  - telnet
id: '586'
categories:
  - - Java
comments: false
date: 2017-07-17 00:56:17
---

公司内部有多个项目中并使用 dubbo 相互间提供服务，每次相关接口调试与联调都是一种折磨，在有多个服务提供方时，不能确定这次调用由哪个提供方处理。在长期的摸索中找到如下调试方法:

## 1\. 封装为 HTTP 接口

把接口封装为 HTTP 服务，就变成了 HTTP 接口的调试； **优点:** 该方法实现相对简单，也是我调试自己写的接口最常用的方法。 **缺点:** 1. 每个接口都要实现一个 HTTP 接口(可以写个工具自动把 dubbo 接口转化为 HTTP 接口的工具)； 2. HTTP 接口不能完整的模拟 dubbo 接口的调用环境。做为服务提供方，代码在执行时是没有 HttpRequest/HttpResponse 的。

## 2\. 构建相对独立 dubbo 的环境

dubbo 联调时，最头疼的是经常会有别的服务提供方把调用处理掉了，所以要构建一个可以指定服务提供方的环境。 可以新启一个注册中心(zookeeper)，跟小伙伴都连上这个注册中心，在这个二人世界里开心的联调。 dubbo 的开发者也考虑到调试的问题提供了相应的配置

### 2.1 文件映射

推荐使用[文件映射](https://xobo.org/dubbo-debug-direct-provider/#i)的方式.

### 2.2 修改注册方式

节点

属性

类型

描述

\\<dubbo:dubbo:registry>

register

boolean

是否向此注册中心注册服务，如果设为false，将只订阅，不注册。开发环境建议设置为 false

\\<dubbo:reference>

url

string

点对点直连服务提供者地址，将绕过注册中心。

示例

```XML
<!-- 不提供服务 -->
<dubbo:registry address="zookeeper://${zookeeper}" register="false"/>

<!-- 直连配置的提供者 --> 
<dubbo:reference id="xxxService" interface="com.alibaba.xxx.XxxService" url="dubbo://localhost:20890" />
```

## 3\. 使用 telnet 调试接口

偶然间发现 dubbo 可以通过 telnet 调试，感觉圣光在照耀着我。 `invoke XxxService.xxxMethod({"prop": "value"})`

```Shell
> telnet localhost 20880
dubbo> invoke org.xobo.say("xobo", {"welcome": "hello world!"})
```

## 4\. 中文乱码

通过 telnet 会发现 dubbo 序列化数据默认编码是 GBK。 可以通过设置或节点的charset属性来修改默认编码。 macOS 自带的 telnet 对中文的输入和输出都不友好，又写了一个简单的 Python 脚本来代替默认的 telnet 客户端。

## 5\. No such method

参数是对象的时候，会报居然报错：No such method xx in xxx, 需要添加一个class属性来指定参数类型。

```Shell
invoke org.xobo.say({"class":"org.xobo.HelloDTO", "welcome": "hello world!"})
```

### 简易 Telnet 客户端

```Python
#!/usr/local/bin/python3
"""用于替代 macOS 下的 telnet 以方便调试 dubbo 接口
Example:
    ./telent_dubbo.py localhost 20880

"""

import sys
import telnetlib
import platform
if platform.system() != "Windows":
    try:
        import readline
    except ImportError:
        print("May need install package readline")

host = sys.argv[1]
port = sys.argv[2]
encoding = "GBK"

if len(sys.argv)==4:
    encoding = sys.argv[3]


tn = telnetlib.Telnet(host, port)
cmd = None
def remove_last_line_from_string(s):
    return s[:s.rfind('\n')]
while cmd != "exit":
    cmd = input("> ").strip()
    tn.write(cmd.encode(encoding) + b"\n")
    if cmd != "exit":
        result = tn.read_until(b"dubbo>").decode(encoding)
        result = remove_last_line_from_string(result);
        print(result)
```

### Telnet命令参考手册

(+) (#) Dubbo2.0.5以上版本服务提供端口支持telnet命令， 使用如： `telnet localhost 20880` 或者： `echo status nc -i 1 localhost 20880` telnet命令可以扩展，参见：扩展参考手册第6条。 status命令所检查的资源也可以扩展，参见：扩展参考手册第5条。

#### ls

(list services and methods) `ls` 显示服务列表。 `ls -l` 显示服务详细信息列表。 `ls XxxService` 显示服务的方法列表。 `ls -l XxxService` 显示服务的方法详细信息列表。

#### ps

(print server ports and connections) `ps` 显示服务端口列表。 `ps -l` 显示服务地址列表。 `ps 20880` 显示端口上的连接信息。 `ps -l 20880` 显示端口上的连接详细信息。

#### cd

(change default service) `cd XxxService` 改变缺省服务，当设置了缺省服务，凡是需要输入服务名作为参数的命令，都可以省略服务参数。 `cd /` 取消缺省服务。

#### pwd

(print working default service) `pwd` 显示当前缺省服务。

#### trace

`trace XxxService` 跟踪1次服务任意方法的调用情况。 `trace XxxService 10` 跟踪10次服务任意方法的调用情况。 `trace XxxService xxxMethod` 跟踪1次服务方法的调用情况 `trace XxxService xxxMethod 10` 跟踪10次服务方法的调用情况。

#### count

`count XxxService` 统计1次服务任意方法的调用情况。 `count XxxService 10` 统计10次服务任意方法的调用情况。 `count XxxService xxxMethod` 统计1次服务方法的调用情况。 `count XxxService xxxMethod 10` 统计10次服务方法的调用情况。

#### invoke

`invoke XxxService.xxxMethod({"prop": "value"})` 调用服务的方法。 `invoke xxxMethod({"prop": "value"})` 调用服务的方法(自动查找包含此方法的服务)。

#### status

`status` 显示汇总状态，该状态将汇总所有资源的状态，当全部OK时则显示OK，只要有一个ERROR则显示ERROR，只要有一个WARN则显示WARN。 `status -l` 显示状态列表。

#### log

2.0.6以上版本支持 `log debug` 修改dubbo logger的日志级别 `log 100` 查看file logger的最后100字符的日志

#### help

`help` 显示telnet命帮助信息。 `help xxx` 显示xxx命令的详细帮助信息。

#### clear

`clear` 清除屏幕上的内容。 `clear 100` 清除屏幕上的指定行数的内容。

#### exit

`exit` 退出当前telnet命令行。