# OpenClaw 双实例迁移指南

## 目标

将两个实例从当前账户迁移到新建的隔离账户：
- **主实例**（我）：端口 19790
- **xiaomeng 实例**：端口 19785

迁移后两个实例独立运行，互不干扰，和现在一样。

---

## 迁移总览

| 实例 | 当前路径 | 新账户路径 | 端口 |
|------|----------|------------|------|
| 主实例 | `~/.openclaw/` | `~/.openclaw/` | 19790 |
| xiaomeng | `~/.openclaw-xiaomeng/` | `~/.openclaw-xiaomeng/` | 19785 |

---

## 第一步：创建新账户

在 **系统设置** → **用户与群组**：
- 账户类型：**标准用户**（不是管理员）
- 用户名：`openclaw`
- 密码：你自己设置

---

## 第二步：备份两个实例（在当前账户）

打开终端，运行：

```bash
# 创建备份目录
mkdir -p ~/Desktop/openclaw_migrate_backup

# 备份主实例
cp -r ~/.openclaw ~/Desktop/openclaw_migrate_backup/openclaw

# 备份 xiaomeng 实例
cp -r ~/.openclaw-xiaomeng ~/Desktop/openclaw_migrate_backup/openclaw-xiaomeng

# 备份 LaunchAgents（开机启动项）
cp ~/Library/LaunchAgents/ai.openclaw.gateway.plist ~/Desktop/openclaw_migrate_backup/
cp ~/Library/LaunchAgents/ai.openclaw.xiaomeng.plist ~/Desktop/openclaw_migrate_backup/

# 查看备份结果
ls -la ~/Desktop/openclaw_migrate_backup/
```

执行完后，桌面上会有 `openclaw_migrate_backup` 文件夹，里面有：

```
openclaw_migrate_backup/
├── openclaw/              ← 主实例完整备份
│   ├── workspace/         ← 我的记忆
│   ├── openclaw.json      ← 主配置
│   └── ...
├── openclaw-xiaomeng/     ← xiaomeng实例完整备份
│   ├── workspace/         ← xiaomeng的记忆
│   ├── openclaw.json      ← xiaomeng配置
│   └── ...
├── ai.openclaw.gateway.plist   ← 主实例启动项
└── ai.openclaw.xiaomeng.plist  ← xiaomeng启动项
```

---

## 第三步：传输备份到共享位置

```bash
# 复制到系统共享目录（两个账户都能访问）
cp -r ~/Desktop/openclaw_migrate_backup /Users/Shared/
```

---

## 第四步：切换到新账户

1. 退出当前账户
2. 登录新建的 `openclaw` 账户

---

## 第五步：在新账户安装依赖

### 5.1 安装 Homebrew

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### 5.2 安装 Node.js

```bash
brew install node
```

### 5.3 安装 OpenClaw

```bash
npm install -g openclaw
```

---

## 第六步：恢复两个实例

在新账户执行：

```bash
# 恢复主实例
cp -r /Users/Shared/openclaw_migrate_backup/openclaw ~/.openclaw

# 恢复 xiaomeng 实例
cp -r /Users/Shared/openclaw_migrate_backup/openclaw-xiaomeng ~/.openclaw-xiaomeng

# 验证文件都在
ls ~/.openclaw/workspace/
ls ~/.openclaw-xiaomeng/workspace/
```

---

## 第七步：设置 LaunchAgents（开机自启动）

在新账户执行：

```bash
# 复制启动项配置
cp /Users/Shared/openclaw_migrate_backup/ai.openclaw.gateway.plist ~/Library/LaunchAgents/
cp /Users/Shared/openclaw_migrate_backup/ai.openclaw.xiaomeng.plist ~/Library/LaunchAgents/

# 加载启动项
launchctl load ~/Library/LaunchAgents/ai.openclaw.gateway.plist
launchctl load ~/Library/LaunchAgents/ai.openclaw.xiaomeng.plist
```

**注意：** plist 文件里的路径会自动指向新账户的家目录，不需要手动修改。

---

## 第八步：启动并验证

### 8.1 启动主实例

```bash
openclaw gateway start
```

### 8.2 启动 xiaomeng 实例

```bash
openclaw gateway start --config ~/.openclaw-xiaomeng/openclaw.json
```

### 8.3 验证两个实例都在运行

```bash
# 检查端口
lsof -i :19790  # 主实例
lsof -i :19785  # xiaomeng实例
```

### 8.4 连接测试

```bash
# 连接主实例
openclaw tui

# 连接 xiaomeng（另一个终端）
openclaw tui --gateway ws://127.0.0.1:19785
```

---

## 第九步：验证记忆完整

在主实例发送消息，检查记忆是否存在：
- MEMORY.md 的内容是否在
- 之前聊过的事情是否记得

在 xiaomeng 实例做同样测试。

---

## 双实例关键配置对照

| 配置项 | 主实例 | xiaomeng实例 |
|--------|--------|--------------|
| 目录 | `~/.openclaw/` | `~/.openclaw-xiaomeng/` |
| Gateway端口 | 19790 | 19785 |
| Browser端口 | 19792 | 19787 |
| Relay端口 | 19793 | 19788 |
| 工作区 | `workspace/` | `workspace/` |
| 启动文件 | `ai.openclaw.gateway.plist` | `ai.openclaw.xiaomeng.plist` |

---

## 可能需要重新配置的

| 项目 | 说明 |
|------|------|
| 飞书 webhook | 如果飞书后台配置的是本地回调地址，需要重新配置 |
| 敏感凭证 | 迁移后建议重新生成 API Key、Token 等 |
| 浏览器 | 两个实例各自独立的浏览器配置，新建账户后是空的 |

---

## 回滚方案

如果迁移失败，回到原账户：

```bash
# 1. 原账户的 .openclaw 和 .openclaw-xiaomeng 都没动
# 2. 直接在原账户使用即可
# 3. 备份文件在 ~/Desktop/openclaw_migrate_backup/
```

**迁移不会删除原账户的任何东西**，只是复制。所以回滚就是继续用原账户。

---

## 执行顺序清单

- [ ] 创建新账户（标准用户）
- [ ] 在当前账户运行备份命令
- [ ] 复制备份到 /Users/Shared/
- [ ] 切换到新账户
- [ ] 安装 Homebrew
- [ ] 安装 Node.js
- [ ] 安装 OpenClaw
- [ ] 复制主实例到 ~/.openclaw/
- [ ] 复制 xiaomeng实例到 ~/.openclaw-xiaomeng/
- [ ] 复制 LaunchAgents
- [ ] 加载 LaunchAgents
- [ ] 启动主实例
- [ ] 启动 xiaomeng 实例
- [ ] 验证端口
- [ ] 测试记忆

---

**准备好开始了吗？我一步步陪你做。**
