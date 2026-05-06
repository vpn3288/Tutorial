# Windows 11 LTSC 完整 AI 代理环境安装指南

> **适用系统**: Windows 11 LTSC（最纯净版本，无 Microsoft Store）  
> **目标**: 从零开始安装所有依赖，使 Hermes Agent 完全控制您的电脑  
> **版本**: 2026年5月更新

---

## 📋 目录

1. [系统要求](#系统要求)
2. [安装概览](#安装概览)
3. [步骤 1: 安装 PowerShell 7.6.1+](#步骤-1-安装-powershell-761)
4. [步骤 2: 安装 Winget 包管理器](#步骤-2-安装-winget-包管理器)
5. [步骤 3: 安装 Git](#步骤-3-安装-git)
6. [步骤 4: 安装 Python 3.11+](#步骤-4-安装-python-311)
7. [步骤 5: 安装 Node.js 22+ LTS](#步骤-5-安装-nodejs-22-lts)
8. [步骤 6: 启用 WSL2](#步骤-6-启用-wsl2)
9. [步骤 7: 安装 Ubuntu 24.04](#步骤-7-安装-ubuntu-2404)
10. [步骤 8: 在 WSL 中安装 Hermes](#步骤-8-在-wsl-中安装-hermes)
11. [步骤 9: 配置 Windows 远程管理 (WinRM)](#步骤-9-配置-windows-远程管理-winrm)
12. [步骤 10: 配置 SSH Server](#步骤-10-配置-ssh-server)
13. [步骤 11: 配置 Windows Defender 排除项](#步骤-11-配置-windows-defender-排除项)
14. [步骤 12: 完整环境验证](#步骤-12-完整环境验证)
15. [步骤 13: 启动 Hermes Agent](#步骤-13-启动-hermes-agent)
16. [故障排查](#故障排查)
17. [安全建议](#安全建议)

---

## 系统要求

- **操作系统**: Windows 11 LTSC（全新安装，无 Microsoft Store）
- **处理器**: 支持虚拟化的 64 位处理器
- **内存**: 至少 8GB RAM（推荐 16GB+）
- **存储**: 至少 50GB 可用空间
- **网络**: 稳定的互联网连接
- **权限**: 管理员权限

---

## 安装概览

本指南将按以下顺序安装所有必要组件：

```
PowerShell 7 → Winget → Git → Python → Node.js → WSL2 → Ubuntu → Hermes → 安全配置
```

**重要提示**: 
- ⚠️ 所有命令必须在**管理员权限**的 PowerShell 中运行
- ⚠️ 某些步骤需要重启系统
- ⚠️ 请按顺序执行，不要跳过任何步骤

---

## 步骤 1: 安装 PowerShell 7.6.1+

PowerShell 7 是现代化的命令行工具，后续所有操作都将在此环境中进行。

### 1.1 下载并安装

以**管理员身份**打开 Windows PowerShell 5.1（系统自带），运行：

```powershell
# 下载 PowerShell 7 安装程序
$url = "https://github.com/PowerShell/PowerShell/releases/latest/download/PowerShell-7.6.1-win-x64.msi"
$output = "$env:TEMP\PowerShell-7.msi"
Invoke-WebRequest -Uri $url -OutFile $output

# 静默安装
Start-Process msiexec.exe -ArgumentList "/i `"$output`" /quiet ADD_EXPLORER_CONTEXT_MENU_OPENPOWERSHELL=1 ADD_FILE_CONTEXT_MENU_RUNPOWERSHELL=1 ENABLE_PSREMOTING=1 REGISTER_MANIFEST=1" -Wait

# 清理安装文件
Remove-Item $output
```

### 1.2 验证安装

关闭当前 PowerShell 窗口，以**管理员身份**打开新的 **PowerShell 7**（在开始菜单搜索 "PowerShell 7"），运行：

```powershell
$PSVersionTable.PSVersion
```

**预期输出**: 版本号应为 7.6.1 或更高

---

## 步骤 2: 安装 Winget 包管理器

Winget 是 Windows 的官方包管理器，但在 LTSC 版本中需要手动安装依赖。

### 2.1 安装依赖和 Winget

在 PowerShell 7（管理员）中运行：

```powershell
try {
    $ProgressPreference = 'SilentlyContinue'
    $ErrorActionPreference = 'Stop'
    
    # 检查并安装 VCLibs
    Write-Host "正在检查 VCLibs..." -ForegroundColor Cyan
    $vcLibsInstalled = Get-AppxPackage -Name "Microsoft.VCLibs.140.00.UWPDesktop" -ErrorAction SilentlyContinue
    if ($vcLibsInstalled) {
        Write-Host "✓ VCLibs 已安装 (版本: $($vcLibsInstalled.Version))" -ForegroundColor Green
    } else {
        Write-Host "正在安装 VCLibs..." -ForegroundColor Yellow
        $vcLibsUrl = "https://aka.ms/Microsoft.VCLibs.x64.14.00.Desktop.appx"
        $vcLibsPath = Join-Path $env:TEMP "VCLibs.appx"
        Invoke-WebRequest -Uri $vcLibsUrl -OutFile $vcLibsPath
        Add-AppxPackage -Path $vcLibsPath
        Remove-Item $vcLibsPath -ErrorAction SilentlyContinue
        Write-Host "✓ VCLibs 安装完成" -ForegroundColor Green
    }
    
    # 安装 Windows App Runtime
    Write-Host "正在安装 Windows App Runtime..." -ForegroundColor Cyan
    $runtimeUrl = "https://aka.ms/windowsappsdk/1.8/latest/windowsappruntimeinstall-x64.exe"
    $runtimePath = Join-Path $env:TEMP "WindowsAppRuntime.exe"
    Invoke-WebRequest -Uri $runtimeUrl -OutFile $runtimePath
    Start-Process -FilePath $runtimePath -ArgumentList "--quiet" -Wait
    Remove-Item $runtimePath -ErrorAction SilentlyContinue
    Write-Host "✓ Windows App Runtime 安装完成" -ForegroundColor Green
    
    # 检查并安装 Winget
    Write-Host "正在检查 Winget..." -ForegroundColor Cyan
    $wingetInstalled = Get-AppxPackage -Name "Microsoft.DesktopAppInstaller" -ErrorAction SilentlyContinue
    if ($wingetInstalled) {
        Write-Host "✓ Winget 已安装 (版本: $($wingetInstalled.Version))" -ForegroundColor Green
    } else {
        Write-Host "正在安装 Winget..." -ForegroundColor Yellow
        $wingetUrl = "https://github.com/microsoft/winget-cli/releases/latest/download/Microsoft.DesktopAppInstaller_8wekyb3d8bbwe.msixbundle"
        $wingetPath = Join-Path $env:TEMP "Winget.msixbundle"
        Invoke-WebRequest -Uri $wingetUrl -OutFile $wingetPath
        Add-AppxPackage -Path $wingetPath
        Remove-Item $wingetPath -ErrorAction SilentlyContinue
        Write-Host "✓ Winget 安装完成" -ForegroundColor Green
    }
    
    Write-Host "`n✓ 安装成功！请关闭并重新打开 PowerShell 以使用 winget 命令。" -ForegroundColor Yellow
    
} catch {
    Write-Host "❌ 安装失败: $_" -ForegroundColor Red
    Write-Host "请检查网络连接并重试。" -ForegroundColor Yellow
    Write-Host "`n如果错误提示已安装更高版本，可以忽略并继续下一步。" -ForegroundColor Cyan
}
```

### 2.2 验证安装

**关闭并重新打开 PowerShell 7（管理员）**，然后运行：

```powershell
winget --version
```

**预期输出**: 显示 Winget 版本号（如 v1.8.x）

---

## 步骤 3: 安装 Git

Git 是版本控制系统，用于管理代码和配置。

### 3.1 使用 Winget 安装

```powershell
winget install --id Git.Git -e --source winget --accept-package-agreements --accept-source-agreements
```

### 3.2 配置环境变量

```powershell
# 刷新环境变量
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
```

### 3.3 验证安装

```powershell
git --version
```

**预期输出**: `git version 2.53.x` 或更高

---

## 步骤 4: 安装 Python 3.11+

Python 是 Hermes Agent 的核心依赖。

### 4.1 使用 Winget 安装

```powershell
winget install --id Python.Python.3.11 -e --source winget --accept-package-agreements --accept-source-agreements
```

### 4.2 刷新环境变量

```powershell
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
```

### 4.3 验证安装

```powershell
python --version
pip --version
```

**预期输出**: 
- `Python 3.11.x` 或更高
- `pip 24.x` 或更高

---

## 步骤 5: 安装 Node.js 22+ LTS

Node.js 提供 JavaScript 运行时环境。

### 5.1 使用 Winget 安装

```powershell
winget install --id OpenJS.NodeJS.LTS -e --source winget --accept-package-agreements --accept-source-agreements
```

### 5.2 刷新环境变量

```powershell
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
```

### 5.3 验证安装

```powershell
node --version
npm --version
```

**预期输出**: 
- `v22.x.x` 或更高
- `10.x.x` 或更高

---

## 步骤 6: 启用 WSL2

WSL2（Windows Subsystem for Linux）允许在 Windows 上运行 Linux 环境。

### 6.1 启用 WSL 功能

```powershell
# 启用 WSL 和虚拟机平台
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

### 6.2 下载并安装 WSL2 内核更新

```powershell
$wslUpdateUrl = "https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi"
$wslUpdatePath = "$env:TEMP\wsl_update_x64.msi"
Invoke-WebRequest -Uri $wslUpdateUrl -OutFile $wslUpdatePath
Start-Process msiexec.exe -ArgumentList "/i `"$wslUpdatePath`" /quiet" -Wait
Remove-Item $wslUpdatePath
```

### 6.3 设置 WSL2 为默认版本

```powershell
wsl --set-default-version 2
```

### 6.4 重启系统

```powershell
Write-Host "⚠️ 请立即重启系统以完成 WSL2 安装！" -ForegroundColor Yellow
Write-Host "重启后，以管理员身份打开 PowerShell 7 继续下一步。" -ForegroundColor Cyan
```

**⚠️ 重要**: 必须重启系统才能继续！

---

## 步骤 7: 安装 Ubuntu 24.04

重启后，以**管理员身份**打开 PowerShell 7，继续安装。

### 7.1 安装 Ubuntu

```powershell
wsl --install -d Ubuntu-24.04
```

### 7.2 首次配置

安装完成后，Ubuntu 会自动启动。按提示：

1. 创建 Linux 用户名（建议使用小写字母）
2. 设置密码（输入时不显示，这是正常的）
3. 确认密码

### 7.3 更新 Ubuntu 系统

在 Ubuntu 终端中运行：

```bash
sudo apt update && sudo apt upgrade -y
```

### 7.4 验证 WSL 安装

返回 PowerShell，运行：

```powershell
wsl --list --verbose
```

**预期输出**: 显示 Ubuntu-24.04，状态为 Running，版本为 2

---

## 步骤 8: 在 WSL 中安装 Hermes

### 8.1 安装 Hermes Agent

在 PowerShell 中运行以下命令，在 WSL 中安装 Hermes：

```powershell
wsl bash -c "curl -fsSL https://hermes.ai/install.sh | bash"
```

### 8.2 配置 Hermes 环境变量

```powershell
wsl bash -c "echo 'export PATH=\$HOME/.hermes/bin:\$PATH' >> ~/.bashrc && source ~/.bashrc"
```

### 8.3 验证 Hermes 安装

```powershell
wsl hermes --version
```

**预期输出**: `Hermes v0.12.0` 或更高

---

## 步骤 9: 配置 Windows 远程管理 (WinRM)

WinRM 允许 Hermes Agent 远程控制 Windows 系统。

### 9.1 启用并配置 WinRM

```powershell
# 启用 WinRM 服务
Enable-PSRemoting -Force

# 配置 WinRM 监听器
winrm quickconfig -quiet

# 设置 WinRM 服务为自动启动
Set-Service WinRM -StartupType Automatic

# 配置信任主机（允许本地连接）
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "localhost,127.0.0.1" -Force

# 允许基本身份验证（仅用于本地测试）
Set-Item WSMan:\localhost\Service\Auth\Basic -Value $true

# 配置防火墙规则
Enable-NetFirewallRule -Name "WINRM-HTTP-In-TCP"
```

### 9.2 验证 WinRM

```powershell
Test-WSMan localhost
```

**预期输出**: 显示 WSMan 配置信息，无错误

---

## 步骤 10: 配置 SSH Server

SSH 提供安全的远程访问能力。

### 10.1 安装 OpenSSH Server

```powershell
# 安装 OpenSSH Server
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# 启动 SSH 服务
Start-Service sshd

# 设置为自动启动
Set-Service -Name sshd -StartupType 'Automatic'

# 配置防火墙
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
```

### 10.2 验证 SSH

```powershell
Get-Service sshd
```

**预期输出**: 状态为 Running

---

## 步骤 11: 配置 Windows Defender 排除项

防止 Windows Defender 干扰 Hermes Agent 运行。

### 11.1 添加排除项

```powershell
# 排除 WSL 目录
Add-MpPreference -ExclusionPath "C:\Users\$env:USERNAME\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu24.04LTS_*"

# 排除 Python 目录
Add-MpPreference -ExclusionPath "C:\Users\$env:USERNAME\AppData\Local\Programs\Python"

# 排除 Node.js 目录
Add-MpPreference -ExclusionPath "C:\Program Files\nodejs"

# 排除 Hermes 进程
Add-MpPreference -ExclusionProcess "hermes.exe"
Add-MpPreference -ExclusionProcess "python.exe"
Add-MpPreference -ExclusionProcess "node.exe"

Write-Host "✓ Windows Defender 排除项已配置" -ForegroundColor Green
```

### 11.2 验证排除项

```powershell
Get-MpPreference | Select-Object -ExpandProperty ExclusionPath
Get-MpPreference | Select-Object -ExpandProperty ExclusionProcess
```

---

## 步骤 12: 完整环境验证

运行完整的环境验证脚本：

```powershell
Write-Host "`n========== 完整环境验证 ==========" -ForegroundColor Cyan

# 验证 PowerShell
Write-Host "`n[1/9] PowerShell 版本:" -ForegroundColor Yellow
$psVersion = $PSVersionTable.PSVersion
Write-Host "  ✓ $psVersion" -ForegroundColor Green

# 验证 Winget
Write-Host "`n[2/9] Winget 版本:" -ForegroundColor Yellow
winget --version

# 验证 Git
Write-Host "`n[3/9] Git 版本:" -ForegroundColor Yellow
git --version

# 验证 Python
Write-Host "`n[4/9] Python 版本:" -ForegroundColor Yellow
python --version
pip --version

# 验证 Node.js
Write-Host "`n[5/9] Node.js 版本:" -ForegroundColor Yellow
node --version
npm --version

# 验证 WSL
Write-Host "`n[6/9] WSL 状态:" -ForegroundColor Yellow
wsl --list --verbose

# 验证 Hermes
Write-Host "`n[7/9] Hermes 版本:" -ForegroundColor Yellow
wsl hermes --version

# 验证 WinRM
Write-Host "`n[8/9] WinRM 状态:" -ForegroundColor Yellow
try {
    Test-WSMan localhost | Out-Null
    Write-Host "  ✓ WinRM 运行正常" -ForegroundColor Green
} catch {
    Write-Host "  ❌ WinRM 未正确配置" -ForegroundColor Red
}

# 验证 SSH
Write-Host "`n[9/9] SSH 服务状态:" -ForegroundColor Yellow
$sshStatus = Get-Service sshd
if ($sshStatus.Status -eq "Running") {
    Write-Host "  ✓ SSH 服务运行中" -ForegroundColor Green
} else {
    Write-Host "  ❌ SSH 服务未运行" -ForegroundColor Red
}

Write-Host "`n========== 验证完成 ==========" -ForegroundColor Cyan
Write-Host "如果所有项目都显示 ✓，则环境配置成功！" -ForegroundColor Green
```

---

## 步骤 13: 启动 Hermes Agent

### 13.1 初始化 Hermes

```powershell
wsl hermes init
```

按提示配置：
- API 密钥（如果需要）
- 工作目录
- 权限设置

### 13.2 启动 Hermes Agent

```powershell
wsl hermes start --full-control
```

**参数说明**:
- `--full-control`: 授予 Hermes 完全控制权限
- `--background`: 后台运行（可选）
- `--log-level debug`: 启用调试日志（可选）

### 13.3 验证 Hermes 运行状态

```powershell
wsl hermes status
```

**预期输出**: 显示 Hermes 正在运行，连接状态正常

---

## 故障排查

### 问题 1: Winget 安装失败

**症状**: `Add-AppxPackage` 命令报错

**解决方案**:
```powershell
# 检查 Windows 更新
Get-WindowsUpdate

# 手动下载并安装依赖
# 访问 https://github.com/microsoft/winget-cli/releases
# 下载最新的 .msixbundle 文件并手动安装
```

### 问题 2: WSL 安装失败

**症状**: `wsl --install` 命令报错

**解决方案**:
```powershell
# 检查虚拟化是否启用
Get-ComputerInfo | Select-Object -Property "HyperV*"

# 如果虚拟化未启用，需要在 BIOS 中启用 VT-x/AMD-V
```

### 问题 3: WinRM 连接失败

**症状**: `Test-WSMan` 报错

**解决方案**:
```powershell
# 重置 WinRM 配置
winrm delete winrm/config/listener?Address=*+Transport=HTTP
winrm quickconfig -quiet

# 检查防火墙
Get-NetFirewallRule -Name "WINRM-HTTP-In-TCP"
```

### 问题 4: Hermes 无法启动

**症状**: `hermes start` 命令失败

**解决方案**:
```bash
# 在 WSL 中检查日志
wsl cat ~/.hermes/logs/hermes.log

# 重新安装 Hermes
wsl bash -c "curl -fsSL https://hermes.ai/install.sh | bash"
```

### 问题 5: Python/Node.js 命令未找到

**症状**: 提示 "command not found"

**解决方案**:
```powershell
# 手动刷新环境变量
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")

# 或者关闭并重新打开 PowerShell
```

---

## 安全建议

### ⚠️ 重要安全提示

1. **仅在受信任的环境中使用完全控制模式**
   - Hermes Agent 的 `--full-control` 模式授予了广泛的系统权限
   - 不要在生产环境或包含敏感数据的系统上使用

2. **定期更新所有组件**
   ```powershell
   # 更新 Winget 包
   winget upgrade --all
   
   # 更新 WSL
   wsl --update
   
   # 更新 Hermes
   wsl hermes update
   ```

3. **配置防火墙规则**
   - 仅允许必要的端口访问
   - 限制 WinRM 和 SSH 仅接受本地连接（如果不需要远程访问）

4. **使用强密码**
   - WSL 用户密码
   - SSH 密钥认证（推荐）

5. **监控系统活动**
   ```powershell
   # 查看 Hermes 日志
   wsl tail -f ~/.hermes/logs/hermes.log
   
   # 查看 Windows 事件日志
   Get-EventLog -LogName Security -Newest 50
   ```

6. **备份重要数据**
   - 在授予完全控制权限前，确保已备份重要文件

---

## 后续步骤

安装完成后，您可以：

1. **配置 Hermes 工作流**
   ```powershell
   wsl hermes config --edit
   ```

2. **安装额外的工具**
   ```powershell
   # 安装 Docker Desktop
   winget install Docker.DockerDesktop
   
   # 安装 VS Code
   winget install Microsoft.VisualStudioCode
   ```

3. **学习 Hermes 命令**
   ```powershell
   wsl hermes help
   wsl hermes docs
   ```

4. **加入社区**
   - 访问 Hermes 官方文档
   - 加入 Discord/Slack 社区
   - 查看 GitHub 仓库

---

## 许可和免责声明

本指南仅供教育和研究目的使用。使用 AI 代理完全控制系统存在风险，请确保：

- 您了解授予的权限范围
- 您在隔离或测试环境中操作
- 您已阅读并理解 Hermes Agent 的使用条款
- 您对系统的任何更改负责

**作者不对因使用本指南而导致的任何数据丢失、系统损坏或其他问题承担责任。**

---

## 贡献

如果您发现本指南有任何错误或改进建议，欢迎：

- 提交 Issue
- 创建 Pull Request
- 分享您的使用经验

---

**最后更新**: 2026年5月6日  
**版本**: 1.0.0  
**维护者**: OpenCode Community

---

## 快速参考

### 常用命令

```powershell
# 启动 Hermes
wsl hermes start --full-control

# 停止 Hermes
wsl hermes stop

# 查看状态
wsl hermes status

# 查看日志
wsl hermes logs

# 重启 WinRM
Restart-Service WinRM

# 重启 SSH
Restart-Service sshd

# 更新所有软件
winget upgrade --all
wsl --update
wsl hermes update
```

### 重要路径

- **PowerShell 7**: `C:\Program Files\PowerShell\7\`
- **Python**: `C:\Users\<用户名>\AppData\Local\Programs\Python\`
- **Node.js**: `C:\Program Files\nodejs\`
- **WSL Ubuntu**: `\\wsl$\Ubuntu-24.04\`
- **Hermes**: `~/.hermes/` (在 WSL 中)

---

🎉 **恭喜！您已成功配置完整的 Hermes Agent 环境！**
