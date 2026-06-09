---
title: Windows 11 开启 SSH Server
tags:
  - Windows
  - SSH
  - OpenSSH
categories:
  - - ops
date: 2026-03-31
---

Windows 11 内置了 OpenSSH，只需要几条 PowerShell 命令就能开启 SSH Server，方便远程管理。本文记录完整的安装、配置流程，以及一个常见的启动失败问题的解决方案。

<!-- more -->

## 安装 OpenSSH Server

以管理员身份打开 PowerShell，先确认 OpenSSH 的安装状态：

```powershell
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
```

如果显示 `State : NotPresent`，执行安装：

```powershell
# 安装 OpenSSH 客户端
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0

# 安装 OpenSSH 服务端
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

## 启动并配置服务

```powershell
# 启动 sshd 服务
Start-Service sshd

# 设置开机自启
Set-Service -Name sshd -StartupType 'Automatic'
```

安装过程会自动创建防火墙规则，放行 22 端口入站流量。可以用以下脚本确认：

```powershell
if (!(Get-NetFirewallRule -Name "OpenSSH-Server-In-TCP" -ErrorAction SilentlyContinue)) {
    New-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
} else {
    Write-Output "Firewall rule 'OpenSSH-Server-In-TCP' already exists."
}
```

## 常见问题：sshd 启动失败

执行 `Start-Service sshd` 时服务无法启动，这是一个已知问题。

### 原因

Windows 安全更新为 `sshd.exe` 启用了 RedirectionGuard 等安全缓解策略（MitigationOptions），某些环境下会导致 sshd 进程异常退出。

### 解决方案

通过注册表将 `sshd.exe` 的 MitigationOptions 清零，禁用这些额外的安全缓解策略。

将以下内容保存为 `.reg` 文件并双击导入：

```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\sshd.exe]
"MitigationOptions"=hex:00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00
```

或者在管理员 PowerShell 中执行：

```powershell
$regPath = "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\sshd.exe"
if (!(Test-Path $regPath)) {
    New-Item -Path $regPath -Force
}
Set-ItemProperty -Path $regPath -Name "MitigationOptions" -Value ([byte[]](,0x00 * 19)) -Type Binary
```

修改后重新启动服务：

```powershell
Start-Service sshd
```

## 修改默认 Shell

SSH 登录后默认使用 `cmd.exe`。如果想改为 PowerShell 7（pwsh），以管理员身份执行：

```powershell
New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" -Name DefaultShell -Value "C:\Program Files\PowerShell\7\pwsh.exe" -PropertyType String -Force
```

设置后无需重启 sshd 服务，新的 SSH 连接即会使用 pwsh。

> 如果使用的是 Windows 自带的 PowerShell 5.1，将路径改为 `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`。

## 连接测试

从另一台机器（或本机）测试连接：

```powershell
ssh username@hostname
```

首次连接会提示确认主机指纹，输入 `yes` 后输入密码即可登录。

## 参考

- [Get started with OpenSSH for Windows - Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse?tabs=powershell&pivots=windows-11)
- [sshd MitigationOptions issue - GitHub](https://github.com/ScoopInstaller/Scoop/issues/6594)
