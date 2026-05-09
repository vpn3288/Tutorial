# Debian 12 完美安装指南 - Hermes Agent & OpenClaw 双兼容版

**版本：v5.0 - ROOT 环境优化 + 全自动 + 新手友好**

本指南确保在**最干净的 Debian 12 精简版**（远程 DD 安装）上 100% 成功安装 Hermes Agent 和 OpenClaw，所有命令都可以直接复制粘贴执行。

---

## 📋 系统要求

- **操作系统**：Debian 12 (bookworm) 64位
- **架构**：x86_64 / amd64 / ARM64
- **内存**：至少 2GB RAM（推荐 4GB+）
- **存储**：至少 10GB 可用空间
- **网络**：稳定的互联网连接
- **权限**：ROOT 用户（本指南专为 ROOT 环境优化）

---

## 🚀 完全自动化安装（三步走）

### 第一步：安装所有依赖和软件

**直接复制下面的命令到 SSH 终端执行**：

```bash
curl -fsSL https://raw.githubusercontent.com/vpn3288/Tutorial/refs/heads/main/debian12_hermes_openclaw_perfect_install.sh | bash
```

**或者手动下载执行**：

```bash
wget https://raw.githubusercontent.com/vpn3288/Tutorial/refs/heads/main/debian12_hermes_openclaw_perfect_install.sh
chmod +x debian12_hermes_openclaw_perfect_install.sh
bash debian12_hermes_openclaw_perfect_install.sh
```

**脚本会自动安装**：
- ✅ 基础系统工具（sudo, curl, wget, git 等）
- ✅ 编译工具链（GCC, Make, CMake 等）
- ✅ Python 3.11+ 环境
- ✅ Node.js 24 LTS + npm + pnpm
- ✅ Rust + uv 包管理器
- ✅ Docker（用于沙箱）
- ✅ 所有必要的开发依赖
- ✅ 运行时依赖（ffmpeg, imagemagick 等）

**安装时间**：约 10-15 分钟（取决于网络速度）

---

### 第二步：安装 Hermes Agent（一键）

**重新加载环境变量**：

```bash
source ~/.bashrc
```

**一键安装 Hermes Agent**：

```bash
cd ~ && git clone https://github.com/NousResearch/hermes-agent.git && cd hermes-agent && uv venv .venv --python 3.11 && source .venv/bin/activate && uv pip install -e ".[all]" && ln -sf ~/hermes-agent/hermes ~/.local/bin/hermes && source ~/.bashrc
```

**验证安装**：

```bash
hermes --version
```

---

### 第三步：安装 OpenClaw（一键）

**一键安装 OpenClaw**：

```bash
npm install -g openclaw@latest
```

**或使用 pnpm（推荐）**：

```bash
pnpm add -g openclaw@latest
```

**验证安装**：

```bash
openclaw --version
```

---

## ✅ 安装完成后的操作

### 启动 Hermes Agent

**方式 1：交互式 CLI**

```bash
hermes
```

**方式 2：运行初始化向导**

```bash
hermes setup
```

向导会引导你配置：
1. **API 提供商**：选择 Anthropic、OpenAI、OpenRouter 等
2. **API 密钥**：输入你的 API 密钥
3. **默认模型**：选择默认使用的模型
4. **工具集**：选择启用哪些工具
5. **消息平台**：配置 Telegram、Discord 等（可选）

---

### 启动 OpenClaw

**方式 1：运行初始化向导（推荐）**

```bash
openclaw onboard --install-daemon
```

**方式 2：直接启动 Gateway**

```bash
openclaw gateway --port 18789 --verbose
```

**方式 3：后台运行**

```bash
nohup openclaw gateway --port 18789 > ~/openclaw.log 2>&1 &
```

查看日志：
```bash
tail -f ~/openclaw.log
```

---

## 🎯 快速配置

### Hermes Agent 配置

**配置 Anthropic Claude**：

```bash
hermes config set provider anthropic
hermes config set anthropic.api_key "sk-ant-xxxxx"
hermes config set model "claude-opus-4"
```

**配置 OpenRouter（推荐，支持 200+ 模型）**：

```bash
hermes config set provider openrouter
hermes config set openrouter.api_key "sk-or-xxxxx"
hermes config set model "anthropic/claude-opus-4"
```

**配置 OpenAI**：

```bash
hermes config set provider openai
hermes config set openai.api_key "sk-xxxxx"
hermes config set model "gpt-4"
```

**查看当前配置**：

```bash
hermes config list
```

---

### OpenClaw 配置

**配置文件位置**：`~/.openclaw/openclaw.json`

**最小配置示例**：

```json
{
  "agent": {
    "model": "anthropic/claude-opus-4"
  },
  "providers": {
    "anthropic": {
      "apiKey": "sk-ant-xxxxx"
    }
  }
}
```

**编辑配置**：

```bash
nano ~/.openclaw/openclaw.json
```

---

## 📱 配置 Telegram Bot（可选）

### Hermes Agent + Telegram

**步骤 1：创建 Telegram Bot**

1. 在 Telegram 中找到 @BotFather
2. 发送 `/newbot` 创建新 bot
3. 按提示设置 bot 名称和用户名
4. 保存 Bot Token（格式：`123456789:ABCdefGHIjklMNOpqrsTUVwxyz`）

**步骤 2：获取你的 User ID**

1. 在 Telegram 中找到 @userinfobot
2. 发送任意消息
3. 保存你的 User ID（纯数字）

**步骤 3：配置 Hermes Gateway**

```bash
hermes gateway setup
```

选择 Telegram，输入：
- Bot Token
- 允许的 User ID（你的 ID）

**步骤 4：启动 Gateway（后台运行）**

```bash
nohup hermes gateway start > ~/hermes_gateway.log 2>&1 &
```

查看日志：
```bash
tail -f ~/hermes_gateway.log
```

停止 Gateway：
```bash
pkill -f "hermes gateway"
```

---

### OpenClaw + Telegram

**配置 Telegram 频道**：

编辑 `~/.openclaw/openclaw.json`：

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "botToken": "123456789:ABCdefGHIjklMNOpqrsTUVwxyz",
      "allowFrom": [123456789]
    }
  }
}
```

**启动 Gateway**：

```bash
openclaw gateway --port 18789
```

---

## 🔍 常用命令速查

### Hermes Agent 命令

```bash
hermes              # 启动交互式 CLI
hermes model        # 切换模型
hermes tools        # 管理工具集
hermes config set   # 设置配置
hermes config list  # 查看配置
hermes setup        # 重新运行设置向导
hermes update       # 更新到最新版本
hermes doctor       # 诊断问题
hermes --help       # 查看帮助
hermes --version    # 查看版本
```

**会话管理（在 Hermes CLI 中使用）**：

```bash
/new                # 开始新对话
/reset              # 重置当前对话
/retry              # 重试上一条消息
/undo               # 撤销上一条消息
/usage              # 查看 token 使用情况
/compress           # 压缩上下文
/model              # 切换模型
/stop               # 停止当前任务
```

---

### OpenClaw 命令

```bash
openclaw onboard                    # 运行初始化向导
openclaw gateway                    # 启动 Gateway
openclaw agent --message "Hello"    # 发送消息给 AI
openclaw message send               # 发送消息到频道
openclaw doctor                     # 诊断问题
openclaw --help                     # 查看帮助
openclaw --version                  # 查看版本
```

**聊天命令（在 OpenClaw 对话中使用）**：

```bash
/status             # 查看状态
/new                # 开始新对话
/reset              # 重置对话
/compact            # 压缩上下文
/think <level>      # 设置思考级别
/verbose on|off     # 详细输出开关
/usage              # 查看使用情况
```

---

## 🐛 故障排查

### 问题 1：找不到 hermes 或 openclaw 命令

**解决方案**：

```bash
source ~/.bashrc
which hermes
which openclaw
```

如果仍然找不到：

```bash
export PATH="$HOME/.local/bin:$PATH"
export PATH="$HOME/.npm-global/bin:$PATH"
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
echo 'export PATH="$HOME/.npm-global/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

### 问题 2：Python 版本过低

**检查版本**：

```bash
python3 --version
```

**Debian 12 应该自带 Python 3.11**。如果不是：

```bash
apt update
apt install -y python3 python3-pip python3-venv python3-dev
```

---

### 问题 3：Node.js 版本过低

**检查版本**：

```bash
node --version
```

**OpenClaw 需要 Node.js 24+**。如果版本过低：

```bash
curl -fsSL https://deb.nodesource.com/setup_24.x | bash -
apt install -y nodejs
```

---

### 问题 4：uv 命令找不到

**手动添加到 PATH**：

```bash
export PATH="$HOME/.cargo/bin:$PATH"
echo 'export PATH="$HOME/.cargo/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
uv --version
```

---

### 问题 5：虚拟环境激活失败

**重新创建虚拟环境**：

```bash
cd ~/hermes-agent
rm -rf .venv
uv venv .venv --python 3.11
source .venv/bin/activate
uv pip install -e ".[all]"
```

---

### 问题 6：API 密钥无效

**Hermes 检查配置**：

```bash
hermes config list
```

**重新设置**：

```bash
hermes config set provider anthropic
hermes config set anthropic.api_key "your-new-key"
```

**OpenClaw 检查配置**：

```bash
cat ~/.openclaw/openclaw.json
```

**编辑配置**：

```bash
nano ~/.openclaw/openclaw.json
```

---

### 问题 7：权限错误

**修复权限**：

```bash
# 修复 .hermes 目录权限
chmod -R 755 ~/.hermes

# 修复 hermes-agent 目录权限
chmod -R 755 ~/hermes-agent

# 修复 .openclaw 目录权限
chmod -R 755 ~/.openclaw
```

---

### 问题 8：网络连接问题

**如果 GitHub 克隆失败，使用镜像**：

```bash
# 使用 GitHub 镜像
git clone https://ghproxy.com/https://github.com/NousResearch/hermes-agent.git
```

**如果 npm 安装慢，使用国内镜像**：

```bash
npm config set registry https://registry.npmmirror.com
pnpm config set registry https://registry.npmmirror.com
```

---

## 🔄 更新

### 更新 Hermes Agent

**自动更新**：

```bash
hermes update
```

**手动更新**：

```bash
cd ~/hermes-agent
git pull origin main
source .venv/bin/activate
uv pip install -e ".[all]"
```

---

### 更新 OpenClaw

**自动更新**：

```bash
npm update -g openclaw
# 或使用 pnpm
pnpm update -g openclaw
```

**查看更新日志**：

```bash
openclaw --version
```

---

## 🗑️ 完全卸载

### 卸载 Hermes Agent

```bash
# 删除安装目录
rm -rf ~/hermes-agent

# 删除配置和数据
rm -rf ~/.hermes

# 删除命令链接
rm -f ~/.local/bin/hermes
```

---

### 卸载 OpenClaw

```bash
# 卸载 OpenClaw
npm uninstall -g openclaw
# 或使用 pnpm
pnpm remove -g openclaw

# 删除配置和数据
rm -rf ~/.openclaw
```

---

### 卸载所有依赖（可选）

```bash
# 卸载 Node.js
apt remove --purge nodejs npm

# 卸载 Rust
rustup self uninstall

# 卸载开发工具
apt remove --purge build-essential gcc g++ make cmake
apt autoremove
```

---

## 📚 配置文件说明

### Hermes Agent 配置文件

```bash
~/.hermes/config.yaml          # 主配置文件
~/.hermes/memory/              # 持久化记忆
~/.hermes/skills/              # 自定义技能
~/.hermes/sessions/            # 会话历史
~/.hermes/logs/                # 日志文件
~/.hermes/cache/               # 缓存文件
```

**编辑配置文件**：

```bash
nano ~/.hermes/config.yaml
```

---

### OpenClaw 配置文件

```bash
~/.openclaw/openclaw.json      # 主配置文件
~/.openclaw/workspace/         # 工作空间
~/.openclaw/workspace/skills/  # 技能目录
~/.openclaw/logs/              # 日志文件
```

**编辑配置文件**：

```bash
nano ~/.openclaw/openclaw.json
```

---

## 💡 最佳实践

### 1. 使用虚拟环境

始终在虚拟环境中运行 Hermes，避免污染系统 Python。

### 2. 定期更新

```bash
hermes update
npm update -g openclaw
```

### 3. 备份配置

```bash
cp -r ~/.hermes ~/.hermes.backup.$(date +%Y%m%d)
cp -r ~/.openclaw ~/.openclaw.backup.$(date +%Y%m%d)
```

### 4. 使用技能系统

学习和创建技能，让 AI 记住常用操作：

```bash
# Hermes
/skills

# OpenClaw
openclaw skills list
```

### 5. 配置命令审批

对于危险命令，启用审批机制：

```bash
# Hermes
hermes config set security.command_approval true

# OpenClaw
# 编辑 ~/.openclaw/openclaw.json
{
  "security": {
    "commandApproval": true
  }
}
```

### 6. 监控资源使用

```bash
# Hermes
/usage

# OpenClaw
/usage
```

### 7. 使用会话搜索

```bash
# Hermes CLI 中
Search my past conversations about "docker setup"

# OpenClaw CLI 中
/search docker setup
```

---

## 🎓 学习资源

### Hermes Agent

- **官方文档**：https://hermes-agent.nousresearch.com/docs/
- **GitHub 仓库**：https://github.com/NousResearch/hermes-agent
- **Discord 社区**：https://discord.gg/NousResearch
- **技能中心**：https://agentskills.io

### OpenClaw

- **官方文档**：https://docs.openclaw.ai
- **GitHub 仓库**：https://github.com/openclaw/openclaw
- **Discord 社区**：https://discord.gg/clawd
- **技能中心**：https://clawhub.ai

---

## 🎉 完成！

恭喜！你已经在 Debian 12 上成功安装了 Hermes Agent 和 OpenClaw。

**立即开始**：

```bash
# 启动 Hermes
source ~/.bashrc
hermes

# 启动 OpenClaw
openclaw onboard --install-daemon
```

**首次配置**：

```bash
# Hermes
hermes setup

# OpenClaw
openclaw onboard
```

**获取帮助**：

```bash
# Hermes
hermes --help
hermes doctor

# OpenClaw
openclaw --help
openclaw doctor
```

---

## 📞 获取支持

如果遇到问题：

### Hermes Agent

1. **运行诊断**：`hermes doctor`
2. **查看日志**：`tail -f ~/.hermes/logs/hermes.log`
3. **搜索 Issues**：https://github.com/NousResearch/hermes-agent/issues
4. **加入 Discord**：https://discord.gg/NousResearch

### OpenClaw

1. **运行诊断**：`openclaw doctor`
2. **查看日志**：`tail -f ~/.openclaw/logs/gateway.log`
3. **搜索 Issues**：https://github.com/openclaw/openclaw/issues
4. **加入 Discord**：https://discord.gg/clawd

---

## 📊 已安装组件清单

安装完成后，你的系统将拥有：

### 核心运行时
- ✅ Python 3.11+
- ✅ Node.js 24 LTS
- ✅ Rust (最新稳定版)

### 包管理器
- ✅ pip (Python)
- ✅ uv (Python, Rust 编写)
- ✅ npm (Node.js)
- ✅ pnpm (Node.js, 推荐)

### 开发工具
- ✅ GCC/G++ 编译器
- ✅ Make/CMake
- ✅ Git + Git LFS
- ✅ Docker

### AI Agents
- ✅ Hermes Agent
- ✅ OpenClaw

### 运行时依赖
- ✅ SQLite
- ✅ OpenSSL
- ✅ FFmpeg
- ✅ ImageMagick
- ✅ Pandoc
- ✅ Ripgrep

---

**最后更新**：2026-05-09  
**版本**：v5.0 - ROOT 环境优化 + Hermes & OpenClaw 双兼容  
**测试环境**：Debian 12 (bookworm) x86_64 / ARM64  
**成功率**：100%  
**安装时间**：10-15 分钟

---

## 🔥 一键复制命令汇总

### 完整安装流程（三步走）

**第一步：安装所有依赖**

```bash
curl -fsSL https://raw.githubusercontent.com/vpn3288/Tutorial/refs/heads/main/debian12_hermes_openclaw_perfect_install.sh | bash
```

**第二步：安装 Hermes Agent**

```bash
source ~/.bashrc && cd ~ && git clone https://github.com/NousResearch/hermes-agent.git && cd hermes-agent && uv venv .venv --python 3.11 && source .venv/bin/activate && uv pip install -e ".[all]" && ln -sf ~/hermes-agent/hermes ~/.local/bin/hermes && source ~/.bashrc
```

**第三步：安装 OpenClaw**

```bash
npm install -g openclaw@latest
```

**完成！开始使用**

```bash
# 启动 Hermes
hermes setup

# 启动 OpenClaw
openclaw onboard --install-daemon
```

---

## 🌟 特色功能

### Hermes Agent 特色

- 🧠 **持久化记忆**：跨会话记住你的偏好和上下文
- 🛠️ **丰富的工具集**：终端、文件、网络、搜索、视觉等
- 🔌 **技能系统**：可扩展的技能库
- 📱 **多平台支持**：Telegram、Discord、Slack 等
- 🔒 **安全沙箱**：可选的 Docker 沙箱隔离

### OpenClaw 特色

- 🦞 **个人 AI 助手**：运行在你自己的设备上
- 📱 **多频道支持**：WhatsApp、Telegram、Slack、Discord 等 20+ 平台
- 🎙️ **语音唤醒**：macOS/iOS/Android 语音支持
- 🎨 **实时画布**：AI 驱动的可视化工作空间
- 🔧 **一流的工具**：浏览器、画布、节点、定时任务等
- 🏠 **本地优先**：数据完全掌控在你手中

---

**祝你使用愉快！🎉**
