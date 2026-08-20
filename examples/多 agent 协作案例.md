# 多 agent 协作案例

> 真实场景演示：3 个 agent 接力完成一个项目。

---

## 场景描述

**项目：** 给「智能文档解析」网站加新功能

**agent 分工：**
- **WorkBuddy（阿宝）** — 规划任务、最终验收
- **Cursor IDE** — 写代码
- **Claude Code** — 跑测试、code review

**协作流程：**

```
Day 1 - 规划（WorkBuddy）
─────────────────────────
👤 cityhonter：制定中转任务：实现文档上传功能
🤖 阿宝：已创建 4 个子任务到 code/active/

  P0  实现前端上传组件
  P1  实现后端 /api/upload 接口
  P1  写上传功能的单元测试
  P2  更新用户文档

Day 1 - 编码（Cursor）
─────────────────────────
👤 cityhonter：执行中转任务
🤖 Cursor：claim P0 任务"实现前端上传组件"
         写代码...
         git commit
         标记 done

Day 2 - 测试（Claude Code）
─────────────────────────
👤 cityhonter：执行中转任务
🤖 Claude Code：claim P1 任务"写上传功能的单元测试"
               跑测试、code review
               标记 done

Day 3 - 验收（WorkBuddy）
─────────────────────────
👤 cityhonter：看下中转层还有什么任务
🤖 阿宝：还剩 P2"更新用户文档"，未开始
        全部 P0/P1 已 done
```

---

## 关键时间线

| 时间 | 事件 | 中转层变化 |
|------|------|-----------|
| 8/20 09:00 | 阿宝创建 4 个任务 | active/ 多 4 个文件 |
| 8/20 10:00 | Cursor claim P0 | P0 状态：pending → claimed |
| 8/20 14:00 | Cursor 完成 P0 | P0 状态：claimed → done |
| 8/21 09:00 | Claude claim P1 测试 | P1 状态：pending → claimed |
| 8/21 18:00 | Claude 完成 P1 | P1 状态：claimed → done |
| 8/22 10:00 | 阿宝验收，关 P2 | 用户说"暂不需要文档更新"，P2 改为 cancelled |

---

## 中转层在协作中的状态快照

### 9:00 状态（4 任务全 pending）

`当前关注.md`：

| 优先级 | 任务 | 状态 | 认领者 |
|---|---|---|---|
| P0 | 实现前端上传组件 | 🟡 pending | — |
| P1 | 实现后端 /api/upload 接口 | 🟡 pending | — |
| P1 | 写上传功能的单元测试 | 🟡 pending | — |
| P2 | 更新用户文档 | 🟡 pending | — |

### 14:00 状态（P0 完成）

| 优先级 | 任务 | 状态 | 认领者 |
|---|---|---|---|
| P0 | 实现前端上传组件 | ✅ done | cursor-本地 |
| P1 | 实现后端 /api/upload 接口 | 🟡 pending | — |
| P1 | 写上传功能的单元测试 | 🟡 pending | — |
| P2 | 更新用户文档 | 🟡 pending | — |

### 18:00 次日状态（大部分完成）

| 优先级 | 任务 | 状态 | 认领者 |
|---|---|---|---|
| P0 | 实现前端上传组件 | ✅ done | cursor-本地 |
| P1 | 实现后端 /api/upload 接口 | 🟡 pending | — |
| P1 | 写上传功能的单元测试 | ✅ done | claude-code |
| P2 | 更新用户文档 | 🚫 cancelled | — |

---

## 协作中的边界与责任

| 角色 | 责任 | 不该做什么 |
|------|------|-----------|
| **WorkBuddy（阿宝）** | 拆任务、验收、把控进度 | 不写代码（除非小修） |
| **Cursor** | 写代码、跑本地调试 | 不改任务文件状态以外的东西 |
| **Claude Code** | 跑测试、code review | 不改业务代码（只 review） |

**协议自动保证：**
- Cursor 写完代码不会"被 Claude 抢走"（任务已 done）
- Claude 跑的测试不会"覆盖 Cursor 的代码"（测试是单独任务）
- 阿宝验收时不会"误以为没完成"（任务状态明确）

---

## 失败场景：超时接管

### 场景

Cursor claim P0 任务，2 小时后突然崩溃 / 断网。

### 24 小时后

```
👤 cityhonter：执行中转任务
🤖 Claude Code：读 P0 任务，claimed_by=cursor-本地，
                 claimed_at=27 小时前，超时！
                 接管，开始 review + 修复
                 
🤖 Claude Code：在任务文件追加"接管说明"：
                 > 接管说明：原 claim agent cursor-本地 于 2026-08-21 09:00 超时
                 > 由 claude-code 接管，已完成剩余 30% 代码 + 测试
```

### 关键点

- 锁机制自动避免任务被多个 agent 重复执行
- 超时机制让"死掉"的 agent 不阻塞项目
- 接管时**注明接管者**，让用户看到任务历史

---

## 跨平台同步示意

```
┌──────────────┐         ┌──────────────┐
│  MacBook     │         │  Windows PC  │
│              │         │              │
│  WorkBuddy   │         │  Cursor      │
│  (规划)      │         │  (编码)      │
│              │         │              │
│  中转层 ◄────┼─ Git ───┼─► 中转层     │
│  ~/Documents │         │  D:\文档     │
└──────────────┘         └──────────────┘
        ↑                       ↑
        └───── GitHub 仓库 ─────┘
              (origin)
```

**关键点：** 中转层在每台机器上都是**完整的 Git 仓库**。任何 agent 在本机做的修改通过 `git push` / `git pull` 同步到其他机器。

---

## 经验总结

1. **任务粒度 = 1 个 agent 一次会话能完成**：避免太大或太小
2. **明确责任边界**：写代码的不改任务、跑测试的不改业务代码
3. **及时更新状态**：claim 后即使没完成也更新 `updated` 字段防超时
4. **进度可视化**：所有 agent 看 `当前关注.md` 就懂全局
5. **失败有兜底**：超时机制让任何 agent 崩溃都不会阻塞项目

---

_更多真实场景，欢迎 [GitHub Discussions](https://github.com/cityhonter/agent-collab-task/discussions) 分享。_