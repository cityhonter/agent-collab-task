---
name: zhongzhuan-task
display_name: 任务中转站
description: 双角色跨平台 agent 协作协议 —— 通过 Obsidian vault 中的"中转层"目录，让多个 AI agent（WorkBuddy、Cursor、Claude Code、ChatGPT、任何能读文件的 agent）协作完成同一组任务，不重复、不遗漏、不撞车。
version: 2.0.0
license: MIT
author: cityhonter（cityhonter）
homepage: https://github.com/cityhonter/agent-collab-task
keywords: [obsidian, multi-agent, task-management, knowledge-base, ai-agents]
config_required: true
---

# 任务中转站（Zhongzhuan Task）

> 双角色跨平台 agent 协作协议 —— 用 Obsidian vault 作为共享状态，让多个 AI agent 接力完成同一组任务。

---

## 这是什么？

一个**纯 Markdown 协议**，没有任何代码依赖。通过约定一个共享文件夹（中转层），让任何能读 Markdown 的 agent（不限平台）都能：

- 📝 **制定任务**：一个人/agent 创建结构化任务文件
- 🎯 **认领执行**：另一个 agent 看到任务后上锁、执行、回报
- 🔄 **无缝接力**：跨平台、跨 agent、跨 session，状态不丢失
- 📊 **可视化追踪**：`当前关注.md` 一页看清所有任务状态

**典型场景：**

- 在 WorkBuddy 里规划任务，让本地 Cursor IDE agent 执行
- 主 agent 负责规划，备用 agent 负责 code review
- 一人 + 多 AI agent 分工协作同一项目
- 在不同 AI 平台之间搬运任务，不丢上下文

---

## 用户配置（必填）

> 第一次使用本 skill 时，agent 会用 AskUserQuestion 收集这些配置，存到 `~/.zhongzhuan-task/config.json`。
> 后续可通过修改该文件调整配置。

| 变量 | 必填 | 说明 | 默认值 |
|------|------|------|--------|
| `TRANSIT_ROOT` | ✅ | 中转层根目录绝对路径 | `~/Documents/中转层` |
| `VAULT_NAME` | ⬜ | 关联的 Obsidian vault 名称（可选） | （自动检测）|
| `CLAIM_TIMEOUT_HOURS` | ⬜ | claim 超时小时数（超时未更新可被接管）| `24` |
| `AGENT_ID` | ⬜ | 当前 agent 唯一标识（写入 frontmatter 区分认领者）| `agent-{hostname}` |

**路径占位符约定**（本文档中所有路径都遵循）：

```
{TRANSIT_ROOT}/当前关注.md          # 索引页
{TRANSIT_ROOT}/code/active/*.md     # 代码类任务（待执行）
{TRANSIT_ROOT}/code/done/*.md       # 代码类任务（已完成）
{TRANSIT_ROOT}/other/active/*.md    # 其他类任务（待执行）
{TRANSIT_ROOT}/other/done/*.md      # 其他类任务（已完成）
{TRANSIT_ROOT}/templates/           # 任务模板目录
{TRANSIT_ROOT}/archive/             # 归档目录
```

---

## 触发词

### 角色一：制定者（Planner）

任一即触发，进入制定者流程：

- "制定中转任务：xxx"
- "新建中转任务"
- "加个中转任务"
- "往中转层加任务"
- "更新中转任务：xxx"（更新已有任务内容）
- "取消中转任务：xxx"

### 角色二：执行者（Executor）

任一即触发，进入执行者流程：

- "执行中转任务"
- "领任务" / "领取任务"
- "看下中转层有什么任务"
- "继续做中转层的任务"
- "中转任务状态"

### 跨角色通用

- "配置中转任务路径" — 重新配置 TRANSIT_ROOT
- "查看中转任务配置" — 显示当前配置

---

## 路径约定

| 项 | 路径（用户配置后实际展开）|
|---|---|
| 中转层根目录 | `{TRANSIT_ROOT}` |
| 索引页 | `{TRANSIT_ROOT}/当前关注.md` |
| 代码类任务（待执行）| `{TRANSIT_ROOT}/code/active/{标题}.md` |
| 代码类任务（已完成）| `{TRANSIT_ROOT}/code/done/{标题}.md` |
| 其他类任务（待执行）| `{TRANSIT_ROOT}/other/active/{标题}.md` |
| 其他类任务（已完成）| `{TRANSIT_ROOT}/other/done/{标题}.md` |
| 任务模板 | `{TRANSIT_ROOT}/templates/代码任务.md` 等 |
| 归档 | `{TRANSIT_ROOT}/archive/YYYY-WW.md` |

---

## 任务状态机

```
pending ──claim──> claimed ──done──> done ──周日归档──> archived
   │                  │
   │                  └─超时(CLAIM_TIMEOUT_HOURS)无人问─> 回到 pending（可重新 claim）
   └─用户取消────────────────────> cancelled
```

**frontmatter 状态字段：**

```yaml
status: pending      # pending / claimed / done / blocked / cancelled
claimed_by: ""       # 哪个 agent 认领
claimed_at: ""       # 认领时间 ISO 格式，用于超时判断
result: ""           # 完成后的一句话结果摘要
```

**锁规则：**

1. claim 前必读 `claimed_by`。非空且 `claimed_at` 在 `CLAIM_TIMEOUT_HOURS` 小时内 → **不许碰**，换下一个任务
2. `claimed_by` 非空但超过 `CLAIM_TIMEOUT_HOURS` 小时无更新 → 视为超时，可覆盖 claim（写明自己是接管者）
3. claim 成功 → 立即写 `claimed_by` + `claimed_at` + `status: claimed`，**先锁再做**

---

# 角色一：制定者（Planner）

## 工作流

### Step 1：识别意图

| 用户说法 | 动作 |
|---|---|
| "制定中转任务：xxx" | 创建新任务 |
| "更新中转任务：xxx" | 先列 `code/active/` 和 `other/active/` 找到文件再改 |
| "取消中转任务：xxx" | status 改 cancelled，说明原因 |

### Step 2：判断任务类型

- **code（代码类）**：写代码、做功能、修 bug、配置环境、写脚本 → `code/active/`
- **other（其他类）**：学习、求职、沟通、运营、写文档、调研 → `other/active/`

**用户没说清类型，必须追问一次，不要默认。**

### Step 3：收集必要信息

**code 任务必填：** `project`、`priority`、`output`、`done_criteria`（≤3 条可验证）

**other 任务必填：** `category`、`priority`、`output`、`done_criteria`

**可选：** `blocked_by`、`deadline`

### Step 4：写入任务文件

1. `Read` 对应模板（`{TRANSIT_ROOT}/templates/代码任务.md` 或 `其他任务.md`）
2. 填充字段，新任务初始 `status: pending`，`claimed_by` 留空
3. `Write` 到 `{TRANSIT_ROOT}/code/active/{动宾结构标题}.md` 或 `other/active/...`

### Step 5：更新索引

`Edit` `{TRANSIT_ROOT}/当前关注.md`：

- 在对应类型表格加一行
- 更新 frontmatter 的 `updated` 和 `total_active`

### Step 6：回报用户

```
✅ 任务已创建
- 类型：code / other
- 优先级：P1
- 路径：<绝对路径>
- 完成标准：N 条
- 任何 agent 可通过"执行中转任务"领取
```

---

# 角色二：执行者（Executor）

## 工作流

### Step 1：读索引

`Read` `{TRANSIT_ROOT}/当前关注.md`，拿到任务列表、优先级、状态。

用户没指定任务时：按 P0 > P1 > P2 > P3 选优先级最高的 **pending** 任务；全部被锁则报告当前 claim 情况让用户定。

### Step 2：检查锁

打开目标任务文件，检查 frontmatter：

- `claimed_by` 为空 → 可以 claim
- `claimed_by` 非空且 `CLAIM_TIMEOUT_HOURS` 小时内 → 报告"该任务已被 {claimed_by} 领取"，换任务或问用户
- `claimed_by` 非空但超时 → 可接管，注明接管

### Step 3：上锁

立即 `Edit` 任务文件 frontmatter：

```yaml
status: claimed
claimed_by: "<当前 AGENT_ID，如 codeBuddy-阿宝>"
claimed_at: "<当前时间 ISO>"
```

### Step 4：通读任务全文

确认理解五要素：**目标 / 背景 / 输入 / 输出 / 完成标准**。

- 缺信息 → 问用户，**不要瞎猜**
- `blocked_by` 非空 → 报告卡点，不要硬做

### Step 5：执行任务

按任务文件的输出/交付物要求干活。

- 改动涉及文件时用任务文件里写的**绝对路径**
- 不要越界做"不要做"清单里的事

### Step 6：完成回报（关键，给下一个 agent 看）

执行完后 `Edit` 任务文件：

```yaml
status: done
result: "<一句话：做了什么，产出在哪>"
updated: "<日期>"
```

同时在文件末尾追加：

```markdown
## 执行记录
- 执行者：<AGENT_ID>
- 完成时间：<时间>
- 结果：<做了什么、产出路径、遗留问题>
```

### Step 7：更新索引 + 归档

- `Edit` `{TRANSIT_ROOT}/当前关注.md`：状态列改 ✅ 或移入"最近完成"
- 把任务文件从 `active/` 移到对应 `done/`

---

## 通用规则

### DO

- ✅ 缺信息就追问，**绝不替用户瞎填**
- ✅ 任务标题用动宾结构（"实现xxx" / "整理xxx"）
- ✅ 完成标准必须**可验证**（agent 能判定"做完了"）
- ✅ 所有文件路径用**绝对路径**
- ✅ 每次 claim / 完成 / 更新都同步改 `{TRANSIT_ROOT}/当前关注.md` 索引
- ✅ 写入文件用 UTF-8 编码

### DON'T

- ❌ 不要跳过锁检查直接干活（会跟其他 agent 撞车）
- ❌ 不要在 `当前关注.md` 放任务详情，只放指针
- ❌ 不要把多个任务塞进一个文件
- ❌ 不要执行"不要做"清单里的事
- ❌ 不要修改任务后不更新索引
- ❌ 不要对已 claimed 且未超时的任务动手

---

## 异常处理

| 场景 | 处理 |
|------|------|
| 配置文件不存在 | 触发本 skill 时主动 AskUserQuestion 收集配置，写入 `~/.zhongzhuan-task/config.json` |
| TRANSIT_ROOT 不存在 | `mkdir -p` 创建完整目录结构（参考 `templates/` 下的模板）|
| 用户没说任务类型 | 追问 |
| 用户描述太模糊 | 追问目标和完成标准 |
| 索引页不存在 | 按 `templates/当前关注.md.template` 创建 |
| 同名任务已存在 | 询问是更新还是新建 |
| 目标任务被别人 claim 且未超时 | 换任务或报告用户 |
| 任务 `blocked_by` 非空 | 报告卡点，问用户是否强行继续 |

---

## 跨平台安装

支持 5 种安装方式，详见 [docs/跨平台兼容.md](docs/跨平台兼容.md)：

1. **WorkBuddy** — 装到 `~/.workbuddy/skills/`，触发词生效
2. **Cursor** — 把 SKILL.md 内容贴到 `.cursorrules`
3. **Claude Code** — 装到 `~/.claude/skills/`
4. **ChatGPT** — 贴到"自定义指令"，告知中转层路径
5. **通用 agent** — 读 `docs/协议详解.md`，照规范做

---

## 协作流程示意

```
用户 ──(说需求)──> 【制定者角色】──写入──> 中转层(任务文件)
                                              ↑
                                    <──读取── 【执行者角色】
                                              │
                                              ↓
                                        (干活)──> 用户
                                              │
                                              └──回报状态/结果──┐
                                                                ↓
                                                    更新到任务文件 + 索引页
```

---

## 文档导航

| 文档 | 内容 |
|------|------|
| [README.md](README.md) | 项目介绍、快速开始 |
| [docs/安装指南.md](docs/安装指南.md) | 5 平台安装详细步骤 |
| [docs/跨平台兼容.md](docs/跨平台兼容.md) | 各平台适配说明 |
| [docs/协议详解.md](docs/协议详解.md) | 锁规则、状态机、异常处理完整说明 |
| [docs/常见问题.md](docs/常见问题.md) | FAQ |
| [config.example.md](config.example.md) | 用户配置示例 |
| [templates/](templates/) | 任务文件、索引页模板 |

---

*本协议不绑定任何平台、任何 agent。任何能读 Markdown 文件的工具都可参与。*
*License: MIT*