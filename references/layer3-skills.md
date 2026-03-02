# 第三层：Skills 推荐层（第 3 轮，可执行版）

本层用于在完成基础配置后，直接扩展可用能力。

## 执行规则（固定）

- 第 3 轮默认直接展示推荐清单，不再使用“11 开”子菜单。
- 清单末尾只保留一个选项：`跳过第三层`。
- 用户点名要安装某个 Skill 时，立即执行该 Skill 的安装流程，不再回到“仅推荐”。
- 安装流程固定为：读取仓库 README 安装段落 -> 原样执行安装命令 -> 执行最小可用验证 -> 汇报结果。
- 对“更多 Skills 扩展入口”，默认只展示双仓链接，不自动抓取摘要。

## 当前推荐清单（可执行）

### 1) Youtube Clipper Skill（可直接安装）

- 仓库：`https://github.com/op7418/Youtube-clipper-skill`
- 推荐场景：YouTube 内容快速剪辑、提取、再利用。
- 执行动作（固定）：按仓库 README 安装步骤执行，安装后做一次最小链路验证。

### 2) OpenClaw 语音播报 / TTS（可直接启用）

- 推荐方案：优先使用 `edge-tts` 路线，做文本转语音播报。
- 推荐场景：希望用“听”的方式接收回复、降低盯屏时间。
- 执行动作（固定）：检查运行环境可用性 -> 按 TTS 路线安装/启用 -> 用一句测试文本生成音频并回报结果。

### 3) RedBookSkills（可安装，需前置检查）

- 仓库：`https://github.com/white0dew/XiaohongshuSkills`
- 核心能力：通过 CDP 自动发布到小红书。
- 前置检查：账号登录态、CDP 连通性、发布频控与平台风控提醒。
- 执行动作（固定）：先做前置检查，检查通过后执行安装并做一次发布链路验证。

### 4) 大黑AI日报（可直接启用）

- 数据源主页：`https://news.daheiai.com`
- RSS 地址：`https://news.daheiai.com/rss.php`
- 核心能力：抓取 RSS 最新 AI 资讯并整理推送。
- 调度规则：固定从 `00:05` 开始，每 4 小时一次（`00:05 / 04:05 / 08:05 / 12:05 / 16:05 / 20:05`）。
- 推荐 cron：`5 */4 * * *`
- 投递规则（强制）：
  - `delivery.mode` 固定为 `announce`
  - `delivery.channel` 必须与当前会话渠道一致（`discord` / `telegram` / `feishu`）
  - `delivery.to` 必填，且必须指向“当前对话窗口”的目标 ID；缺失时禁止宣告安装成功
  - 渠道映射优先级：
    - Discord：优先当前会话 `channelId`（如 `conversation_label`）作为 `delivery.to`
    - Telegram：优先当前会话 `chat_id`；若无 `chat_id` 则回退 `sender_id`
    - Feishu：优先当前会话 `chat_id`；若仅有单聊上下文则回退 `open_id`
- 执行动作（固定）：直接告知 OpenClaw 按上述固定时刻抓取 RSS、去重、摘要并推送；优先使用 OpenClaw 内部定时触发器，不额外要求用户编写程序。
- 执行顺序（强制）：
  - 第一步：先手动触发一次“即时日报”并投递到当前窗口（不是等定时）
  - 第二步：让用户确认“当前窗口已收到测试消息”
  - 第三步：确认后再启用固定时刻调度（`00:05` 起每 4 小时）
- 安装后验收（强制）：
  - 先输出解析到的 `delivery.channel` 与 `delivery.to`
  - 立即执行一次“手动触发日报”测试，并在消息中显式标注“测试消息/即时触发”
  - 若当前窗口未收到测试消息，则标记为“未完成”，并进入投递目标修复分流（先修 `delivery.to` 再重试）

### 5) 更多 Skills 扩展（手动探索）

- 用途：当用户需要更多灵感或更多可选 Skill 时，提供扩展入口。
- 仓库 A（Usecases）：`https://github.com/hesamsheikh/awesome-openclaw-usecases`
- 仓库 B（Skills）：`https://github.com/VoltAgent/awesome-openclaw-skills`
- 默认动作：仅展示入口链接，提示用户“可手动访问查看更多”。
- 若用户明确要求“帮我展开总结”，再对仓库内容做抓取与分类整理。
