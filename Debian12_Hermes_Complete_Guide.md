# Debian 12 精简版 Hermes Agent 完整安装指南

本指南适用于**全新的 Debian 12 精简版系统**，所有命令均可**一键复制粘贴到 SSH 执行**，从零开始安装所有依赖和 Hermes Agent。

**特点：全部采用直接下载安装，无需在线编译。**

## 📋 目录
1. [系统准备与基础工具](#1-系统准备与基础工具)
2. [Git 安装](#2-git-安装)
3. [Python 3.11+ 安装](#3-python-311-安装)
4. [Node.js 22 LTS 安装](#4-nodejs-22-lts-安装)
5. [Hermes Agent 安装](#5-hermes-agent-安装)
6. [配置与初始化](#6-配置与初始化)
7. [验证测试](#7-验证测试)
8. [远程管理配置（可选）](#8-远程管理配置可选)
9. [一键安装脚本](#9-一键安装脚本)
10. [故障排查](#10-故障排查)

---

## 💻 系统要求

- **操作系统**：Debian 12 (bookworm) 64位
- **架构**：x86_64 / amd64 / ARM64
- **内存**：至少 2GB RAM（推荐 4GB+）
- **存储**：至少 10GB 可用空间
- **网络**：稳定的互联网连接
- **权限**：root 权限或 sudo 权限

---

## 1. 系统准备与基础工具

### 步骤 1.1：安装 sudo（如果系统没有）

**一键复制执行**：

```bash
# 如果当前不是 root 用户，先切换到 root
su -

# 更新包列表并安装 sudo
apt update && apt install -y sudo

# 将当前用户添加到 sudo 组（替换 yourusername 为实际用户名）
usermod -aG sudo yourusername

# 退出 root，重新登录以使组权限生效
exit
```

**验证**：
```bash
sudo -v
```

---

### 步骤 1.2：更新系统并安装基础工具

**一键复制执行**：

```bash
sudo apt update && sudo apt upgrade -y && \
sudo apt install -y \
    curl \
    wget \
    ca-certificates \
    gnupg \
    lsb-release \
    locales \
    tzdata \
    vim \
    nano \
    unzip \
    zip \
    tar \
    gzip \
    bzip2 \
    xz-utils
```

**配置语言环境（可选）**：

```bash
sudo sed -i 's/^# *\(en_US.UTF-8\)/\1/' /etc/locale.gen && \
sudo sed -i 's/^# *\(zh_CN.UTF-8\)/\1/' /etc/locale.gen && \
sudo locale-gen && \
sudo update-locale LANG=en_US.UTF-8
```

**配置时区（可选）**：

```bash
sudo timedatectl set-timezone Asia/Shanghai
```

**验证**：
```bash
curl --version && \
wget --version
```

---

## 2. Git 安装

### 步骤 2.1：直接从 Debian 仓库安装 Git

**一键复制执行**：

```bash
sudo apt install -y git
```

**验证**：
```bash
git --version
```

**配置 Git**：

```bash
git config --global user.name "Your Name" && \
git config --global user.email "your.email@example.com" && \
git config --global init.defaultBranch main && \
git config --global credential.helper 'cache --timeout=3600'
```

---

## 3. Python 3.11+ 安装

### 步骤 3.1：安装 Python 3.11 和开发工具

Debian 12 默认自带 Python 3.11，直接安装相关工具即可。

**一键复制执行**：

```bash
sudo apt install -y \
    python3 \
    python3-pip \
    python3-venv \
    python3-dev \
    python3-setuptools \
    python3-wheel \
    python3-full
```

**创建符号链接（可选，方便使用）**：

```bash
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 1 && \
sudo update-alternatives --install /usr/bin/pip pip /usr/bin/pip3 1
```

**升级 pip 到最新版本**：

```bash
python3 -m pip install --upgrade pip --break-system-packages
```

**安装常用 Python 工具**：

```bash
pip3 install --user --break-system-packages \
    setuptools \
    wheel \
    virtualenv \
    pipx
```

**配置 pipx**：

```bash
python3 -m pipx ensurepath && \
source ~/.bashrc
```

**验证**：
```bash
python3 --version && \
pip3 --version
```

---

## 4. Node.js 22 LTS 安装

### 步骤 4.1：使用 NodeSource 仓库直接安装 Node.js 22 LTS

**一键复制执行**：

```bash
# 添加 NodeSource 仓库
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -

# 安装 Node.js
sudo apt install -y nodejs

# 验证安装
node --version && npm --version
```

**配置 npm 全局包路径（避免权限问题）**：

```bash
mkdir -p ~/.npm-global && \
npm config set prefix '~/.npm-global' && \
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc && \
source ~/.bashrc
```

**升级 npm 到最新版本**：

```bash
npm install -g npm@latest
```

**验证**：
```bash
node --version && \
npm --version
```

---

## 5. Hermes Agent 安装

### 步骤 5.1：使用官方安装脚本

**一键复制执行**：

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash && \
source ~/.bashrc
```

**验证安装**：

```bash
hermes --version && \
hermes --help
```

---

### 步骤 5.2：手动安装（如果自动安装失败）

**一键复制执行**：

```bash
cd ~ && \
git clone https://github.com/NousResearch/hermes-agent.git && \
cd hermes-agent && \
pip3 install -r requirements.txt --break-system-packages && \
npm install && \
npm run build && \
sudo ln -sf $(pwd)/bin/hermes /usr/local/bin/hermes && \
cd ~
```

**验证**：
```bash
hermes --version
```

---

## 6. 配置与初始化

### 步骤 6.1：初始化 Hermes Agent

**一键复制执行**：

```bash
hermes init
```

根据提示配置：
- API 提供商（Anthropic, OpenAI, OpenRouter 等）
- API 密钥
- 默认模型
- 其他设置

---

### 步骤 6.2：配置 API 密钥（命令行方式）

**Anthropic Claude**：

```bash
hermes config set provider anthropic && \
hermes config set anthropic.api_key "your-anthropic-api-key-here" && \
hermes config set model "claude-opus-4"
```

**OpenAI**：

```bash
hermes config set provider openai && \
hermes config set openai.api_key "your-openai-api-key-here" && \
hermes config set model "gpt-4"
```

**OpenRouter**：

```bash
hermes config set provider openrouter && \
hermes config set openrouter.api_key "your-openrouter-api-key-here" && \
hermes config set model "anthropic/claude-opus-4"
```

---

### 步骤 6.3：配置环境变量（可选）

**一键复制执行**：

```bash
cat >> ~/.bashrc << 'EOF'

# Hermes Agent 环境变量
export HERMES_HOME="$HOME/.hermes"
export ANTHROPIC_API_KEY="your-anthropic-api-key-here"
export OPENAI_API_KEY="your-openai-api-key-here"
export OPENROUTER_API_KEY="your-openrouter-api-key-here"

EOF

source ~/.bashrc
```

**记得替换上面的 API 密钥为你的实际密钥！**

---

## 7. 验证测试

### 步骤 7.1：创建并运行验证脚本

**一键复制执行**（创建验证脚本）：

```bash
cat > ~/verify_hermes.sh << 'EOFSCRIPT'
#!/bin/bash

echo "=== Debian 12 Hermes Agent 安装验证 ==="
echo ""

GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m'

verify_command() {
    local cmd=$1
    local name=$2
    local version_flag=${3:---version}
    
    echo -n "[$name] "
    if command -v $cmd &> /dev/null; then
        version=$($cmd $version_flag 2>&1 | head -n 1)
        echo -e "${GREEN}✓${NC} $version"
        return 0
    else
        echo -e "${RED}✗ 未安装${NC}"
        return 1
    fi
}

echo "[1] 基础工具"
verify_command curl "curl"
verify_command wget "wget"
echo ""

echo "[2] Git"
verify_command git "git"
echo ""

echo "[3] Python"
verify_command python3 "python3"
verify_command pip3 "pip3"
echo ""

echo "[4] Node.js"
verify_command node "node"
verify_command npm "npm"
echo ""

echo "[5] Hermes Agent"
verify_command hermes "hermes"
echo ""

echo "[6] 系统信息"
echo -n "操作系统: "
cat /etc/os-release | grep PRETTY_NAME | cut -d'"' -f2
echo -n "架构: "
uname -m
echo -n "内核: "
uname -r
echo -n "内存: "
free -h | awk '/^Mem:/ {print $2}'
echo -n "磁盘可用: "
df -h / | awk 'NR==2 {print $4}'
echo ""

echo "[7] Hermes 配置"
if [ -f ~/.hermes/config.yaml ]; then
    echo -e "${GREEN}✓${NC} 配置文件存在: ~/.hermes/config.yaml"
else
    echo -e "${YELLOW}⚠${NC} 配置文件不存在，请运行: hermes init"
fi
echo ""

echo "=== 验证完成 ==="
EOFSCRIPT

chmod +x ~/verify_hermes.sh
```

**运行验证脚本**：

```bash
~/verify_hermes.sh
```

---

### 步骤 7.2：测试 Hermes Agent 基本功能

**测试对话功能**：

```bash
hermes chat "Hello, please confirm you are working correctly and tell me your current model."
```

**测试文件操作**：

```bash
hermes chat "Create a test file named test_hermes.txt with content 'Hermes Agent is working!' and show me the file content."
```

**测试代码执行**：

```bash
hermes chat "Write a Python script that prints system information (OS, Python version, current time) and execute it."
```

---

## 8. 远程管理配置（可选）

### 步骤 8.1：配置 SSH Server

**一键复制执行**：

```bash
sudo apt install -y openssh-server && \
sudo systemctl start ssh && \
sudo systemctl enable ssh && \
sudo systemctl status ssh
```

**增强 SSH 安全配置**：

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak && \
sudo tee -a /etc/ssh/sshd_config > /dev/null << 'EOF'

# Hermes Agent 远程管理安全配置
PermitRootLogin no
PasswordAuthentication yes
PubkeyAuthentication yes
X11Forwarding no
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
EOF

sudo systemctl restart ssh
```

**配置防火墙（如果使用 ufw）**：

```bash
sudo apt install -y ufw && \
sudo ufw allow 22/tcp && \
sudo ufw --force enable && \
sudo ufw status
```

---

### 步骤 8.2：配置 Telegram Bot（可选）

**安装 Telegram 集成**：

```bash
hermes setup telegram
```

根据提示输入：
- Telegram Bot Token（从 @BotFather 获取）
- 允许的用户 ID

**测试 Telegram 连接**：

```bash
hermes telegram test
```

---

## 9. 一键安装脚本

### 完整自动化安装脚本

**创建安装脚本**：

```bash
cat > ~/install_hermes_debian12.sh << 'EOFINSTALL'
#!/bin/bash

set -e

echo "=== Debian 12 Hermes Agent 一键安装脚本 ==="
echo ""

# 颜色定义
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
NC='\033[0m'

# 检查是否为 Debian 12
if ! grep -q "bookworm" /etc/os-release; then
    echo -e "${RED}错误：此脚本仅支持 Debian 12 (bookworm)${NC}"
    exit 1
fi

echo -e "${GREEN}[1/6] 更新系统并安装基础工具...${NC}"
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget ca-certificates gnupg lsb-release locales tzdata vim nano unzip zip tar gzip bzip2 xz-utils

echo -e "${GREEN}[2/6] 安装 Git...${NC}"
sudo apt install -y git
git config --global init.defaultBranch main

echo -e "${GREEN}[3/6] 安装 Python 3.11...${NC}"
sudo apt install -y python3 python3-pip python3-venv python3-dev python3-setuptools python3-wheel python3-full
python3 -m pip install --upgrade pip --break-system-packages
pip3 install --user --break-system-packages setuptools wheel virtualenv pipx
python3 -m pipx ensurepath

echo -e "${GREEN}[4/6] 安装 Node.js 22 LTS...${NC}"
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
npm install -g npm@latest

echo -e "${GREEN}[5/6] 安装 Hermes Agent...${NC}"
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash

echo -e "${GREEN}[6/6] 配置环境...${NC}"
source ~/.bashrc

echo ""
echo -e "${GREEN}=== 安装完成！===${NC}"
echo ""
echo "请运行以下命令完成配置："
echo "  source ~/.bashrc"
echo "  hermes init"
echo ""
echo "验证安装："
echo "  hermes --version"
echo ""

EOFINSTALL

chmod +x ~/install_hermes_debian12.sh
```

**运行一键安装脚本**：

```bash
~/install_hermes_debian12.sh
```

---

## 10. 故障排查

### 常见问题

#### 问题 1：Git 版本过旧

**解决方案**：
```bash
sudo apt update && sudo apt install -y git
git --version
```

#### 问题 2：Python pip 安装失败

**解决方案**：
```bash
python3 -m pip install --upgrade pip --break-system-packages
```

#### 问题 3：Node.js 安装失败

**解决方案**：
```bash
# 清理旧的 NodeSource 仓库
sudo rm -f /etc/apt/sources.list.d/nodesource.list
sudo apt update

# 重新安装
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
```

#### 问题 4：Hermes 命令找不到

**解决方案**：
```bash
source ~/.bashrc
which hermes
```

如果仍然找不到：
```bash
cd ~/hermes-agent
sudo ln -sf $(pwd)/bin/hermes /usr/local/bin/hermes
```

#### 问题 5：权限错误

**解决方案**：
```bash
# 修复 npm 权限
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# 修复 Python 权限
pip3 install --user --break-system-packages <package-name>
```

---

### 日志查看

**查看 Hermes 日志**：

```bash
tail -f ~/.hermes/logs/hermes.log
```

**查看系统日志**：

```bash
sudo journalctl -u hermes -f
```

---

### 完全卸载

**卸载 Hermes Agent**：

```bash
rm -rf ~/hermes-agent
rm -rf ~/.hermes
sudo rm -f /usr/local/bin/hermes
```

**卸载 Node.js**：

```bash
sudo apt remove -y nodejs
sudo rm -f /etc/apt/sources.list.d/nodesource.list
sudo apt update
```

---

## 📚 参考资源

- **Hermes Agent 官方文档**：https://hermes-agent.nousresearch.com/docs
- **Hermes Agent GitHub**：https://github.com/NousResearch/hermes-agent
- **Debian 官方文档**：https://www.debian.org/doc/
- **Node.js 官方文档**：https://nodejs.org/docs/
- **Python 官方文档**：https://docs.python.org/3/

---

## 🎉 完成

恭喜！你已经在 Debian 12 上成功安装了 Hermes Agent。

**下一步**：
1. 运行 `hermes init` 配置 API 密钥
2. 运行 `hermes chat "Hello"` 测试对话功能
3. 查看官方文档了解更多功能

**获取帮助**：
```bash
hermes --help
hermes chat --help
hermes config --help
```

---

**最后更新**：2026-05-09  
**版本**：v2.0 - 直接安装版（无编译）
