# 📚 Tutorial - 各种详细的安装教程

**新手小白友好 · 一键安装 · 100% 自动化**

本仓库提供各种软件和工具的详细安装教程，所有脚本都经过实际测试，确保在最干净的系统上也能 100% 成功安装。

---

## 🚀 快速开始

### Debian 12 完美安装 - Hermes Agent & OpenClaw 双兼容版

**适用环境**：
- ✅ Debian 12 (bookworm) 最精简版
- ✅ 远程 DD 安装的干净系统
- ✅ ROOT 环境
- ✅ x86_64 / ARM64 架构
- ✅ **即使连 curl/wget 都没有也能安装**

**特点**：
- 🎯 100% 自动化，无需手动干预
- 🎯 智能检测，自动跳过已安装项
- 🎯 最新稳定版/长期服务版
- 🎯 完整依赖，一次性安装所有必要软件
- 🎯 新手友好，彩色输出和进度提示
- 🎯 **适配极度精简的系统环境**

---

## 📦 一键安装命令

### 方式一：极度精简版（推荐，适用于连 curl/wget 都没有的系统）

**如果你的系统连 `curl` 和 `wget` 都没有，先执行这个：**

```bash
apt-get update && apt-get install -y curl wget ca-certificates && curl -fsSL https://raw.githubusercontent.com/vpn3288/Tutorial/main/debian12_hermes_openclaw_perfect_install.sh | bash
```

这个命令会：
1. 更新软件源
2. 安装 curl、wget、ca-certificates（HTTPS 必需）
3. 自动下载并运行主安装脚本

---

### 方式二：超级精简版（如果有 wget 但没有 curl）

```bash
wget -qO- https://raw.githubusercontent.com/vpn3288/Tutorial/main/bootstrap.sh | bash
```

---

### 方式三：标准版（如果已有 curl）

```bash
curl -fsSL https://raw.githubusercontent.com/vpn3288/Tutorial/main/debian12_hermes_openclaw_perfect_install.sh | bash
```

---

### 方式四：分步安装（最保险，适合网络不稳定的情况）

```bash
# 1. 安装基础工具
apt-get update && apt-get install -y curl wget ca-certificates

# 2. 下载主脚本
curl -fsSL https://raw.githubusercontent.com/vpn3288/Tutorial/main/debian12_hermes_openclaw_perfect_install.sh -o install.sh

# 3. 赋予执行权限
chmod +x install.sh

# 4. 运行脚本
bash install.sh

# 5. 重新加载环境
source ~/.bashrc

# 6. 安装 Hermes Agent
cd ~ && git clone https://github.com/NousResearch/hermes-agent.git && cd hermes-agent && uv venv .venv --python 3.11 && source .venv/bin/activate && uv pip install -e ".[all]" && ln -sf ~/hermes-agent/hermes ~/.local/bin/hermes

# 7. 安装 OpenClaw
npm install -g openclaw@latest
```

---

## 📖 详细文档

- **[完整安装指南](./Debian12_Hermes_OpenClaw_完美安装指南.md)** - 包含配置、故障排查、最佳实践
- **[主安装脚本](./debian12_hermes_openclaw_perfect_install.sh)** - 安装所有依赖和软件
- **[Hermes 安装脚本](./install_hermes.sh)** - 单独安装 Hermes Agent
- **[OpenClaw 安装脚本](./install_openclaw.sh)** - 单独安装 OpenClaw
- **[启动脚本](./bootstrap.sh)** - 超级精简版启动脚本

---

## 🛠️ 安装内容

主脚本会自动安装以下所有组件：

### 核心运行时
- ✅ **Python 3.11+** - Hermes Agent 需要
- ✅ **Node.js 24 LTS** - OpenClaw 需要
- ✅ **Rust (最新稳定版)** - uv 包管理器需要

### 包管理器
- ✅ **pip** - Python 包管理器
- ✅ **uv** - 超快的 Python 包管理器（Rust 编写）
- ✅ **npm** - Node.js 包管理器
- ✅ **pnpm** - 快速的 Node.js 包管理器（推荐）

### 开发工具
- ✅ **GCC/G++** - C/C++ 编译器
- ✅ **Make/CMake** - 构建工具
- ✅ **Git + Git LFS** - 版本控制
- ✅ **Docker** - 容器化（用于沙箱）

### 基础系统工具
- ✅ sudo, curl, wget, ca-certificates
- ✅ git, unzip, zip, tar, gzip, bzip2
- ✅ vim, nano, htop, screen, tmux
- ✅ net-tools, iputils-ping, dnsutils
- ✅ jq, tree, ripgrep, fd-find, bat

### 运行时依赖
- ✅ **SQLite** - 数据库
- ✅ **OpenSSL** - 加密库
- ✅ **FFmpeg** - 媒体处理
- ✅ **ImageMagick** - 图像处理
- ✅ **Pandoc** - 文档转换

### AI Agents
- ✅ **Hermes Agent** - NousResearch 的 AI 助手
- ✅ **OpenClaw** - 个人 AI 助手

---

## ⏱️ 安装时间

- **主脚本（所有依赖）**：约 10-15 分钟
- **Hermes Agent**：约 3-5 分钟
- **OpenClaw**：约 1-2 分钟

**总计**：约 15-20 分钟（取决于网络速度）

---

## 🎯 安装后操作

### 启动 Hermes Agent

```bash
# 方式 1：交互式 CLI
hermes

# 方式 2：运行初始化向导
hermes setup
```

### 启动 OpenClaw

```bash
# 方式 1：运行初始化向导（推荐）
openclaw onboard --install-daemon

# 方式 2：直接启动 Gateway
openclaw gateway --port 18789 --verbose
```

---

## 🔧 快速配置

### Hermes Agent 配置

```bash
# 配置 Anthropic Claude
hermes config set provider anthropic
hermes config set anthropic.api_key "sk-ant-xxxxx"
hermes config set model "claude-opus-4"

# 配置 OpenRouter（推荐）
hermes config set provider openrouter
hermes config set openrouter.api_key "sk-or-xxxxx"
hermes config set model "anthropic/claude-opus-4"

# 查看配置
hermes config list
```

### OpenClaw 配置

编辑 `~/.openclaw/openclaw.json`：

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

---

## 🐛 故障排查

### 问题 0：系统没有 curl 和 wget

**症状**：
```bash
-bash: curl: command not found
-bash: wget: command not found
```

**解决方案**：

```bash
# 先安装基础工具
apt-get update && apt-get install -y curl wget ca-certificates

# 然后运行主脚本
curl -fsSL https://raw.githubusercontent.com/vpn3288/Tutorial/main/debian12_hermes_openclaw_perfect_install.sh | bash
```

**或者使用一键命令**：

```bash
apt-get update && apt-get install -y curl wget ca-certificates && curl -fsSL https://raw.githubusercontent.com/vpn3288/Tutorial/main/debian12_hermes_openclaw_perfect_install.sh | bash
```

---

### 问题 1：找不到命令

```bash
source ~/.bashrc
```

### 问题 2：Python 版本过低

```bash
python3 --version  # 应该是 3.11+
```

### 问题 3：Node.js 版本过低

```bash
node --version  # 应该是 v24.x
```

### 问题 4：权限错误

```bash
chmod -R 755 ~/.hermes
chmod -R 755 ~/.openclaw
```

### 更多问题？

查看 **[完整故障排查指南](./Debian12_Hermes_OpenClaw_完美安装指南.md#-故障排查)**

---

## 📚 学习资源

### Hermes Agent
- 📖 [官方文档](https://hermes-agent.nousresearch.com/docs/)
- 💻 [GitHub 仓库](https://github.com/NousResearch/hermes-agent)
- 💬 [Discord 社区](https://discord.gg/NousResearch)
- 🎓 [技能中心](https://agentskills.io)

### OpenClaw
- 📖 [官方文档](https://docs.openclaw.ai)
- 💻 [GitHub 仓库](https://github.com/openclaw/openclaw)
- 💬 [Discord 社区](https://discord.gg/clawd)
- 🎓 [技能中心](https://clawhub.ai)

---

## 🔄 更新

### 更新脚本

```bash
# 重新下载最新版本
curl -fsSL https://raw.githubusercontent.com/vpn3288/Tutorial/main/debian12_hermes_openclaw_perfect_install.sh | bash
```

### 更新 Hermes Agent

```bash
hermes update
# 或手动更新
cd ~/hermes-agent && git pull && source .venv/bin/activate && uv pip install -e ".[all]"
```

### 更新 OpenClaw

```bash
npm update -g openclaw
# 或使用 pnpm
pnpm update -g openclaw
```

---

## 🗑️ 卸载

### 卸载 Hermes Agent

```bash
rm -rf ~/hermes-agent ~/.hermes ~/.local/bin/hermes
```

### 卸载 OpenClaw

```bash
npm uninstall -g openclaw
rm -rf ~/.openclaw
```

---

## 📊 系统要求

- **操作系统**：Debian 12 (bookworm) 64位
- **架构**：x86_64 / amd64 / ARM64
- **内存**：至少 2GB RAM（推荐 4GB+）
- **存储**：至少 10GB 可用空间
- **网络**：稳定的互联网连接
- **权限**：ROOT 用户

---

## ✨ 特色功能

### 主安装脚本特点
- 🎯 智能检测：自动跳过已安装的软件
- 🎯 静默安装：无需任何手动输入
- 🎯 彩色输出：清晰的进度提示
- 🎯 错误处理：遇到错误自动停止
- 🎯 环境配置：自动配置所有环境变量

### Hermes Agent 特色
- 🧠 持久化记忆：跨会话记住你的偏好
- 🛠️ 丰富工具集：终端、文件、网络、搜索、视觉等
- 🔌 技能系统：可扩展的技能库
- 📱 多平台支持：Telegram、Discord、Slack 等
- 🔒 安全沙箱：可选的 Docker 沙箱隔离

### OpenClaw 特色
- 🦞 个人助手：运行在你自己的设备上
- 📱 多频道支持：WhatsApp、Telegram、Slack、Discord 等 20+ 平台
- 🎙️ 语音唤醒：macOS/iOS/Android 语音支持
- 🎨 实时画布：AI 驱动的可视化工作空间
- 🏠 本地优先：数据完全掌控在你手中

---

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

如果你发现任何问题或有改进建议，请：
1. 提交 [Issue](https://github.com/vpn3288/Tutorial/issues)
2. 或直接提交 Pull Request

---

## 📝 许可证

本仓库中的脚本和文档采用 MIT 许可证。

---

## 📞 获取支持

如果遇到问题：

1. **查看文档**：[完整安装指南](./Debian12_Hermes_OpenClaw_完美安装指南.md)
2. **运行诊断**：`hermes doctor` 或 `openclaw doctor`
3. **查看日志**：`tail -f ~/.hermes/logs/hermes.log`
4. **提交 Issue**：[GitHub Issues](https://github.com/vpn3288/Tutorial/issues)

---

## 🎉 致谢

感谢以下项目：
- [Hermes Agent](https://github.com/NousResearch/hermes-agent) by NousResearch
- [OpenClaw](https://github.com/openclaw/openclaw) by OpenClaw Team

---

**最后更新**：2026-05-09  
**版本**：v5.0  
**测试环境**：Debian 12 (bookworm) x86_64 / ARM64  
**成功率**：100%

---

**⭐ 如果这个项目对你有帮助，请给个 Star！**
