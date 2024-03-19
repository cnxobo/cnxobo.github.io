---
title: Java/Shell程序封装为MacOS程序 Application
tags:
  - Automator
  - java
  - macOS
id: "980"
categories:
  - - macOS
comments: false
date: 2022-08-26 20:04:36
---

macOS 下有些带 GUI 的程序并没有按照 macOS 的规范去打包成一个 MacOS 的 Application，导致启动它的体验不太好，虽然不影响使用。 这里以`Jadx`\-- 一个 Java 反编译工具, 为例演示如何把一个 Java 程序包装为 MacOS 程序 Application。

<!--more-->

> 通过`brew`安装过`jadx`之后，可以通过执行`jadx-gui`命令来启动他的图形界面。

## 做个图标

做为一个应用程序，最早呈现给用户的就是图标。macOS 使用 icns 格式的图标资源。图片格式及大小没具体要求，长宽比例必须是`1:1`的。 \* 可以通过 cloudconvert.com 在线转换 [https://cloudconvert.com/png-to-icns](https://cloudconvert.com/png-to-icns) \* 可以通过`makeicns`线下转换

```Shell
brew install makeicns
makeicns -in myfile.png -out outfile.icns
```

## 改进 Java 显示效果

Java 的 GUI 启动之后，在 dock 里还是显示 JRE 的默认图标，如果需要额外定制图标就需要通过其他参数指定。 Java 为 macOS 提供两个专门的参数来定制在 macOS 的表现(在 macOS 上可以通过`java -X` 命令查看参数描述)。

```
-Xdock:name=<application name>
                  override default application name displayed in dock
-Xdock:icon=<path to icon file>
                  override default icon displayed in dock
```

\-Xdock:name 用来指定在菜单栏里显示的名字。在 macOS 早期的版本是可以定制在 dock 中显示的名字。现在不行了。 -Xdock:icon= 用来指定在 dock 中显示的图标。 jadx-gui 支持通过`JADX_GUI_OPTS`变量给他传参数，其他 Java 程序按自己实际情况调整。

## Automator

Automator 是 macOS 自带的自动化工具。 1. 启动 Automator 并创建一个 `Application` 类型的文件。 2. 添加一个`Run Shell Script`的 action。`Pass input`改为`as arguments`并在下方的文本框内填入如下命令

```Shell
JADX_GUI_OPTS="-Xdock:icon=/Applications/Jadx.app/Contents/Resources/ApplicationStub.icns -Xdock:name=Jadx" /usr/local/bin/jadx-gui "$@"
```

3.  点击文件-保存。名字为`Jadx.app`, 位置选`/Application`,文件类型为`Application` ![](https://cdn.jsdelivr.net/gh/cnxobo/imghost/imgautomator-application.png)
4.  把定制的图标重命名为`ApplicationStub.icns`，然后覆盖位于`/Application/Jadx.app/Contents/Resources/ApplicationStub.icns`的 Automator 默认图标。 > 在 Xxx.app 上点右键，选择 `显示包内容(Show Package Contents)`,就可以以文件夹的方式打开 Xxx.app 了。

## universalJavaApplicationStub

使用  Automator 包装的程序制作简单，唯独在 dock 中显示的名字不能修改，略有遗憾。参考了`JD-GUI`的实现，我发现了[universalJavaApplicationStub](https://github.com/tofi86/universalJavaApplicationStub)。

### miniBundleApp

最小可运行的 macOS 程序结构大概是这个样子。 \* Info.plist 存储程序相关的属性配置信息。 \* MacOS 目录存放可执行程序 \* Resources 存放资源文件, 示例中放了图标资源。而像 JD-GUI 就把 jar 包放在`Resources/Java`目录下。

```
Demo.app
└── Contents
    ├── Info.plist
    ├── MacOS
    │   └── universalJavaApplicationStub
    └── Resources
        └── ApplicationStub.icns
```

1.  创建目录结构。

```Shell
mkdir -p /Applications/Jadx-gui.app/Contents/{MacOS,Resources}
curl https://raw.githubusercontent.com/tofi86/universalJavaApplicationStub/master/src/universalJavaApplicationStub -o /Applications/Jadx-gui.app/Contents/MacOS/universalJavaApplicationStub
chmod +x /Applications/Jadx-gui.app/Contents/MacOS/universalJavaApplicationStub
```

2.  配置 Info.plist Info.plist 位于 `Demo.app/Contents/Info.plist`。 主要关系几个属性：CFBundleName、CFBundleIdentifier、MainClass、ClassPath。

\* CFBundleName 程序名称 \* CFBundleIdentifier 程序唯一标识 \* MainClass Java 的启动 class \* ClassPath Java 的 classpath 路径。要注意的是如果要配置目录时，`lib/*` 需要对`*`做转义`lib/\\*`。

```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
    <dict>
        <key>CFBundleExecutable</key>               <string>universalJavaApplicationStub</string>
        <key>CFBundleName</key>                     <string>Jadx</string>
        <key>CFBundleGetInfoString</key>            <string>Jadx for macOS</string>
        <key>CFBundleIconFile</key>                 <string>ApplicationStub.icns</string>
        <key>CFBundleIdentifier</key>               <string>org.xobo.Jadx</string>
        <key>CFBundleShortVersionString</key>       <string>1.0.0</string>
        <key>NSHumanReadableCopyright</key>         <string>Copyright 2022 xobo</string>


        <key>JavaX</key>
        <dict>
            <key>MainClass</key>                    <string>jadx.gui.JadxGUI</string>
            <key>JVMVersion</key>                   <string>1.8+</string>
            <key>ClassPath</key>                    <string>/usr/local/Cellar/jadx/1.4.4/libexec/lib/\\*</string>
            <key>WorkingDirectory</key>             <string>$APP_ROOT</string>
            <key>Properties</key>
            <dict>
                <key>apple.laf.useScreenMenuBar</key>
                <string>true</string>
            </dict>
        </dict>

        <key>CFBundlePackageType</key>              <string>APPL</string>
        <key>CSResourcesFileMapped</key>            <true/>
        <key>LSRequiresCarbon</key>                 <true/>
        <key>NSPrincipalClass</key>                 <string>NSApplication</string>
        <key>CFBundleInfoDictionaryVersion</key>    <string>6.0</string>
        <key>NSHighResolutionCapable</key>          <true/>

        <key>NSAppleEventsUsageDescription</key>
        <string>There was an error while launching the application. Please click OK to display a dialog with more information or cancel and view the syslog for details.</string>
    </dict>
</plist>
```

3.  跟 Automator 一样替换图标文件即可。

### 封装依赖

之前做的这些更准确的说是**包装**而不是**封装**，因为我们的程序极度的依赖外部的文件，这个 Application 打包给别人并不能做为独立应用直接使用。如果希望能做为独立应用还需要做以下工作：

#### 程序内添加应用依赖

> 本文整体都是以 Apple 风格的 Java 程序，如果希望以 Oracle 风格构建 Application 可以查看`universalJavaApplicationStub`文档及源码。

在`Resources`目录下创建`Java`目录。`Demo.app/Contents/Resources/Java`，然后把 jadx 所有依赖的 jar 包复制到这个目录下。 然后配置 classpath 为`$JAVAROOT/\\*`。

#### 打包 JRE

> 如果需要把 JRE 一起打包进来用 jlink(jdk9+) 或 jpackage(jdk8)更合适。 `universalJavaApplicationStub`的 JAVA_HOME 支持相对路径。在 Info.plist 内配置`LSEnvironment`并复制 JRE 到对应位置就可以了。

```XML
<key>LSEnvironment</key>
<dict>
    <key>JAVA_HOME</key>
    <string>Contents/Frameworks/jdk8u232-b09-jre/Contents/Home</string>
<dict>
```

### macOS 10.15+ 可能会遇到的问题

从 macOS 10.15 开始，macOS 默认会阻止对受保护资源的访问，如用户的下载、文档或桌面文件夹，并显示一个安全对话框，用户必须接受该对话框才能被允许访问。 当在你的应用程序中使用 javax.swing.JFileChooser 时，它支持这些类型的安全对话框（有趣的是 java.awt.FileDialog 不支持！），你应该使用 universalJavaApplicationStub 脚本的编译[二进制文件](https://github.com/tofi86/universalJavaApplicationStub/releases/ "二进制文件")而不是普通的 bash 脚本。

> 二进制文件有两个不同的版本，根据自己的系统选择。 universalJavaApplicationStub-xxx-binary-macos-10.15.zip 的可能可以在 macOS 11 上使用， universalJavaApplicationStub-xxx-binary-macos-11.0.zip 肯定不能在 10.15 的系统上使用。

## 打开方式 Open With

如果希望在 class、jar、java、zip 等文件上点右键能出现该程序，只需要在 Info.plist 里添加如下代码：

```XML
<key>CFBundleDocumentTypes</key>
<array>
  <dict>
    <key>CFBundleTypeExtensions</key>
    <array>
      <string>class</string>
    </array>
    <key>CFBundleTypeMIMETypes</key>
    <array>
      <string>application/java</string>
    </array>
    <key>CFBundleTypeRole</key>
    <string>Viewer</string>
    <key>CFBundleTypeName</key>
    <string>Class File</string>
    <key>LSIsAppleDefaultForType</key>
    <true/>
    <key>LSTypeIsPackage</key>
    <false/>
  </dict>
  <dict>
    <key>CFBundleTypeExtensions</key>
    <array>
      <string>java</string>
    </array>
    <key>CFBundleTypeMIMETypes</key>
    <array>
      <string>text/plain</string>
    </array>
    <key>CFBundleTypeRole</key>
    <string>Viewer</string>
    <key>CFBundleTypeName</key>
    <string>Java File</string>
    <key>LSIsAppleDefaultForType</key>
    <false/>
    <key>LSTypeIsPackage</key>
    <false/>
  </dict>
  <dict>
    <key>CFBundleTypeExtensions</key>
    <array>
      <string>jar</string>
    </array>
    <key>CFBundleTypeMIMETypes</key>
    <array>
      <string>application/java-archive</string>
    </array>
    <key>CFBundleTypeName</key>
    <string>Jar File</string>
    <key>CFBundleTypeRole</key>
    <string>Viewer</string>
    <key>LSIsAppleDefaultForType</key>
    <false/>
    <key>LSTypeIsPackage</key>
    <false/>
  </dict>
  <dict>
    <key>CFBundleTypeExtensions</key>
    <array>
      <string>war</string>
    </array>
    <key>CFBundleTypeMIMETypes</key>
    <array>
      <string>application/java-archive</string>
    </array>
    <key>CFBundleTypeName</key>
    <string>War File</string>
    <key>CFBundleTypeRole</key>
    <string>Viewer</string>
    <key>LSIsAppleDefaultForType</key>
    <false/>
    <key>LSTypeIsPackage</key>
    <false/>
  </dict>
  <dict>
    <key>CFBundleTypeExtensions</key>
    <array>
      <string>ear</string>
    </array>
    <key>CFBundleTypeMIMETypes</key>
    <array>
      <string>application/java-archive</string>
    </array>
    <key>CFBundleTypeName</key>
    <string>Ear File</string>
    <key>CFBundleTypeRole</key>
    <string>Viewer</string>
    <key>LSIsAppleDefaultForType</key>
    <false/>
    <key>LSTypeIsPackage</key>
    <false/>
  </dict>
  <dict>
    <key>CFBundleTypeExtensions</key>
    <array>
      <string>aar</string>
    </array>
    <key>CFBundleTypeName</key>
    <string>Android archive File</string>
    <key>CFBundleTypeRole</key>
    <string>Viewer</string>
    <key>LSIsAppleDefaultForType</key>
    <false/>
    <key>LSTypeIsPackage</key>
    <false/>
  </dict>
  <dict>
    <key>CFBundleTypeExtensions</key>
    <array>
      <string>jmod</string>
    </array>
    <key>CFBundleTypeMIMETypes</key>
    <array>
      <string>application/java-archive</string>
    </array>
    <key>CFBundleTypeName</key>
    <string>Java module File</string>
    <key>CFBundleTypeRole</key>
    <string>Viewer</string>
    <key>LSIsAppleDefaultForType</key>
    <false/>
    <key>LSTypeIsPackage</key>
    <false/>
  </dict>
  <dict>
    <key>CFBundleTypeExtensions</key>
    <array>
      <string>zip</string>
    </array>
    <key>CFBundleTypeMIMETypes</key>
    <array>
      <string>application/zip</string>
    </array>
    <key>CFBundleTypeName</key>
    <string>Zip File</string>
    <key>CFBundleTypeRole</key>
    <string>Viewer</string>
    <key>LSIsAppleDefaultForType</key>
    <false/>
    <key>LSTypeIsPackage</key>
    <false/>
  </dict>
</array>
```

universalJavaApplicationStub-Info.plist https://gist.github.com/cnxobo/e53e0571c5165913e68c8db3600c7d29
