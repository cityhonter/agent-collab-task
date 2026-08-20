# 配置文件示例

> 本文件展示「Agent 任务中转站」skill 的所有可配置项。
>
> 实际配置文件位置：`~/.zhongzhuan-task/config.json`

---

## 完整示例

```json
{
  "TRANSIT_ROOT": "D:\\文档\\workSpace\\中转层",
  "VAULT_NAME": "workSpace",
  "CLAIM_TIMEOUT_HOURS": 24,
  "AGENT_ID": "codeBuddy-阿宝"
}
```

## 字段说明

### `TRANSIT_ROOT`（必填）

中转层根目录的**绝对路径**。所有任务文件、模板、索引页都在这个目录下。

| 系统 | 示例 |
|------|------|
| Windows | `D:\\文档\\workSpace\\中转层` |
| Windows（OneDrive）| `C:\\Users\\你的用户名\\OneDrive\\Documents\\中转层` |
| macOS | `/Users/你的用户名/Documents/中转层` 或 `~/Documents/中转层` |
| Linux | `/home/你的用户名/Documents/中转层` 或 `~/Documents/中转层` |

> ⚠️ Windows 路径用**双反斜杠** `\\`，JSON 转义要求。
> macOS / Linux 路径用**单斜杠** `/`。

### `VAULT_NAME`（可选）

中转层所在的 Obsidian vault 名称。仅在使用 Obsidian 配套工具（如 `notesmd-cli`）时需要。

如果留空，agent 会自动通过 `obsidian.json` 检测。

### `CLAIM_TIMEOUT_HOURS`（可选，默认 `24`）

任务 claim 后多少小时未更新视为可被接管。

- 短（如 4）：适合协作密集、agent 响应快的场景
- 长（如 72）：适合时差大、跨日跨夜的协作

### `AGENT_ID`（建议填写）

当前 agent 的唯一标识，会写入任务 frontmatter 的 `claimed_by` 字段。

**格式建议：** `<平台>-<昵称>`

| 示例 | 适用 |
|------|------|
| `codeBuddy-阿宝` | WorkBuddy 中的阿宝 |
| `cursor-本地` | Cursor IDE agent |
| `claude-code` | Claude Code |
| `chatgpt-gpt4` | ChatGPT |
| `claude-sonnet-4.5` | Claude.ai |

**唯一性：** 同一时间有多个 agent 协作时，AGENT_ID 必须互不相同。

---

## 首次配置（WorkBuddy 流程）

第一次触发"制定中转任务"或"执行中转任务"时：

1. agent 检测到 `~/.zhongzhuan-task/config.json` 不存在
2. 自动用 `AskUserQuestion` 询问配置
3. 用户回答后写入配置文件

---

## 手动配置方式

如果不想走 AskUserQuestion 引导，可以手动创建文件：

**Windows (PowerShell)：**

```powershell
mkdir $HOME\.zhongzhuan-task
@'
{
  "TRANSIT_ROOT": "D:\\文档\\workSpace\\中转层",
  "CLAIM_TIMEOUT_HOURS": 24,
  "AGENT_ID": "codeBuddy-阿宝"
}
'@ | Out-File -Encoding utf8 $HOME\.zhongzhuan-task\config.json
```

**macOS / Linux：**

```bash
mkdir -p ~/.zhongzhuan-task
cat > ~/.zhongzhuan-task/config.json << 'EOF'
{
  "TRANSIT_ROOT": "/Users/你的用户名/Documents/中转层",
  "CLAIM_TIMEOUT_HOURS": 24,
  "AGENT_ID": "claude-code"
}
EOF
```

---

## 多环境配置（如同时用 WorkBuddy + Cursor）

每个 agent 的配置文件是独立的，按各自平台的规范存放：

- **WorkBuddy**：`~/.zhongzhuan-task/config.json`
- **Cursor**：`.cursorrules` 或项目级 `.cursor/config.json`
- **Claude Code**：`~/.claude/zhongzhuan-task-config.json`

但**所有 agent 共享同一个 `TRANSIT_ROOT`**（用 Git 同步或云盘同步）。

---

## 配置验证

配置完成后，运行：

```bash
# WorkBuddy
对 agent 说："查看中转任务配置"

# 命令行（如果 agent 支持）
cat ~/.zhongzhuan-task/config.json
ls <TRANSIT_ROOT>/
```

应能看到 `当前关注.md` 和 `code/`、`other/`、`templates/` 等子目录。