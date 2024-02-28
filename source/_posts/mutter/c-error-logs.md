---
title: c++ error logs
tags: []
id: '103'
categories:
  - - Something
date: 2011-06-07 15:26:29
---

**启用-Wall** 帮忙解决太多bug.

alias g++='g++ -Wall'

写入配置文件

echo "alias g++='g++ -Wall'" >> ~/.bashrc && source ~/.bashrc

以下代码皆为错误代码 **括号,万恶的括号！** accept 返回值是0

if ((fd = accept(Sockfd, Addr, (socklen\_t\*) AddrLen) < 0)) {
}

**函数没有返回值！** string产生的Segmentation fault

string testString(string &a){
    a = "I'm string";
}

**循环** 栈溢出

char \*lists\[maxToken\];
lists\[0\] = strtok(buffer, white);
int i = 1;
while ((lists\[i\] = strtok(NULL, white)) != NULL && i < maxToken) {
    i++;
}