---
title: 拉取老版本Docker镜像报unsupported manifest解决方案
tags:
  - docker
  - skopeo
categories:
  - - ops
date: 2026-03-31 18:00:00
comments: false
---

## 0x01 问题描述

需要从私有仓库拉取一个老版本的 Docker 镜像，直接 `docker pull` 报错：

```Shell
docker pull hub.shuyun.com/newbi4/app:4.8.16-420260330170921
```

```
Error response from daemon: unsupported manifest media type and no default available: application/json
```

镜像的 manifest 格式是 `application/json`，而新版 Docker (25+) 只接受标准的 `application/vnd.docker.distribution.manifest.v2+json` 等格式，直接拒绝了。

## 0x02 原因分析

这个问题通常出现在以下场景：

1. 镜像是用较老版本的 Docker 或非标准工具推送的，manifest 格式不符合当前 Docker 的要求
2. 新版 Docker 对 manifest media type 的校验更严格，不再兼容非标准格式

本质上是客户端与 registry 之间的格式协商失败。

## 0x03 解决方案：使用 skopeo

[skopeo](https://github.com/containers/skopeo) 是一个专门用于镜像搬运的工具，对各种 manifest 格式兼容性很强，可以在不同存储之间复制镜像。

### 3.1 安装 skopeo

skopeo 不支持 Windows 原生安装，在 WSL 中安装即可：

```Shell
wsl
sudo apt update && sudo apt install skopeo
```

### 3.2 登录私有仓库

skopeo 可以直接读取 Docker 的登录凭证，如果已经在 WSL 中 `docker login` 过就不需要再登录。否则手动登录：

```Shell
skopeo login hub.shuyun.com -u docker -p docker
```

### 3.3 拉取镜像

先保存为 tar 文件，再用 `docker load` 导入：

```Shell
skopeo copy --src-tls-verify=false docker://hub.shuyun.com/newbi4/app:4.8.16-420260330170921 docker-archive:/tmp/app.tar
```

然后导入 Docker：

```Shell
docker load -i /tmp/app.tar
```

`docker load` 只是解压导入 tar 包中的镜像层，不走 registry API 协商，所以不受 manifest 格式限制。

> 注意：不要用 `docker-daemon:` 作为目标，WSL 中的 skopeo 版本可能因为 Docker Engine API 版本不匹配而报错：
> `client version 1.22 is too old. Minimum supported API version is 1.44`
> 走 `docker-archive` + `docker load` 是最稳的方案。

## 0x04 过程中的其他坑

### 4.1 WSL 中 docker login 报 credential-desktop.exe 错误

在 WSL 中执行 `docker login` 时可能报错：

```
error saving credentials: error storing credentials - err: fork/exec /usr/bin/docker-credential-desktop.exe: exec format error
```

这是因为 WSL 继承了 Windows 侧 Docker 的 `~/.docker/config.json`，其中 `credsStore` 指向了 Windows 的凭据管理器。

解决方法是把 `credsStore` 置空：

```Shell
sed -i 's/"credsStore": "desktop"/"credsStore": ""/' ~/.docker/config.json
```

之后重新 `docker login` 即可，凭证会以 base64 明文存在 config.json 中。

### 4.2 Windows 凭据管理器中找回 Docker 密码

如果之前在 Windows 侧 `docker login` 过但忘记了密码，凭证存在 Windows 凭据管理器中（`credsStore: "desktop"`）。

控制面板的凭据管理器界面可以看到条目但密码可能无法直接显示。可以通过 PowerShell 调用 Win32 API 读取：

```PowerShell
Add-Type -Namespace Win32 -Name Credman -MemberDefinition @'
[DllImport("advapi32.dll",CharSet=CharSet.Unicode,SetLastError=true)]
public static extern bool CredRead(string target,int type,int flags,out IntPtr cred);

[StructLayout(LayoutKind.Sequential,CharSet=CharSet.Unicode)]
public struct CREDENTIAL{
    public int Flags;
    public int Type;
    public string TargetName;
    public string Comment;
    public long LastWritten;
    public int CredentialBlobSize;
    public IntPtr CredentialBlob;
    public int Persist;
    public int AttributeCount;
    public IntPtr Attributes;
    public string TargetAlias;
    public string UserName;
}
'@

$ptr = [IntPtr]::Zero
[Win32.Credman]::CredRead('hub.shuyun.com',1,0,[ref]$ptr)
$cred = [Runtime.InteropServices.Marshal]::PtrToStructure($ptr,[type][Win32.Credman+CREDENTIAL])
$bytes = New-Object byte[] $cred.CredentialBlobSize
[Runtime.InteropServices.Marshal]::Copy($cred.CredentialBlob, $bytes, 0, $cred.CredentialBlobSize)
$pass = [Text.Encoding]::UTF8.GetString($bytes)
Write-Host "User:" $cred.UserName
Write-Host "Pass:" $pass
```

将以上内容保存为 `.ps1` 文件执行，即可拿到用户名和密码。

## 0x05 不启动容器提取镜像内文件

如果只是想从镜像中提取某个文件，不需要创建或启动容器，直接解压 tar 包即可。

Docker 镜像本质上是分层的 tar 包，`docker load` 用的 tar 文件里包含了每一层的 `layer.tar`。

### 5.1 解压镜像 tar

```Shell
mkdir -p /tmp/app_extracted
cd /tmp/app_extracted
tar -xf /tmp/app.tar
```

### 5.2 找到目标层

解压后会看到多个 `*.tar` 文件，每个对应镜像的一层。按大小排序找到应用层：

```Shell
ls -lhS /tmp/app_extracted/*.tar
```

### 5.3 查看层内容并提取

```Shell
# 列出层内文件
tar -tf /tmp/app_extracted/<最大的那个>.tar

# 提取指定文件，比如 WAR 包
tar -xf /tmp/app_extracted/<最大的那个>.tar -C /tmp/ var/lib/jetty/webapps/ROOT.war
```

文件会被解压到 `/tmp/var/lib/jetty/webapps/ROOT.war`。

> 如果是在 Windows 侧操作，可以通过 UNC 路径访问 WSL 中的文件：
> `\\wsl.localhost\Ubuntu-22.04\tmp\app_extracted\`

## 0x06 完整操作步骤

汇总一下从零开始的完整流程：

```Shell
# 1. 进入 WSL
wsl

# 2. 安装 skopeo
sudo apt update && sudo apt install skopeo

# 3. 修复 credential 问题
sed -i 's/"credsStore": "desktop"/"credsStore": ""/' ~/.docker/config.json

# 4. 登录私有仓库
skopeo login hub.shuyun.com -u <username> -p <password>

# 5. 拉取镜像到 tar
skopeo copy --src-tls-verify=false docker://hub.shuyun.com/newbi4/app:4.8.16-420260330170921 docker-archive:/tmp/app.tar

# 6. 导入 Docker
docker load -i /tmp/app.tar

# 7. 验证
docker images | grep newbi4/app

# 8. (可选) 不建容器直接提取文件
mkdir -p /tmp/app_extracted && tar -xf /tmp/app.tar -C /tmp/app_extracted
ls -lhS /tmp/app_extracted/*.tar
tar -xf /tmp/app_extracted/<目标层>.tar -C /tmp/ <目标文件路径>
```

## 0x07 总结

新版 Docker 对 manifest 格式校验越来越严格，遇到老镜像无法直接 pull 时，skopeo 是最靠谱的替代方案。核心思路就是绕开 Docker 客户端的格式校验，用 skopeo 先存为 tar，再通过 `docker load` 导入。整个过程中 Windows + WSL 环境下还需要注意 credential helper 和凭据管理器的兼容问题。
