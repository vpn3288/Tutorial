# Windows 11 LTSC Hermes Agent 完整安装指南

本指南适用于全新的 Windows 11 LTSC 系统（无 Microsoft Store 版本），从零开始安装所有依赖和 Hermes Agent。

## 目录
1. [PowerShell 7 安装](#1-powershell-7-安装)
2. [Winget 安装](#2-winget-安装)
3. [Git 安装](#3-git-安装)
4. [Python 安装](#4-python-安装)
5. [Node.js 安装](#5-nodejs-安装)
6. [WSL2 安装](#6-wsl2-安装)
7. [Hermes Agent 安装（WSL内）](#7-hermes-agent-安装wsl内)
8. [远程管理配置](#8-远程管理配置)
9. [安全配置](#9-安全配置)
10. [验证与测试](#10-验证与测试)

---

## 1. PowerShell 7 安装

### 一键安装脚本（带版本检测）

```powershell
# 检查 PowerShell 7 版本
try {
    $pwshVersion = pwsh -Command '$PSVersionTable.PSVersion.ToString()' 2>$null
    if ($pwshVersion) {
        Write-Host "✓ PowerShell 7 已安装 (版本: $pwshVersion)" -ForegroundColor Green
        $continue = Read-Host "是否重新安装最新版本？(y/N)"
        if ($continue -ne 'y') {
            Write-Host "跳过 PowerShell 7 安装" -ForegroundColor Yellow
            exit 0
        }
    }
} catch {
    Write-Host "PowerShell 7 未安装，开始安装..." -ForegroundColor Cyan
}

# 安装 PowerShell 7
$ProgressPreference = 'SilentlyContinue'
Write-Host "正在下载 PowerShell 7..." -ForegroundColor Cyan
Invoke-WebRequest -Uri "https://github.com/PowerShell/PowerShell/releases/latest/download/PowerShell-7-win-x64.msi" -OutFile "$env:TEMP\PowerShell-7.msi"

Write-Host "正在安装 PowerShell 7..." -ForegroundColor Cyan
Start-Process msiexec.exe -ArgumentList "/i `"$env:TEMP\PowerShell-7.msi`" /quiet ADD_EXPLORER_CONTEXT_MENU_OPENPOWERSHELL=1 ADD_FILE_CONTEXT_MENU_RUNPOWERSHELL=1 ENABLE_PSREMOTING=1 REGISTER_MANIFEST=1" -Wait

Remove-Item "$env:TEMP\PowerShell-7.msi" -Force
Write-Host "✓ PowerShell 7 安装完成" -ForegroundColor Green
Write-Host "请关闭当前窗口，使用 PowerShell 7 继续后续步骤" -ForegroundColor Yellow
```

---

## 2. Winget 安装

### 一键安装脚本（带版本检测）

```powershell
# 检查 Winget 是否已安装
try {
    $wingetVersion = winget --version 2>$null
    if ($wingetVersion) {
        Write-Host "✓ Winget 已安装 (版本: $wingetVersion)" -ForegroundColor Green
        exit 0
    }
} catch {
    Write-Host "Winget 未安装，开始安装..." -ForegroundColor Cyan
}

$ProgressPreference = 'SilentlyContinue'
$ErrorActionPreference = 'Stop'

# 检查并安装 VCLibs
Write-Host "正在检查 VCLibs..." -ForegroundColor Cyan
$vcLibsInstalled = Get-AppxPackage -Name "Microsoft.VCLibs.140.00.UWPDesktop" -ErrorAction SilentlyContinue
if ($vcLibsInstalled) {
    Write-Host "✓ VCLibs 已安装 (版本: $($vcLibsInstalled.Version))" -ForegroundColor Green
} else {
    Write-Host "正在下载 VCLibs..." -ForegroundColor Cyan
    $vcLibsUrl = "https://aka.ms/Microsoft.VCLibs.x64.14.00.Desktop.appx"
    $vcLibsPath = "$env:TEMP\Microsoft.VCLibs.x64.14.00.Desktop.appx"
    Invoke-WebRequest -Uri $vcLibsUrl -OutFile $vcLibsPath
    
    Write-Host "正在安装 VCLibs..." -ForegroundColor Cyan
    Add-AppxPackage -Path $vcLibsPath
    Remove-Item $vcLibsPath -Force
    Write-Host "✓ VCLibs 安装完成" -ForegroundColor Green
}

# 检查并安装 Windows App Runtime
Write-Host "正在检查 Windows App Runtime..." -ForegroundColor Cyan
$appRuntimeInstalled = Get-AppxPackage -Name "Microsoft.WindowsAppRuntime.1.6" -ErrorAction SilentlyContinue
if ($appRuntimeInstalled) {
    Write-Host "✓ Windows App Runtime 已安装 (版本: $($appRuntimeInstalled.Version))" -ForegroundColor Green
} else {
    Write-Host "正在下载 Windows App Runtime..." -ForegroundColor Cyan
    $appRuntimeUrl = "https://aka.ms/windowsappruntimelatest"
    $appRuntimePath = "$env:TEMP\Microsoft.WindowsAppRuntime.appx"
    Invoke-WebRequest -Uri $appRuntimeUrl -OutFile $appRuntimePath
    
    Write-Host "正在安装 Windows App Runtime..." -ForegroundColor Cyan
    Add-AppxPackage -Path $appRuntimePath
    Remove-Item $appRuntimePath -Force
    Write-Host "✓ Windows App Runtime 安装完成" -ForegroundColor Green
}

# 安装 Winget
Write-Host "正在下载 Winget..." -ForegroundColor Cyan
$wingetUrl = "https://github.com/microsoft/winget-cli/releases/latest/download/Microsoft.DesktopAppInstaller_8wekyb3d8bbwe.msixbundle"
$wingetPath = "$env:TEMP\Microsoft.DesktopAppInstaller.msixbundle"
Invoke-WebRequest -Uri $wingetUrl -OutFile $wingetPath

Write-Host "正在安装 Winget..." -ForegroundColor Cyan
Add-AppxPackage -Path $wingetPath
Remove-Item $wingetPath -Force

Write-Host "✓ Winget 安装完成" -ForegroundColor Green
Write-Host "请重新打开 PowerShell 窗口以使用 winget 命令" -ForegroundColor Yellow
```

---

## 3. Git 安装

### 一键安装脚本（带版本检测）

```powershell
# 检查 Git 版本
try {
    $gitVersion = git --version 2>$null
    if ($gitVersion -match 'git version (\d+\.\d+\.\d+)') {
        $version = [version]$matches[1]
        Write-Host "✓ Git 已安装 (版本: $gitVersion)" -ForegroundColor Green
        
        if ($version -ge [version]"2.53.0") {
            Write-Host "Git 版本符合要求 (>= 2.53.0)" -ForegroundColor Green
            $continue = Read-Host "是否重新安装最新版本？(y/N)"
            if ($continue -ne 'y') {
                Write-Host "跳过 Git 安装" -ForegroundColor Yellow
                exit 0
            }
        } else {
            Write-Host "Git 版本过旧，需要升级到 2.53.0 或更高版本" -ForegroundColor Yellow
        }
    }
} catch {
    Write-Host "Git 未安装，开始安装..." -ForegroundColor Cyan
}

# 使用 Winget 安装 Git
Write-Host "正在安装 Git..." -ForegroundColor Cyan
winget install --id Git.Git -e --source winget --accept-package-agreements --accept-source-agreements

Write-Host "✓ Git 安装完成" -ForegroundColor Green
Write-Host "请重新打开 PowerShell 窗口以使用 git 命令" -ForegroundColor Yellow
```

---

## 4. Python 安装

### 一键安装脚本（带版本检测）

```powershell
# 检查 Python 版本
try {
    $pythonVersion = python --version 2>$null
    if ($pythonVersion -match 'Python (\d+\.\d+\.\d+)') {
        $version = [version]$matches[1]
        Write-Host "✓ Python 已安装 (版本: $pythonVersion)" -ForegroundColor Green
        
        if ($version -ge [version]"3.11.0") {
            Write-Host "Python 版本符合要求 (>= 3.11.0)" -ForegroundColor Green
            $continue = Read-Host "是否重新安装最新版本？(y/N)"
            if ($continue -ne 'y') {
                Write-Host "跳过 Python 安装" -ForegroundColor Yellow
                exit 0
            }
        } else {
            Write-Host "Python 版本过旧，需要升级到 3.11.0 或更高版本" -ForegroundColor Yellow
        }
    }
} catch {
    Write-Host "Python 未安装，开始安装..." -ForegroundColor Cyan
}

# 使用 Winget 安装 Python 3.11
Write-Host "正在安装 Python 3.11..." -ForegroundColor Cyan
winget install --id Python.Python.3.11 -e --source winget --accept-package-agreements --accept-source-agreements

Write-Host "✓ Python 安装完成" -ForegroundColor Green
Write-Host "请重新打开 PowerShell 窗口以使用 python 命令" -ForegroundColor Yellow
```

---

## 5. Node.js 安装

### 一键安装脚本（带版本检测）

```powershell
# 检查 Node.js 版本
try {
    $nodeVersion = node --version 2>$null
    if ($nodeVersion -match 'v(\d+\.\d+\.\d+)') {
        $version = [version]$matches[1]
        Write-Host "✓ Node.js 已安装 (版本: $nodeVersion)" -ForegroundColor Green
        
        if ($version -ge [version]"22.0.0") {
            Write-Host "Node.js 版本符合要求 (>= 22.0.0 LTS)" -ForegroundColor Green
            $continue = Read-Host "是否重新安装最新 LTS 版本？(y/N)"
            if ($continue -ne 'y') {
                Write-Host "跳过 Node.js 安装" -ForegroundColor Yellow
                exit 0
            }
        } else {
            Write-Host "Node.js 版本过旧，需要升级到 22.0.0 LTS 或更高版本" -ForegroundColor Yellow
        }
    }
} catch {
    Write-Host "Node.js 未安装，开始安装..." -ForegroundColor Cyan
}

# 使用 Winget 安装 Node.js LTS
Write-Host "正在安装 Node.js LTS..." -ForegroundColor Cyan
winget install --id OpenJS.NodeJS.LTS -e --source winget --accept-package-agreements --accept-source-agreements

Write-Host "✓ Node.js 安装完成" -ForegroundColor Green
Write-Host "请重新打开 PowerShell 窗口以使用 node 和 npm 命令" -ForegroundColor Yellow
```

---

## 6. WSL2 安装

### 一键安装脚本（带版本检测）

```powershell
# 检查 WSL 是否已安装
try {
    $wslVersion = wsl --version 2>$null
    if ($wslVersion) {
        Write-Host "✓ WSL 已安装" -ForegroundColor Green
        Write-Host $wslVersion
        
        # 检查是否已安装 Ubuntu
        $ubuntuInstalled = wsl -l -v | Select-String "Ubuntu"
        if ($ubuntuInstalled) {
            Write-Host "✓ Ubuntu 已安装" -ForegroundColor Green
            Write-Host "跳过 WSL 安装" -ForegroundColor Yellow
            exit 0
        } else {
            Write-Host "Ubuntu 未安装，开始安装..." -ForegroundColor Cyan
        }
    }
} catch {
    Write-Host "WSL 未安装，开始安装..." -ForegroundColor Cyan
}

# 启用 WSL 功能
Write-Host "正在启用 WSL 功能..." -ForegroundColor Cyan
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# 启用虚拟机平台
Write-Host "正在启用虚拟机平台..." -ForegroundColor Cyan
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# 下载并安装 WSL2 内核更新包
Write-Host "正在下载 WSL2 内核更新包..." -ForegroundColor Cyan
$wslUpdateUrl = "https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi"
$wslUpdatePath = "$env:TEMP\wsl_update_x64.msi"
Invoke-WebRequest -Uri $wslUpdateUrl -OutFile $wslUpdatePath

Write-Host "正在安装 WSL2 内核更新包..." -ForegroundColor Cyan
Start-Process msiexec.exe -ArgumentList "/i `"$wslUpdatePath`" /quiet" -Wait
Remove-Item $wslUpdatePath -Force

# 设置 WSL2 为默认版本
Write-Host "正在设置 WSL2 为默认版本..." -ForegroundColor Cyan
wsl --set-default-version 2

# 安装 Ubuntu 24.04
Write-Host "正在安装 Ubuntu 24.04..." -ForegroundColor Cyan
wsl --install -d Ubuntu-24.04

Write-Host "✓ WSL2 和 Ubuntu 24.04 安装完成" -ForegroundColor Green
Write-Host "请重启计算机以完成 WSL2 安装" -ForegroundColor Yellow
Write-Host "重启后，首次启动 Ubuntu 时需要创建用户名和密码" -ForegroundColor Yellow
```

---

## 7. Hermes Agent 安装（WSL内）

**重要：Hermes Agent 必须在 WSL Ubuntu 环境内安装，而不是在 Windows 中安装。**

### 步骤 1：进入 WSL Ubuntu

```powershell
# 从 Windows PowerShell 进入 WSL
wsl
```

### 步骤 2：在 WSL 内安装 Hermes Agent

在 WSL Ubuntu 终端中执行以下命令：

```bash
# 更新系统包
sudo apt update && sudo apt upgrade -y

# 安装必要的依赖
sudo apt install -y curl git build-essential

# 下载并执行 Hermes Agent 安装脚本
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash

# 安装完成后，重新加载 shell 配置
source ~/.bashrc
```

### 步骤 3：验证 Hermes Agent 安装

```bash
# 检查 Hermes 版本
hermes --version

# 查看 Hermes 帮助信息
hermes --help
```

### 步骤 4：配置 Hermes Agent

```bash
# 初始化 Hermes 配置
hermes init

# 根据提示配置 API 密钥和其他设置
```

---

## 8. 远程管理配置

### WinRM 配置（Windows 远程管理）

```powershell
# 启用 WinRM 服务
Enable-PSRemoting -Force

# 配置 WinRM 监听器
winrm quickconfig -quiet

# 设置 WinRM 服务为自动启动
Set-Service WinRM -StartupType Automatic

# 配置防火墙规则
New-NetFirewallRule -Name "WinRM-HTTP" -DisplayName "Windows Remote Management (HTTP-In)" -Enabled True -Direction Inbound -Protocol TCP -LocalPort 5985

Write-Host "✓ WinRM 配置完成" -ForegroundColor Green
```

### SSH Server 配置

```powershell
# 安装 OpenSSH Server
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# 启动 SSH 服务
Start-Service sshd

# 设置 SSH 服务为自动启动
Set-Service -Name sshd -StartupType 'Automatic'

# 配置防火墙规则（通常会自动创建）
if (!(Get-NetFirewallRule -Name "OpenSSH-Server-In-TCP" -ErrorAction SilentlyContinue)) {
    New-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
}

Write-Host "✓ SSH Server 配置完成" -ForegroundColor Green
Write-Host "SSH 端口: 22" -ForegroundColor Cyan
```

---

## 9. 安全配置

### 执行策略配置

```powershell
# 设置执行策略为 RemoteSigned（允许本地脚本和已签名的远程脚本）
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser -Force

Write-Host "✓ 执行策略已设置为 RemoteSigned" -ForegroundColor Green
```

### 防火墙配置

```powershell
# 确保 Windows 防火墙已启用
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True

# 允许 Hermes Agent 相关端口（根据实际需求调整）
# 示例：允许端口 8080
New-NetFirewallRule -DisplayName "Hermes Agent" -Direction Inbound -Protocol TCP -LocalPort 8080 -Action Allow

Write-Host "✓ 防火墙配置完成" -ForegroundColor Green
```

---

## 10. 验证与测试

### 完整验证脚本

```powershell
Write-Host "`n=== 系统环境验证 ===" -ForegroundColor Cyan

# 验证 PowerShell 7
Write-Host "`n[1] PowerShell 7:" -ForegroundColor Yellow
try {
    $pwshVersion = pwsh -Command '$PSVersionTable.PSVersion.ToString()'
    Write-Host "  ✓ 版本: $pwshVersion" -ForegroundColor Green
} catch {
    Write-Host "  ✗ 未安装或无法访问" -ForegroundColor Red
}

# 验证 Winget
Write-Host "`n[2] Winget:" -ForegroundColor Yellow
try {
    $wingetVersion = winget --version
    Write-Host "  ✓ 版本: $wingetVersion" -ForegroundColor Green
} catch {
    Write-Host "  ✗ 未安装或无法访问" -ForegroundColor Red
}

# 验证 Git
Write-Host "`n[3] Git:" -ForegroundColor Yellow
try {
    $gitVersion = git --version
    Write-Host "  ✓ $gitVersion" -ForegroundColor Green
} catch {
    Write-Host "  ✗ 未安装或无法访问" -ForegroundColor Red
}

# 验证 Python
Write-Host "`n[4] Python:" -ForegroundColor Yellow
try {
    $pythonVersion = python --version
    Write-Host "  ✓ $pythonVersion" -ForegroundColor Green
    $pipVersion = pip --version
    Write-Host "  ✓ $pipVersion" -ForegroundColor Green
} catch {
    Write-Host "  ✗ 未安装或无法访问" -ForegroundColor Red
}

# 验证 Node.js
Write-Host "`n[5] Node.js:" -ForegroundColor Yellow
try {
    $nodeVersion = node --version
    Write-Host "  ✓ Node.js $nodeVersion" -ForegroundColor Green
    $npmVersion = npm --version
    Write-Host "  ✓ npm $npmVersion" -ForegroundColor Green
} catch {
    Write-Host "  ✗ 未安装或无法访问" -ForegroundColor Red
}

# 验证 WSL
Write-Host "`n[6] WSL:" -ForegroundColor Yellow
try {
    $wslStatus = wsl --status
    Write-Host "  ✓ WSL 已安装" -ForegroundColor Green
    $wslList = wsl -l -v
    Write-Host "  已安装的发行版:" -ForegroundColor Cyan
    Write-Host $wslList
} catch {
    Write-Host "  ✗ 未安装或无法访问" -ForegroundColor Red
}

# 验证 Hermes Agent（在 WSL 内）
Write-Host "`n[7] Hermes Agent (WSL):" -ForegroundColor Yellow
try {
    $hermesVersion = wsl bash -c "hermes --version 2>/dev/null"
    if ($hermesVersion) {
        Write-Host "  ✓ $hermesVersion" -ForegroundColor Green
    } else {
        Write-Host "  ✗ 未安装或无法访问" -ForegroundColor Red
    }
} catch {
    Write-Host "  ✗ 未安装或无法访问" -ForegroundColor Red
}

# 验证 WinRM
Write-Host "`n[8] WinRM:" -ForegroundColor Yellow
try {
    $winrmStatus = Get-Service WinRM
    if ($winrmStatus.Status -eq 'Running') {
        Write-Host "  ✓ WinRM 服务运行中" -ForegroundColor Green
    } else {
        Write-Host "  ✗ WinRM 服务未运行" -ForegroundColor Red
    }
} catch {
    Write-Host "  ✗ WinRM 服务未配置" -ForegroundColor Red
}

# 验证 SSH Server
Write-Host "`n[9] SSH Server:" -ForegroundColor Yellow
try {
    $sshStatus = Get-Service sshd
    if ($sshStatus.Status -eq 'Running') {
        Write-Host "  ✓ SSH 服务运行中" -ForegroundColor Green
    } else {
        Write-Host "  ✗ SSH 服务未运行" -ForegroundColor Red
    }
} catch {
    Write-Host "  ✗ SSH 服务未安装" -ForegroundColor Red
}

Write-Host "`n=== 验证完成 ===" -ForegroundColor Cyan
```

---

## 故障排查

### 常见问题

#### 1. Winget 安装失败
- **问题**：VCLibs 或 Windows App Runtime 安装失败
- **解决方案**：
  - 检查是否已安装更高版本：`Get-AppxPackage | Select-String "VCLibs"`
  - 手动下载并安装依赖包
  - 确保以管理员权限运行 PowerShell

#### 2. WSL 安装后无法启动
- **问题**：WSL 启动失败或报错
- **解决方案**：
  - 确保已启用虚拟化（在 BIOS 中）
  - 运行 `wsl --update` 更新 WSL
  - 检查 Windows 更新是否完整

#### 3. Hermes Agent 安装失败
- **问题**：安装脚本下载失败或执行错误
- **解决方案**：
  - 确保在 WSL Ubuntu 内执行安装命令
  - 检查网络连接
  - 手动克隆 GitHub 仓库：`git clone https://github.com/NousResearch/hermes-agent.git`
  - 查看安装脚本内容并手动执行步骤

#### 4. 远程管理连接失败
- **问题**：无法通过 WinRM 或 SSH 连接
- **解决方案**：
  - 检查防火墙规则是否正确配置
  - 确保服务正在运行：`Get-Service WinRM,sshd`
  - 测试端口连通性：`Test-NetConnection -ComputerName localhost -Port 5985`

---

## 附录

### 系统要求
- **操作系统**：Windows 11 LTSC（64位）
- **内存**：至少 8GB RAM（推荐 16GB）
- **存储**：至少 50GB 可用空间
- **网络**：稳定的互联网连接
- **权限**：管理员权限

### 安装顺序
1. PowerShell 7（必须）
2. Winget（必须）
3. Git（必须）
4. Python 3.11+（必须）
5. Node.js 22+ LTS（必须）
6. WSL2 + Ubuntu 24.04（必须）
7. Hermes Agent（在 WSL 内安装）
8. 远程管理配置（可选）
9. 安全配置（推荐）

### 参考链接
- [PowerShell 7 官方文档](https://docs.microsoft.com/powershell/)
- [Winget 官方文档](https://docs.microsoft.com/windows/package-manager/)
- [WSL 官方文档](https://docs.microsoft.com/windows/wsl/)
- [Hermes Agent GitHub](https://github.com/NousResearch/hermes-agent)
- [Git 官方网站](https://git-scm.com/)
- [Python 官方网站](https://www.python.org/)
- [Node.js 官方网站](https://nodejs.org/)

---

## 许可证
本指南基于原始教程创建，供学习和参考使用。

## 贡献
欢迎提交问题和改进建议。

---

**最后更新**：2026年5月7日
