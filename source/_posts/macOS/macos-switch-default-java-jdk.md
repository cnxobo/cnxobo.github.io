---
title: MacOS 下切换默认 Java (JDK)
tags: []
id: '784'
categories:
  - - Java
  - - macOS
comments: false
date: 2019-10-27 16:22:18
---

### TLDR

如果同时安装了 adoptopenjdk 11 和 adoptopenjdk 8，同时希望 8 做为默认 JDK，只需要把 `/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Info.plist`里的`JVMVersion`的值由`1.8.0_222`改为 `x1.8.0_222`(大概第42行)。这样我们的MacOS默认JDK就成为adoptopenjdk-8了。

### 查看 JDK 相关信息

macOS下 JDK 默认安装在 `/Library/Java/JavaVirtualMachines`目录下，同时提供了一个小工具`/usr/libexec/java_home` 帮助我们快速的查看 JDK 相关的信息。 默认情况下 MacOS 会自动的选择 `/Library/Java/JavaVirtualMachines`目录下版本号最高的 JDK 做为默认 JDK 。

#### 查看当前 JDK 版本

```Shell
➜  ~ java -version
openjdk version "1.8.0_222"
OpenJDK Runtime Environment (AdoptOpenJDK)(build 1.8.0_222-b10)
OpenJDK 64-Bit Server VM (AdoptOpenJDK)(build 25.222-b10, mixed mode)
```

#### 查看当前 JDK 的安装目录

```Shell
➜  ~ /usr/libexec/java_home
/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home
```

#### 查看已安装的 JDK 版本及目录

查看所有

```Shell
➜  ~ /usr/libexec/java_home -V
Matching Java Virtual Machines (3):
    1.8.0_222, x86_64:  "AdoptOpenJDK 8"    /Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home
    1.8.0_201, x86_64:  "Java SE 8" /Library/Java/JavaVirtualMachines/jdk1.8.0_201.jdk/Contents/Home
    1.7.0_80, x86_64:   "Java SE 7" /Library/Java/JavaVirtualMachines/jdk1.7.0_80.jdk/Contents/Home
```

查看指定版本 可以通过`/usr/libexec/java_home -v <version>`来过滤版本号。 返回前缀匹配到的最新 JDK。

```Shell
➜  ~  /usr/libexec/java_home -v 1
/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home
➜  ~  /usr/libexec/java_home -v 1.7
/Library/Java/JavaVirtualMachines/jdk1.7.0_80.jdk/Contents/Home
➜  ~  /usr/libexec/java_home -v 1.8.0_201
/Library/Java/JavaVirtualMachines/jdk1.8.0_201.jdk/Contents/Home
➜  ~  /usr/libexec/java_home -v 1.8
/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home
```

### 切换 JDK

#### 使用指定的 JDK 执行单次命令

可以通过`java_home`的`exec`选项来执行单次任务。 /usr/libexec/java\_home -v version --exec command

```Shell
➜  ~ /usr/libexec/java_home -v 1.7 --exec java -version
java version "1.7.0_80"
Java(TM) SE Runtime Environment (build 1.7.0_80-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.80-b11, mixed mode)
```

#### 切换 Shell 的 JDK到指定版本

Shell 环境只需要指定一下`JAVA_HOME`环境变量就可以。

```Shell
➜  ~ export JAVA_HOME=`/usr/libexec/java_home -v 1.7`
➜  ~ java -version
java version "1.7.0_80"
Java(TM) SE Runtime Environment (build 1.7.0_80-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.80-b11, mixed mode)
```

为了方便切换把以下别名配置粘到对应的 Shell 的配置文件 .bashrc 或 .zshrc，然后就可以方便的切换 JDK 版本了。 **别名配置** 需要根据自己实际已安装 JDK 做增减。

```Shell
alias j12="export JAVA_HOME=`/usr/libexec/java_home -v 12`; java -version"
alias j11="export JAVA_HOME=`/usr/libexec/java_home -v 11`; java -version"
alias j10="export JAVA_HOME=`/usr/libexec/java_home -v 10`; java -version"
alias j9="export JAVA_HOME=`/usr/libexec/java_home -v 9`; java -version"
alias j8="export JAVA_HOME=`/usr/libexec/java_home -v 1.8`; java -version"
alias j7="export JAVA_HOME=`/usr/libexec/java_home -v 1.7`; java -version"
```

**使用效果**

```Shell
➜  ~ j7
java version "1.7.0_80"
Java(TM) SE Runtime Environment (build 1.7.0_80-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.80-b11, mixed mode)
➜  ~ j8
java version "1.8.0_201"
Java(TM) SE Runtime Environment (build 1.8.0_201-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.201-b09, mixed mode)
➜  ~ j11
openjdk version "11.0.4" 2019-07-16
OpenJDK Runtime Environment AdoptOpenJDK (build 11.0.4+11)
OpenJDK 64-Bit Server VM AdoptOpenJDK (build 11.0.4+11, mixed mode)
```

#### 切换 GUI 程序的默认 JDK

GUI 程序使用的默认 Java 也是 `/usr/libexec/java_home -V` 中看到的最高版本。

##### 指定全局环境变量

创建`setenv.javahome.plist`并通过`launchctl`指定设置环境变量`JAVA_HOME`，需要注销再登录才生效。而且有些程序不兼容该方式。使用该方式之后，`/usr/libexec/java_home`的显示跟实际执行也会出现不一致。 生成 `setenv.javahome.plist` 并加载的脚本:

```Shell
cat > ~/Library/LaunchAgents/setenv.javahome.plist <<EOF
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
  <plist version="1.0">
  <dict>
  <key>Label</key>
  <string>setenv.javahome</string>
  <key>ProgramArguments</key>
  <array>
    <string>/bin/launchctl</string>
    <string>setenv</string>
    <string>JAVA_HOME</string>
    <string>/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home</string>
  </array>
  <key>RunAtLoad</key>
  <true/>
  <key>ServiceIPC</key>
  <false/>
</dict>
</plist>
EOF
launchctl load ~/Library/LaunchAgents/setenv.javahome.plist
```

##### 修改 JDK 版本号

我们还可以通过修改版本号实现指定版本的JDK做为默认JDK, 我目前正在使用该方式。 `/usr/libexec/java_home`是通过`/Library/Java/JavaVirtualMachines/<JDK>/Contents/Info.plist`里的`JVMVersion`值来获取版本号的，所以只需要修改这个值为当前最大版本号即可实现指定默认 JDK。经过测试这个还是即时生效。 像我安装过adoptopenjdk 11 ，但还是希望 adoptopenjdk 8做为默认 JDK，只需要把 `/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Info.plist`里的`JVMVersion`的值由`1.8.0_222`改为 `x1.8.0_222`(大概第42行)。这样我们的**adoptopenjdk-8.jdk**就变成最新版本的 JDK 了。

> 排序是通过 ASCII 值来排的，版本号只要改的比最新的 11 大都行,字符'x'的ASCII值远大于字符'1', 为了方便版本区分我只加了一个字符 **x** 。

修改完成之后再查看 JDK 信息，就会发现我们修改的x1.8.0\_222会排到第一位，同时 Java version 是 1.8。

```
➜  ~ /usr/libexec/java_home -V
Matching Java Virtual Machines (4):
    x1.8.0_222, x86_64: "AdoptOpenJDK 8"    /Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home
    11.0.4, x86_64: "AdoptOpenJDK 11"   /Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home
    1.8.0_201, x86_64:  "Java SE 8" /Library/Java/JavaVirtualMachines/jdk1.8.0_201.jdk/Contents/Home
    1.7.0_80, x86_64:   "Java SE 7" /Library/Java/JavaVirtualMachines/jdk1.7.0_80.jdk/Contents/Home

/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home

➜  ~ java -version
openjdk version "1.8.0_222"
OpenJDK Runtime Environment (AdoptOpenJDK)(build 1.8.0_222-b10)
OpenJDK 64-Bit Server VM (AdoptOpenJDK)(build 25.222-b10, mixed mode)
```

**操作流程:** 备份原始文件并打开,并使用 vim 编辑.

```Shell
sudo cp /Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Info.plist /Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Info.plist.bak
sudo vim /Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Info.plist
```

修改后的`Info.plist`文件差异

```Shell
➜  ~ diff -c  /Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Info.plist /Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Info.plist.bak
*** /Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Info.plist    2019-10-27 14:01:10.000000000 +0800
--- /Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Info.plist.bak    2019-10-27 14:17:36.000000000 +0800
***************
*** 39,45 ****
        <key>JVMVendor</key>
        <string>AdoptOpenJDK</string>
        <key>JVMVersion</key>
!       <string>x1.8.0_222</string>
    </dict>
  </dict>
  </plist>
--- 39,45 ----
        <key>JVMVendor</key>
        <string>AdoptOpenJDK</string>
        <key>JVMVersion</key>
!       <string>1.8.0_222</string>
    </dict>
  </dict>
  </plist>
```

### macOS 下 java 的真身

通过简单的探索，就能发现我们使用的 `java` 其实是软链到 `/System/Library/Frameworks/JavaVM.framework/Versions/A/Commands/java`。

```Shell
➜  ~ greadlink -f `which java`
/System/Library/Frameworks/JavaVM.framework/Versions/A/Commands/java
```

> greadlink 需要安装coreutils `brew install coreutils`

而 `/System/Library/Frameworks/JavaVM.framework/Versions/A/Commands` 目录下的文件多是固定 38k 大小。猜测这些文件应该只是包装器，根据系统配置把命令转发给相应的 JDK 的对应命令。

```Shell
➜  ~ ll /System/Library/Frameworks/JavaVM.framework/Versions/A/Commands
total 1200
-rwxr-xr-x  1 root  wheel    38K Sep 30 04:28 appletviewer
-rwxr-xr-x  1 root  wheel    38K Sep 30 04:28 apt
-rwxr-xr-x  1 root  wheel    38K Sep 30 04:28 extcheck
-rwxr-xr-x  1 root  wheel    38K Sep 30 04:28 idlj
-rwxr-xr-x  1 root  wheel    38K Sep 30 04:28 jar
-rwxr-xr-x  1 root  wheel    38K Sep 30 04:28 jarsigner
-rwxr-xr-x  1 root  wheel    38K Sep 30 04:28 java
-rwxr-xr-x  1 root  wheel    47K Sep 30 04:28 java_home
-rwxr-xr-x  1 root  wheel    38K Sep 30 04:28 javac
-rwxr-xr-x  1 root  wheel    38K Sep 30 04:28 javadoc
...
```