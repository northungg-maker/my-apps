# OpenClaw 迁移指南 - 新账户隔离方案

## 预计时间：30-60分钟

---

## 第一步：创建新账户（在 Mac 系统设置里做）

1. 打开 **系统设置** → **用户与群组**
2. 点击左下角 **锁图标** 解锁（输入密码）
3. 点击 **+** 添加用户
4. 选择：
   - **账户类型：** 标准用户（不要选管理员）
   - **全名：** OpenClaw（或你喜欢的名字）
   - **账户名称：** openclaw
   - **密码：** 设置一个你能记住的密码
5. 点击 **创建用户**

---

## 第二步：备份现有数据（在当前账户做）

### 2.1 显示隐藏文件夹

Finder 默认不显示隐藏文件夹（以 `.` 开头的文件夹）。显示方法：

- 打开 Finder
- 按 `Command + Shift + .`（句号键）
- 再按一次可以隐藏回去

### 2.2 需要备份的文件夹

**位置：** 你的家目录下有一个 `.openclaw` 文件夹

| 你看到的 | 真实路径 | 内容 |
|----------|----------|------|
| `.openclaw` | `/Users/tolo/.openclaw/` | 所有记忆、配置、工作文件 |

**里面有这些重要东西：**

```
.openclaw/
├── workspace/          ← 你的记忆文件、工作文档
│   ├── MEMORY.md       ← 长期记忆
│   ├── memory/         ← 日志
│   ├── SOUL.md         ← 我的人格设定
│   └── ...
├── openclaw.json       ← 主配置文件
├── agents/             ← 代理配置
└── logs/               ← 日志（不重要，可以不备份）
```

### 2.3 备份方法

**方法一：用终端命令（推荐，最简单）**

打开终端，复制粘贴这三行：

```bash
# 创建备份目录
mkdir -p ~/Desktop/openclaw_backup

# 复制整个 .openclaw 目录到桌面备份
cp -r ~/.openclaw ~/Desktop/openclaw_backup/
```

执行完后，桌面上会出现 `openclaw_backup` 文件夹，里面有完整备份。

**方法二：用 Finder 手动复制**

1. 打开 Finder
2. 按 `Command + Shift + G`（前往文件夹）
3. 输入 `~/.openclaw` 回车
4. 选中 `.openclaw` 文件夹
5. 按 `Command + C` 复制
6. 去桌面，按 `Command + V` 粘贴

---

## 第三步：切换到新账户

1. 点击屏幕右上角的用户名（或苹果菜单 → 退出登录）
2. 登录到刚才创建的 **openclaw** 账户
3. 完成首次登录设置

---

## 第四步：在新账户安装依赖

新账户是一个"干净"的环境，需要重新安装 Node.js 和 OpenClaw。

### 4.1 安装 Homebrew

打开终端，粘贴：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

按提示操作，等待安装完成。

### 4.2 安装 Node.js

```bash
brew install node
```

### 4.3 安装 OpenClaw

```bash
npm install -g openclaw
```

---

## 第五步：复制记忆文件

### 5.1 找到备份文件

你需要从主账户把备份文件传过来。有几个方法：

**方法一：使用共享文件夹**

在主账户执行：

```bash
# 复制备份到共享目录
cp -r ~/Desktop/openclaw_backup /Users/Shared/
```

然后切到新账户：

```bash
# 在新账户执行
cp -r /Users/Shared/openclaw_backup/.openclaw ~/.openclaw
```

**方法二：用 U 盘或 AirDrop**

把备份文件夹传到新账户。

### 5.2 放置文件

在新账户执行：

```bash
# 确保 .openclaw 目录存在
mkdir -p ~/.openclaw

# 复制 workspace（记忆文件）
cp -r /path/to/backup/.openclaw/workspace ~/.openclaw/
```

---

## 第六步：重新配置（必须做的）

### 6.1 重新生成配置文件

配置文件里的敏感凭证要重新生成。运行：

```bash
openclaw config
```

按提示重新配置一遍。

### 6.2 重新配置飞书

飞书的 webhook 和机器人在新账户要重新设置：

1. 把 `openclaw.json` 里的 App ID、App Secret 复制过去
2. 或者去飞书开放平台重新创建机器人

### 6.3 设置开机自启动

```bash
openclaw gateway install
```

---

## 第七步：验证

1. 启动 Gateway：`openclaw gateway start`
2. 连接 TUI：`openclaw tui`
3. 发一条消息测试：看记忆是不是还在

---

## 文件对照表

| 这是什么 | 在哪里 | 要不要动 |
|----------|--------|----------|
| 记忆文件 | `~/.openclaw/workspace/MEMORY.md` | ✅ 必须复制 |
| 日志文件 | `~/.openclaw/memory/` | ✅ 必须复制 |
| 人格设定 | `~/.openclaw/workspace/SOUL.md` | ✅ 必须复制 |
| 主配置 | `~/.openclaw/openclaw.json` | ⚠️ 复制后要改凭证 |
| LaunchAgents | `~/Library/LaunchAgents/ai.openclaw.*.plist` | ❌ 不用复制，新账户重新生成 |
| 日志 | `~/.openclaw/logs/` | ❌ 不用复制 |

---

## 常见问题

**Q: 怎么找到隐藏文件夹？**
A: 在 Finder 里按 `Command + Shift + .`

**Q: 备份在哪里？**
A: 我建议你备份到桌面 `~/Desktop/openclaw_backup/`，方便找到

**Q: 记忆会丢吗？**
A: 只要复制了 `workspace` 目录，记忆就不会丢

**Q: 如果出问题了怎么办？**
A: 你原来的账户和 `.openclaw` 都没动，回原账户继续用就行

---

## 执行顺序总结

1. ✅ 创建新账户（标准用户）
2. ✅ 显示隐藏文件（Command+Shift+.）
3. ✅ 备份 `.openclaw` 到桌面
4. ✅ 切换到新账户
5. ✅ 安装 Homebrew、Node.js、OpenClaw
6. ✅ 复制备份的 `workspace` 目录
7. ✅ 重新运行 `openclaw config`
8. ✅ 重新配置飞书
9. ✅ 设置开机自启动
10. ✅ 测试验证

---

**准备好了告诉我，我帮你一步步执行。**
