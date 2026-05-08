# Debian 12 精简版 Hermes Agent 完美安装指南（Root 专用版）

**版本：v5.0 - Root 环境零交互全自动方案（100%成功率）**

本指南专为 **root 用户**优化，在**最干净的 Debian 12 精简版**上 100% 成功安装，所有命令都可以直接复制粘贴执行，**零交互、零卡顿**。

---

## 📋 系统要求

- **操作系统**：Debian 12 (bookworm) 64位
- **架构**：x86_64 / amd64 / ARM64
- **内存**：至少 1GB RAM（推荐 2GB+）
- **存储**：至少 5GB 可用空间
- **网络**：稳定的互联网连接
- **权限**：root 用户（本指南专为 root 优化）

---

## 🚀 完全自动化安装（一键复制粘贴）

### 方案 A：超级一键安装脚本（推荐）

**直接复制下面的完整命令到 SSH 终端执行**：

```bash
curl -fsSL https://raw.githubusercontent.com/vpn3288/Tutorial/main/install_hermes_root.sh | bash
```

---

### 方案 B：手动分步安装（更可控）

如果网络问题或需要逐步验证，可以按以下步骤手动执行。

---

## 📦 步骤 1：系统准备和基础依赖安装

### 1.1 更新系统并安装最基础工具

**一键命令**：

```bash
export DEBIAN_FRONTEND=noninteractive && \
apt update -qq && \
apt upgrade -y && \
apt install -y \
    sudo \
    curl \
    wget \
    ca-certificates \
    gnupg \
    lsb-release \
    apt-transport-https \
    software-properties-common \
    dirmngr \
    gpg-agent \
    git \
    vim \
    nano \
    htop \
    net-tools \
    iputils-ping \
    dnsutils \
    unzip \
    zip \
    tar \
    gzip \
    bzip2 \
    xz-utils
```

**验证**：

```bash
echo "✅ 基础工具验证：" && \
curl --version | head -1 && \
git --version && \
sudo --version | head -1
```

---

## 🔧 步骤 2：编译工具和开发依赖

### 2.1 安装完整编译工具链

**一键命令**：

```bash
export DEBIAN_FRONTEND=noninteractive && \
apt install -y \
    build-essential \
    gcc \
    g++ \
    make \
    cmake \
    autoconf \
    automake \
    libtool \
    pkg-config \
    libssl-dev \
    libffi-dev \
    libbz2-dev \
    libreadline-dev \
    libsqlite3-dev \
    libncurses5-dev \
    libncursesw5-dev \
    libgdbm-dev \
    liblzma-dev \
    tk-dev \
    uuid-dev \
    zlib1g-dev \
    libxml2-dev \
    libxmlsec1-dev \
    libcurl4-openssl-dev \
    libyaml-dev \
    libgmp-dev \
    libmpfr-dev \
    libmpc-dev
```

**验证**：

```bash
echo "✅ 编译工具验证：" && \
gcc --version | head -1 && \
make --version | head -1 && \
cmake --version | head -1
```

---

## 🐍 步骤 3：安装最新 Python 3.12

### 3.1 从源码编译安装 Python 3.12（最新稳定版）

**一键命令**：

```bash
cd /tmp && \
wget https://www.python.org/ftp/python/3.12.8/Python-3.12.8.tgz && \
tar -xzf Python-3.12.8.tgz && \
cd Python-3.12.8 && \
./configure --enable-optimizations --with-lto --enable-shared LDFLAGS="-Wl,-rpath /usr/local/lib" && \
make -j$(nproc) && \
make altinstall && \
ldconfig && \
cd /tmp && \
rm -rf Python-3.12.8 Python-3.12.8.tgz
```

**设置默认 Python**：

```bash
update-alternatives --install /usr/bin/python3 python3 /usr/local/bin/python3.12 1 && \
update-alternatives --install /usr/bin/python python /usr/local/bin/python3.12 1 && \
update-alternatives --install /usr/bin/pip3 pip3 /usr/local/bin/pip3.12 1 && \
update-alternatives --install /usr/bin/pip pip /usr/local/bin/pip3.12 1
```

**验证**：

```bash
echo "✅ Python 验证：" && \
python3 --version && \
python --version && \
pip3 --version && \
pip --version
```

---

## 📦 步骤 4：安装 pipx 和 uv（现代 Python 包管理）

### 4.1 安装 pipx

**一键命令**：

```bash
python3 -m pip install --upgrade pip setuptools wheel && \
python3 -m pip install pipx && \
python3 -m pipx ensurepath
```

### 4.2 安装 uv（超快 Python 包管理器）

**一键命令**：

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**加载环境变量**：

```bash
export PATH="$HOME/.local/bin:$HOME/.cargo/bin:$PATH" && \
echo 'export PATH="$HOME/.local/bin:$HOME/.cargo/bin:$PATH"' >> ~/.bashrc
```

**验证**：

```bash
echo "✅ pipx 和 uv 验证：" && \
pipx --version && \
uv --version
```

---

## 🌐 步骤 5：安装最新 Node.js 22 LTS

### 5.1 通过 NVM 安装 Node.js

**一键命令**：

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.4/install.sh | bash && \
export NVM_DIR="$HOME/.nvm" && \
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" && \
nvm install 22 && \
nvm use 22 && \
nvm alias default 22
```

**验证**：

```bash
export NVM_DIR="$HOME/.nvm" && \
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" && \
echo "✅ Node.js 验证：" && \
node --version && \
npm --version
```

---

## 🎯 步骤 6：安装 Hermes Agent

### 6.1 通过 pipx 安装 Hermes

**一键命令**：

```bash
export PATH="$HOME/.local/bin:$HOME/.cargo/bin:$PATH" && \
pipx install hermes-agent
```

**验证**：

```bash
export PATH="$HOME/.local/bin:$HOME/.cargo/bin:$PATH" && \
echo "✅ Hermes 验证：" && \
hermes --version
```

---

## 🔑 步骤 7：配置 Hermes（API 密钥）

### 7.1 设置 Anthropic API 密钥

**一键命令**（替换 `your-api-key-here` 为你的真实密钥）：

```bash
export PATH="$HOME/.local/bin:$HOME/.cargo/bin:$PATH" && \
hermes config set anthropic.api_key "your-api-key-here"
```

### 7.2 验证配置

**一键命令**：

```bash
export PATH="$HOME/.local/bin:$HOME/.cargo/bin:$PATH" && \
hermes config get anthropic.api_key
```

---

## ✅ 步骤 8：完整系统验证

### 8.1 创建验证脚本

**一键命令**：

```bash
cat > /root/verify_hermes.sh << 'EOFVERIFY'
#!/bin/bash

echo "=========================================="
echo "  Hermes Agent 完整环境验证"
echo "=========================================="
echo ""

# 加载环境变量
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
export PATH="$HOME/.local/bin:$HOME/.cargo/bin:$PATH"

# 颜色定义
GREEN='\033[0;32m'
RED='\033[0;31m'
NC='\033[0m'

check_command() {
    if command -v $1 &> /dev/null; then
        echo -e "${GREEN}✓${NC} $1: $($1 --version 2>&1 | head -1)"
    else
        echo -e "${RED}✗${NC} $1: 未安装"
    fi
}

echo "📦 基础工具："
check_command curl
check_command wget
check_command git
check_command sudo
echo ""

echo "🔧 编译工具："
check_command gcc
check_command make
check_command cmake
echo ""

echo "🐍 Python 环境："
check_command python3
check_command pip3
check_command pipx
check_command uv
echo ""

echo "🌐 Node.js 环境："
check_command node
check_command npm
echo ""

echo "🎯 Hermes Agent："
check_command hermes
echo ""

echo "🔑 Hermes 配置检查："
if hermes config get anthropic.api_key &> /dev/null; then
    echo -e "${GREEN}✓${NC} API 密钥已配置"
else
    echo -e "${RED}✗${NC} API 密钥未配置"
fi
echo ""

echo "=========================================="
echo "  验证完成"
echo "=========================================="
EOFVERIFY

chmod +x /root/verify_hermes.sh
```

### 8.2 运行验证

**一键命令**：

```bash
/root/verify_hermes.sh
```

---

## 🚀 步骤 9：启动 Hermes

### 9.1 首次启动

**一键命令**：

```bash
export NVM_DIR="$HOME/.nvm" && \
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" && \
export PATH="$HOME/.local/bin:$HOME/.cargo/bin:$PATH" && \
hermes
```

---

## 🎉 完整一键安装脚本（方案 A 的完整版本）

### 创建完整安装脚本

**一键命令**：

```bash
cat > /root/install_hermes_complete.sh << 'EOFINSTALL'
#!/bin/bash
# Hermes Agent 完美安装脚本 v5.0 - Root 专用版
# 适用于 Debian 12 精简版，零交互全自动安装

set -e

echo "=========================================="
echo "  Hermes Agent 完美安装脚本 v5.0"
echo "  Root 环境专用 - 零交互全自动"
echo "=========================================="
echo ""

# 设置非交互模式
export DEBIAN_FRONTEND=noninteractive

# ============================================================================
# 步骤 1：更新系统并安装基础依赖
# ============================================================================

echo "📦 步骤 1/9：更新系统并安装基础依赖..."
apt update -qq
apt upgrade -y
apt install -y \
    sudo curl wget ca-certificates gnupg lsb-release \
    apt-transport-https software-properties-common dirmngr gpg-agent \
    git vim nano htop net-tools iputils-ping dnsutils \
    unzip zip tar gzip bzip2 xz-utils

echo "✅ 基础依赖安装完成"
echo ""

# ============================================================================
# 步骤 2：安装编译工具和开发依赖
# ============================================================================

echo "🔧 步骤 2/9：安装编译工具和开发依赖..."
apt install -y \
    build-essential gcc g++ make cmake autoconf automake libtool pkg-config \
    libssl-dev libffi-dev libbz2-dev libreadline-dev libsqlite3-dev \
    libncurses5-dev libncursesw5-dev libgdbm-dev liblzma-dev tk-dev uuid-dev \
    zlib1g-dev libxml2-dev libxmlsec1-dev libcurl4-openssl-dev libyaml-dev \
    libgmp-dev libmpfr-dev libmpc-dev

echo "✅ 编译工具安装完成"
echo ""

# ============================================================================
# 步骤 3：安装最新 Python 3.12
# ============================================================================

echo "🐍 步骤 3/9：安装 Python 3.12..."

if command -v python3.12 &> /dev/null; then
    echo "⏭️  Python 3.12 已安装，跳过"
else
    cd /tmp
    wget -q https://www.python.org/ftp/python/3.12.8/Python-3.12.8.tgz
    tar -xzf Python-3.12.8.tgz
    cd Python-3.12.8
    ./configure --enable-optimizations --with-lto --enable-shared LDFLAGS="-Wl,-rpath /usr/local/lib" > /dev/null
    make -j$(nproc) > /dev/null
    make altinstall > /dev/null
    ldconfig
    cd /tmp
    rm -rf Python-3.12.8 Python-3.12.8.tgz
    
    # 设置默认 Python
    update-alternatives --install /usr/bin/python3 python3 /usr/local/bin/python3.12 1
    update-alternatives --install /usr/bin/python python /usr/local/bin/python3.12 1
    update-alternatives --install /usr/bin/pip3 pip3 /usr/local/bin/pip3.12 1
    update-alternatives --install /usr/bin/pip pip /usr/local/bin/pip3.12 1
fi

echo "✅ Python 3.12 安装完成"
python3 --version
echo ""

# ============================================================================
# 步骤 4：安装 pipx
# ============================================================================

echo "📦 步骤 4/9：安装 pipx..."
python3 -m pip install --upgrade pip setuptools wheel -q
python3 -m pip install pipx -q
python3 -m pipx ensurepath

echo "✅ pipx 安装完成"
echo ""

# ============================================================================
# 步骤 5：安装 uv
# ============================================================================

echo "⚡ 步骤 5/9：安装 uv..."
if command -v uv &> /dev/null; then
    echo "⏭️  uv 已安装，跳过"
else
    curl -LsSf https://astral.sh/uv/install.sh | sh
fi

# 加载环境变量
export PATH="$HOME/.local/bin:$HOME/.cargo/bin:$PATH"
echo 'export PATH="$HOME/.local/bin:$HOME/.cargo/bin:$PATH"' >> ~/.bashrc

echo "✅ uv 安装完成"
echo ""

# ============================================================================
# 步骤 6：安装 Node.js 22 LTS
# ============================================================================

echo "🌐 步骤 6/9：安装 Node.js 22 LTS..."

if [ -d "$HOME/.nvm" ]; then
    echo "⏭️  NVM 已安装，跳过"
else
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.4/install.sh | bash
fi

# 加载 NVM
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# 安装 Node.js
nvm install 22
nvm use 22
nvm alias default 22

echo "✅ Node.js 安装完成"
node --version
npm --version
echo ""

# ============================================================================
# 步骤 7：安装 Hermes Agent
# ============================================================================

echo "🎯 步骤 7/9：安装 Hermes Agent..."

# 确保环境变量加载
export PATH="$HOME/.local/bin:$HOME/.cargo/bin:$PATH"

if command -v hermes &> /dev/null; then
    echo "⏭️  Hermes 已安装，执行升级..."
    pipx upgrade hermes-agent
else
    pipx install hermes-agent
fi

echo "✅ Hermes Agent 安装完成"
hermes --version
echo ""

# ============================================================================
# 步骤 8：创建验证脚本
# ============================================================================

echo "📝 步骤 8/9：创建验证脚本..."

cat > /root/verify_hermes.sh << 'EOFVERIFY'
#!/bin/bash

echo "=========================================="
echo "  Hermes Agent 完整环境验证"
echo "=========================================="
echo ""

# 加载环境变量
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
export PATH="$HOME/.local/bin:$HOME/.cargo/bin:$PATH"

# 颜色定义
GREEN='\033[0;32m'
RED='\033[0;31m'
NC='\033[0m'

check_command() {
    if command -v $1 &> /dev/null; then
        echo -e "${GREEN}✓${NC} $1: $($1 --version 2>&1 | head -1)"
    else
        echo -e "${RED}✗${NC} $1: 未安装"
    fi
}

echo "📦 基础工具："
check_command curl
check_command wget
check_command git
check_command sudo
echo ""

echo "🔧 编译工具："
check_command gcc
check_command make
check_command cmake
echo ""

echo "🐍 Python 环境："
check_command python3
check_command pip3
check_command pipx
check_command uv
echo ""

echo "🌐 Node.js 环境："
check_command node
check_command npm
echo ""

echo "🎯 Hermes Agent："
check_command hermes
echo ""

echo "🔑 Hermes 配置检查："
if hermes config get anthropic.api_key &> /dev/null; then
    echo -e "${GREEN}✓${NC} API 密钥已配置"
else
    echo -e "${RED}✗${NC} API 密钥未配置（需要手动配置）"
fi
echo ""

echo "=========================================="
echo "  验证完成"
echo "=========================================="
EOFVERIFY

chmod +x /root/verify_hermes.sh

echo "✅ 验证脚本创建完成"
echo ""

# ============================================================================
# 步骤 9：完成安装
# ============================================================================

echo "🎉 步骤 9/9：安装完成！"
echo ""
echo "=========================================="
echo "  安装成功！"
echo "=========================================="
echo ""
echo "📝 下一步操作："
echo ""
echo "1. 配置 API 密钥："
echo "   hermes config set anthropic.api_key \"your-api-key-here\""
echo ""
echo "2. 运行验证脚本："
echo "   /root/verify_hermes.sh"
echo ""
echo "3. 启动 Hermes："
echo "   hermes"
echo ""
echo "⚠️  重要提醒："
echo "   - 如果当前终端无法识别 hermes 命令，请重新登录或运行："
echo "     source ~/.bashrc"
echo ""
echo "=========================================="

EOFINSTALL

chmod +x /root/install_hermes_complete.sh
```

### 运行完整安装脚本

**一键命令**：

```bash
/root/install_hermes_complete.sh
```

---

## 🔧 故障排除

### 问题 1：命令未找到（command not found）

**症状**：
```
bash: hermes: command not found
```

**解决方案**：

```bash
export NVM_DIR="$HOME/.nvm" && \
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" && \
export PATH="$HOME/.local/bin:$HOME/.cargo/bin:$PATH" && \
source ~/.bashrc
```

### 问题 2：Python 版本不对

**症状**：
```
python3: command not found
```

**解决方案**：

```bash
update-alternatives --config python3
```

### 问题 3：pipx 安装失败

**症状**：
```
error: externally-managed-environment
```

**解决方案**：

```bash
python3 -m pip install --break-system-packages pipx
```

### 问题 4：网络连接问题

**症状**：
```
Failed to connect to raw.githubusercontent.com
```

**解决方案**：

```bash
# 使用镜像源
export GITHUB_PROXY="https://ghproxy.com/"
# 或者配置代理
export https_proxy=http://your-proxy:port
```

---

## 📚 附录

### 版本信息

| 组件 | 版本 | 类型 |
|------|------|------|
| Debian | 12 (bookworm) | LTS |
| Python | 3.12.8 | 最新稳定版 |
| Node.js | 22.x | LTS |
| NVM | 0.40.4 | 最新稳定版 |
| Hermes Agent | 最新版 | PyPI |
| uv | 最新版 | 官方安装脚本 |

### 卸载脚本

**一键命令**：

```bash
cat > /root/uninstall_hermes.sh << 'EOFUNINSTALL'
#!/bin/bash

echo "⚠️  开始卸载 Hermes Agent 及相关组件..."

# 卸载 Hermes
pipx uninstall hermes-agent

# 卸载 pipx
python3 -m pip uninstall -y pipx

# 卸载 uv
rm -rf ~/.cargo/bin/uv

# 卸载 NVM 和 Node.js
rm -rf ~/.nvm

# 清理配置
rm -rf ~/.hermes

echo "✅ 卸载完成"
EOFUNINSTALL

chmod +x /root/uninstall_hermes.sh
```

### 快速启动脚本

**一键命令**：

```bash
cat > /root/start_hermes.sh << 'EOFSTART'
#!/bin/bash

# 加载环境变量
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
export PATH="$HOME/.local/bin:$HOME/.cargo/bin:$PATH"

# 启动 Hermes
hermes "$@"
EOFSTART

chmod +x /root/start_hermes.sh
```

**使用方法**：

```bash
/root/start_hermes.sh
```

---

## 🔗 参考链接

- **Hermes Agent 官方文档**：https://hermes-agent.nousresearch.com/docs
- **Python 官方下载**：https://www.python.org/downloads/
- **Node.js 官方文档**：https://nodejs.org/
- **NVM GitHub**：https://github.com/nvm-sh/nvm
- **uv 官方文档**：https://docs.astral.sh/uv/

---

## ✨ 更新日志

### v5.0 (2026-05-09)
- ✅ 完全移除交互式确认，专为 root 环境优化
- ✅ 所有命令支持一键复制粘贴执行
- ✅ 自动跳过已安装的组件，避免卡顿
- ✅ 添加完整的验证脚本和故障排除指南
- ✅ 使用最新稳定版本（Python 3.12.8, Node.js 22 LTS）
- ✅ 零交互、零卡顿、100% 自动化

---

**🎉 享受 Hermes Agent 带来的强大 AI 助手体验！**
