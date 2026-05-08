# Debian 12 精简版 Hermes Agent 完整安装指南

本指南适用于**全新的 Debian 12 精简版系统**，所有命令均可**一键复制粘贴到 SSH 执行**，从零开始安装所有依赖和 Hermes Agent。

## 📋 目录
1. [系统准备与基础工具](#1-系统准备与基础工具)
2. [Git 最新版安装](#2-git-最新版安装)
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
- **架构**：x86_64 / amd64
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
    build-essential \
    software-properties-common \
    apt-transport-https \
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
    xz-utils \
    libssl-dev \
    libcurl4-openssl-dev \
    libexpat1-dev \
    gettext \
    zlib1g-dev \
    autoconf \
    pkg-config
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
wget --version && \
gcc --version
```

---

## 2. Git 最新版安装

### 步骤 2.1：从源码编译安装最新稳定版 Git

**一键复制执行**（安装 Git 2.48.1）：

```bash
cd /tmp && \
wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.48.1.tar.gz && \
tar -xzf git-2.48.1.tar.gz && \
cd git-2.48.1 && \
make prefix=/usr/local all && \
sudo make prefix=/usr/local install && \
cd ~ && \
rm -rf /tmp/git-*
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

### 步骤 4.1：使用 NVM 安装 Node.js 最新 LTS 版本

**一键复制执行**（安装 NVM 0.40.4 和 Node.js 22 LTS）：

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.4/install.sh | bash && \
export NVM_DIR="$HOME/.nvm" && \
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" && \
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion" && \
nvm install 22 && \
nvm use 22 && \
nvm alias default 22
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
verify_command gcc "gcc"
verify_command make "make"
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

---

### 步骤 8.2：配置防火墙

**一键复制执行**：

```bash
sudo apt install -y ufw && \
sudo ufw default deny incoming && \
sudo ufw default allow outgoing && \
sudo ufw allow ssh && \
sudo ufw allow 22/tcp && \
sudo ufw --force enable && \
sudo ufw status verbose
```

---

### 步骤 8.3：创建 Hermes Agent 系统服务

**一键复制执行**：

```bash
sudo tee /etc/systemd/system/hermes-agent.service > /dev/null << EOF
[Unit]
Description=Hermes Agent Service
After=network.target

[Service]
Type=simple
User=$USER
WorkingDirectory=$HOME
ExecStart=/usr/local/bin/hermes serve
Restart=on-failure
RestartSec=10
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload && \
sudo systemctl enable hermes-agent && \
sudo systemctl start hermes-agent && \
sudo systemctl status hermes-agent
```

**查看服务日志**：

```bash
sudo journalctl -u hermes-agent -f
```

---

## 9. 一键安装脚本

### 完整自动化安装脚本

**一键复制执行**（创建安装脚本）：

```bash
cat > ~/install_hermes_debian12.sh << 'EOFINSTALL'
#!/bin/bash

set -e

echo "=== Debian 12 Hermes Agent 一键安装脚本 ==="
echo "本脚本将安装所有依赖和 Hermes Agent"
echo ""

GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
NC='\033[0m'

# 检查是否为 root 或有 sudo 权限
if [ "$EUID" -ne 0 ] && ! command -v sudo &> /dev/null; then
    echo -e "${RED}错误：需要 root 权限或 sudo 命令${NC}"
    echo "请先运行: su - 然后执行 apt install sudo"
    exit 1
fi

SUDO=""
if [ "$EUID" -ne 0 ]; then
    SUDO="sudo"
fi

# 1. 更新系统并安装基础工具
echo -e "${YELLOW}[1/6] 更新系统并安装基础工具...${NC}"
$SUDO apt update && $SUDO apt upgrade -y
$SUDO apt install -y \
    curl wget build-essential software-properties-common \
    apt-transport-https ca-certificates gnupg lsb-release \
    vim nano unzip zip tar gzip libssl-dev libcurl4-openssl-dev \
    libexpat1-dev gettext zlib1g-dev autoconf pkg-config

# 2. 安装 Git 最新版
echo -e "${YELLOW}[2/6] 安装 Git 最新版...${NC}"
cd /tmp
wget -q https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.48.1.tar.gz
tar -xzf git-2.48.1.tar.gz
cd git-2.48.1
make prefix=/usr/local all > /dev/null 2>&1
$SUDO make prefix=/usr/local install > /dev/null 2>&1
cd ~
rm -rf /tmp/git-*
git --version

# 3. 安装 Python 3.11+
echo -e "${YELLOW}[3/6] 安装 Python 3.11+...${NC}"
$SUDO apt install -y python3 python3-pip python3-venv python3-dev \
    python3-setuptools python3-wheel python3-full
python3 -m pip install --upgrade pip --break-system-packages > /dev/null 2>&1
python3 --version
pip3 --version

# 4. 安装 Node.js 22 LTS
echo -e "${YELLOW}[4/6] 安装 Node.js 22 LTS...${NC}"
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.4/install.sh | bash > /dev/null 2>&1
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
nvm install 22 > /dev/null 2>&1
nvm use 22 > /dev/null 2>&1
nvm alias default 22 > /dev/null 2>&1
node --version
npm --version

# 5. 安装 Hermes Agent
echo -e "${YELLOW}[5/6] 安装 Hermes Agent...${NC}"
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash

# 6. 重新加载配置
echo -e "${YELLOW}[6/6] 重新加载 shell 配置...${NC}"
if [ -f ~/.bashrc ]; then
    source ~/.bashrc
fi

echo ""
echo -e "${GREEN}=== 安装完成 ===${NC}"
echo ""
echo "请运行以下命令验证安装："
echo "  hermes --version"
echo ""
echo "初始化 Hermes Agent："
echo "  hermes init"
echo ""
echo "如果 hermes 命令未找到，请重新加载 shell："
echo "  source ~/.bashrc"
echo "  或者重新登录 SSH"
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

### 问题 1：没有 sudo 命令

**症状**：
```
bash: sudo: command not found
```

**解决方案**：
```bash
su -
apt update && apt install -y sudo
usermod -aG sudo yourusername
exit
```

---

### 问题 2：Git 编译失败

**症状**：
```
make: *** [Makefile:xxx] Error 1
```

**解决方案**：
```bash
sudo apt install -y libssl-dev libcurl4-openssl-dev libexpat1-dev gettext zlib1g-dev autoconf
```

---

### 问题 3：Python pip 安装包失败

**症状**：
```
error: externally-managed-environment
```

**解决方案**：
```bash
# 使用 --break-system-packages 标志
pip3 install --user --break-system-packages package-name

# 或者使用虚拟环境
python3 -m venv ~/myenv
source ~/myenv/bin/activate
pip install package-name
```

---

### 问题 4：Node.js 版本不对

**症状**：
```
node --version
v18.x.x (< 22.0.0)
```

**解决方案**：
```bash
nvm install 22
nvm use 22
nvm alias default 22
```

---

### 问题 5：Hermes 安装脚本下载失败

**症状**：
```
curl: (7) Failed to connect to raw.githubusercontent.com
```

**解决方案**：
```bash
# 方法 1：使用代理
export https_proxy=http://your-proxy:port
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash

# 方法 2：手动克隆仓库
git clone https://github.com/NousResearch/hermes-agent.git
cd hermes-agent
bash scripts/install.sh

# 方法 3：使用 GitHub 镜像
git clone https://mirror.ghproxy.com/https://github.com/NousResearch/hermes-agent.git
```

---

### 问题 6：hermes 命令未找到

**症状**：
```
bash: hermes: command not found
```

**解决方案**：
```bash
# 重新加载 shell 配置
source ~/.bashrc

# 或者重新登录 SSH
exit
# 然后重新连接

# 检查 hermes 是否在 PATH 中
which hermes

# 如果没有，手动添加符号链接
sudo ln -sf ~/hermes-agent/bin/hermes /usr/local/bin/hermes
```

---

### 查看日志

**Hermes Agent 日志**：
```bash
cat ~/.hermes/logs/hermes.log
tail -f ~/.hermes/logs/hermes.log
```

**系统日志**：
```bash
sudo journalctl -xe
sudo journalctl -u hermes-agent -f
```

**SSH 日志**：
```bash
sudo tail -f /var/log/auth.log
```

---

## 📚 附录

### A. 版本信息

| 组件 | 版本 | 说明 |
|------|------|------|
| Debian | 12 (bookworm) | 稳定版 |
| Git | 2.48.1 | 最新稳定版 |
| Python | 3.11.x | Debian 12 默认版本 |
| Node.js | 22.x LTS | 长期支持版本 |
| NVM | 0.40.4 | 最新版本 |
| Hermes Agent | latest | 从 GitHub main 分支 |

### B. 安装顺序总结

```
1. 系统准备（sudo, apt update）
   ↓
2. 基础工具（curl, wget, build-essential, 开发库）
   ↓
3. Git 2.48.1（源码编译）
   ↓
4. Python 3.11+（pip, venv, dev tools）
   ↓
5. Node.js 22 LTS（NVM 安装）
   ↓
6. Hermes Agent（官方脚本）
   ↓
7. 配置与验证
```

### C. 卸载脚本

**一键复制执行**（创建卸载脚本）：

```bash
cat > ~/uninstall_hermes.sh << 'EOFUNINSTALL'
#!/bin/bash

echo "=== Hermes Agent 卸载脚本 ==="
echo ""

read -p "确定要卸载 Hermes Agent 吗？(y/N) " -n 1 -r
echo
if [[ ! $REPLY =~ ^[Yy]$ ]]; then
    echo "取消卸载"
    exit 0
fi

# 备份配置
if [ -d ~/.hermes ]; then
    echo "备份配置到 ~/.hermes.backup..."
    cp -r ~/.hermes ~/.hermes.backup.$(date +%Y%m%d_%H%M%S)
fi

# 停止服务
sudo systemctl stop hermes-agent 2>/dev/null
sudo systemctl disable hermes-agent 2>/dev/null
sudo rm -f /etc/systemd/system/hermes-agent.service
sudo systemctl daemon-reload

# 删除 Hermes
rm -rf ~/.hermes
sudo rm -f /usr/local/bin/hermes
rm -rf ~/hermes-agent

# 清理 shell 配置
sed -i '/hermes/d' ~/.bashrc
sed -i '/HERMES/d' ~/.bashrc

echo "✓ Hermes Agent 已卸载"
echo "配置备份位于: ~/.hermes.backup.*"

EOFUNINSTALL

chmod +x ~/uninstall_hermes.sh
```

### D. 参考链接

- [Hermes Agent GitHub](https://github.com/NousResearch/hermes-agent)
- [Hermes Agent 官方文档](https://hermes-agent.nousresearch.com/docs)
- [Debian 官方文档](https://www.debian.org/doc/)
- [Git 官方网站](https://git-scm.com/)
- [Python 官方网站](https://www.python.org/)
- [Node.js 官方网站](https://nodejs.org/)
- [NVM GitHub](https://github.com/nvm-sh/nvm)

---

## 🎯 快速开始（TL;DR）

如果你想最快速度安装，只需复制以下命令：

```bash
# 1. 安装 sudo（如果没有）
su -
apt update && apt install -y sudo
usermod -aG sudo yourusername
exit

# 2. 运行一键安装脚本
curl -fsSL https://raw.githubusercontent.com/vpn3288/Tutorial/main/install_hermes_debian12.sh | bash

# 3. 重新加载配置
source ~/.bashrc

# 4. 初始化 Hermes
hermes init

# 5. 验证安装
hermes --version
```

---

**最后更新**：2026年5月8日  
**适用版本**：Debian 12 (bookworm)  
**维护者**：Hermes Agent 社区  
**许可证**：MIT

---

## ✅ 安装检查清单

- [ ] sudo 已安装并配置
- [ ] 系统已更新（apt update && apt upgrade）
- [ ] 基础工具已安装（curl, wget, gcc, make）
- [ ] Git 2.48.1+ 已安装
- [ ] Python 3.11+ 已安装
- [ ] Node.js 22+ LTS 已安装
- [ ] Hermes Agent 已安装
- [ ] Hermes 配置已初始化（hermes init）
- [ ] API 密钥已配置
- [ ] 验证脚本测试通过
- [ ] SSH 远程访问已配置（可选）
- [ ] 防火墙已配置（可选）
- [ ] systemd 服务已创建（可选）

---

**祝你使用愉快！如有问题，请参考故障排查章节或提交 Issue。**
