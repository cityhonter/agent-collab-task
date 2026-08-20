# Changelog

所有版本变更记录于此。格式基于 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.0.0/)。

## [2.0.0] - 2026-08-20

### 重构（BREAKING）

本版本从私有内部 skill 重构为通用开源版本，与 1.x 不兼容。

**变更：**

- 路径配置化：从硬编码 `D:\文档\workSpace\中转层\` 改为 `TRANSIT_ROOT` 配置项
- 中转层目录结构模板化（新增 `templates/` 目录）
- 状态机参数可配置（`CLAIM_TIMEOUT_HOURS`，默认 24h）
- 新增 `AGENT_ID` 配置项，明确标记每个 agent 身份
- SKILL.md 重写，文档结构分离（SKILL.md 只保留核心工作流，详细文档进 `docs/`）
- 新增 4 个平台接入文档（WorkBuddy / Cursor / Claude / ChatGPT）
- 新增 `config.example.md` 配置示例
- 新增 GitHub Actions 自动 lint
- License 改为 MIT

### 新增

- `docs/安装指南.md` — 5 平台安装详细步骤
- `docs/跨平台兼容.md` — 各平台适配说明
- `docs/协议详解.md` — 锁规则、状态机、异常处理完整说明
- `docs/常见问题.md` — FAQ
- `templates/代码任务.md.template` — 代码任务模板
- `templates/其他任务.md.template` — 其他任务模板
- `templates/当前关注.md.template` — 索引页模板
- `examples/` — 任务示例、配置示例
- `.github/workflows/lint-skill.yml` — 自动校验 SKILL.md 格式

## [1.0.0] - 2026-08-19

### 初始发布

- 双角色（制定者 / 执行者）协作协议
- 状态机：pending / claimed / done / blocked / cancelled
- 锁规则：24h 超时可接管
- 适用 WorkBuddy 内部使用