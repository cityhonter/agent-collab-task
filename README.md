# 任务中转站 (Zhongzhuan Task)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-2.0.0-blue.svg)](CHANGELOG.md)
[![Obsidian](https://img.shields.io/badge/Obsidian-compatible-purple.svg)](https://obsidian.md)

> 双角色跨平台 agent 协作协议 —— 让多个 AI agent 接力完成同一组任务。

通过约定一个 **Obsidian vault 中的"中转层"目录**，让任何能读 Markdown 文件的 AI agent（不限平台）协作完成同一组任务，不重复、不遗漏、不撞车。

---

## ✨ 这是什么？

**一个纯 Markdown 协议，没有任何代码依赖。** 核心思想：

```
用户 ──(说需求)──> 【制定者角色】──写入──> 中转层(任务文件) <──读取── 【执行者角色】──(干活)──> 用户
                                              ↑_____________________回报状态/结果________________↓
```

- 📝 **制定任务**：一个人/agent 创建结构化任务文件
- 🎯 **认领执行**：另一个 agent 看到任务后上锁、执行、回报
- 🔄 **无缝接力**：跨平台、跨 agent、跨 session，状态不丢失
- 📊 **可视化追踪**：`当前关注.md` 一页看清所有任务状态

---

## 🚀 30 秒上手

### 1. 准备工作

- 一个 **Obsidian vault**（任意位置）
- 在 vault 内创建"中转层"目录（默认 `~/Documents/中转层/`）

### 2. 复制协议文件

```bash
# 从 GitHub 克隆
git clone https://github.com/cityhonter/agent-collab-task.git
cd agent-collab-task

# 复制模板到你的中转层
cp -r templates/* <你的中转层>/templates/
cp examples/当前关注.md.example <你的中转层>/当前关注.md
```

### 3. 在你的 agent 中配置

参考 [config.example.md](config.example.md)，告诉 agent 你的 `TRANSIT_ROOT` 路径。

### 4. 开始协作

对 agent 说：

> "制定中转任务：实现 xx 功能"

agent 会自动创建任务文件、加入索引。其他 agent 说"执行中转任务"就能接力。

---

## 🎯 典型场景

| 场景 | 怎么用 |
|------|------|
| **WorkBuddy 规划 + Cursor 执行** | 在 WorkBuddy 里规划任务，让 Cursor IDE agent 执行 |
| **多 agent 分工** | 主 agent 规划，备用 agent 做 code review / 测试 |
| **跨 session 协作** | 任务文件存本地，session 关闭后状态不丢 |
| **跨平台搬运** | 在 ChatGPT 规划 → Claude 执行 → WorkBuddy 验收 |
| **外包任务交接** | 客户用 WorkBuddy 看任务进度，开发用 Cursor 执行 |

---

## 📦 目录结构

```
中转层/
├── 当前关注.md              # 索引页（一页看清所有任务）
├── code/
│   ├── active/              # 代码类任务（待执行）
│   └── done/                # 已完成
├── other/
│   ├── active/              # 其他类任务
│   └── done/
├── templates/               # 任务模板
└── archive/                 # 归档
```

---

## 🛠 支持的 agent 平台

| 平台 | 接入方式 | 文档 |
|------|---------|------|
| **WorkBuddy** | 装 skill，触发词生效 | [安装指南](docs/安装指南.md#workbuddy) |
| **Cursor** | 把 SKILL.md 贴到 `.cursorrules` | [Cursor 适配](docs/跨平台兼容.md#cursor) |
| **Claude Code** | 装到 `~/.claude/skills/` | [Claude 适配](docs/跨平台兼容.md#claude-code) |
| **ChatGPT** | 贴到"自定义指令" | [ChatGPT 适配](docs/跨平台兼容.md#chatgpt) |
| **通用 agent** | 读协议文档照规范做 | [协议详解](docs/协议详解.md) |

---

## 📖 核心概念（5 分钟理解）

### 双角色

- **制定者（Planner）**：创建任务、定义交付物
- **执行者（Executor）**：认领任务、执行、回报

### 任务状态机

```
pending ─claim→ claimed ─done→ done ─周日归档→ archived
   │              │
   │              └超时（默认24h）→ 回到 pending
   └用户取消→ cancelled
```

### 锁规则

- claim 前**必读** `claimed_by` 字段
- 非空且未超时 → **不许碰**
- 超时 → 可接管，注明接管者

详细文档：[docs/协议详解.md](docs/协议详解.md)

---

## ⚙️ 配置

最小配置（必填）：

```json
{
  "TRANSIT_ROOT": "~/Documents/中转层",
  "AGENT_ID": "codeBuddy-阿宝"
}
```

完整配置项：[config.example.md](config.example.md)

---

## 🤝 贡献

欢迎贡献：

- 🐛 [报告问题](https://github.com/cityhonter/agent-collab-task/issues/new)
- 💡 [提新功能](https://github.com/cityhonter/agent-collab-task/issues/new)
- 🔧 [提交 PR](https://github.com/cityhonter/agent-collab-task/pulls)
- 📝 改进文档

详见 [贡献指南](.github/ISSUE_TEMPLATE/)（待补充）。

---

## 📄 License

MIT © cityhonter（cityhonter）

---

## 🌟 致谢

本协议受以下项目启发：

- [Obsidian](https://obsidian.md) — 提供 vault 基础设施
- [notesmd-cli](https://github.com/Yakitrak/notesmd-cli) — 命令行操作 Obsidian vault
- 所有在多 agent 协作领域探索的前辈

---

_如果这个项目对你有帮助，给个 ⭐ Star 是最大的鼓励。_