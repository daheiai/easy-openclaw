# easy-openclaw

一个面向 OpenClaw 的对话式配置优化技能（Skill）。

目标是让用户用最少轮次完成常用优化，并保持可回滚、可验证、可扩展。

## 解决什么问题

- 新用户配置 OpenClaw 时步骤多、容易漏项。
- 安全收紧、渠道联动、审批机制容易配错。
- 多渠道（Discord / Feishu / Telegram）接入与验证成本高。
- 测试阶段缺少统一观测口径。

## 核心流程

- 第 0 层（可选）：测试观测模式（输出详细写入/重启/验收信息）
- 第 1 层：基础推荐（流式消息、记忆、回执、联网搜索、安全收紧）
- 第 2 层：渠道增强（Discord / Feishu / Telegram）
- 第 3 层：Skills 推荐（默认展示，点名即安装）
- 第 4 层：新增渠道接入引导（可跳过）

原则：先收集选择，再统一写入；尽量只重启一次 Gateway（Feishu 首接入可有一次前置重启）。

## 当前能力概览

- 安全收紧：已支持（含 `exec-approvals` 默认策略）
- Discord：免 @ 响应 + 审批转发联动
- Feishu：探测 24h 缓存优化（降低配额消耗）
- Telegram：审批联动（可在 Telegram 内 `/approve`）
- Skills 第三层：可执行（非占位），含大黑 AI 日报等条目

## 目录说明

- `SKILL.md`：主流程与行为规则
- `references/configs.md`：总览入口
- `references/layer0-testing.md`：测试观测模式
- `references/layer1-base.md`：基础推荐层
- `references/layer2-channels.md`：渠道增强层
- `references/layer3-skills.md`：Skills 推荐层
- `references/layer4-onboarding.md`：新增渠道接入引导
- `references/troubleshooting.md`：故障排查与重启验证
- `openclaw插件开发记录.md`：开发记录与 TODO

## 使用建议

1. 先在测试环境跑一轮完整流程（建议开启第 0 层测试观测）。
2. 验证关键闭环：写入成功、重启成功、渠道 `works`、审批 `/approve` 可用。
3. 通过后再在正式环境复用同一套流程。

## 版本同步（GitHub）

- 本地更新：`git add -A && git commit -m "msg" && git push`
- 测试机拉取：`git pull --ff-only`

