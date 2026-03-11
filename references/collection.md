# 条件补充提问与参数收集

本文件只用于“四轮结束后、统一执行前”的最小补充提问。

原则：
- 只问当前已开启项真正缺失的参数。
- 先按文档解释预期效果，不要为了回答说明性问题去读现场配置。
- 只有用户明确要求“查当前实际状态”时，才读取当前环境。

## 通用规则

- 若用户已选择 `跳过第四层`：直接进入收尾确认，不要停住。
- 若第 3 轮刚装完 Skill：先确认已选条目全部处理完，再进入第 4 轮或收尾确认。
- 若审批已经开启，用户只是问“以后会怎样”，不要为了回答解释性问题触发多条 exec 读取命令。

## 第 4 项：联网搜索

- 无需额外补问本地搜索服务或 Brave API Key。
- 默认策略固定为：`defuddle -> r.jina.ai -> browser(profile=openclaw)`。

## 第 5 项：权限模式

- 若用户选 `维持现状`：无需额外补问。
- 若用户选 `完全开放`：只需补充风险提醒。
- 若用户选 `最小安全`：执行前必须确认 Docker 可用；不可用时先提示安装 Docker，不直接落盘。

## 第 7 项：Exec 高危操作审批

若第 7 项不为 `关`，需要确认审批投递位置：
- `session`
- `targets`
- `both`

若包含 `targets`：
- 优先自动获取当前会话目标，不要先让用户手填。
- 自动获取规则：
  - Telegram：优先 `chat_id`，回退 `sender_id`
  - Discord：频道会话优先 `channel:<channelId>`；DM 优先 `user:<userId>`
  - Feishu：优先 `chat_id`，回退 `open_id`
- 只有当前会话拿不到目标 ID 时，才让用户手动提供。

同时遵守：
- `minimal` 下第 7 项强制视为 `关`
- 审批时必须使用完整 UUID，不要建议短 ID

## 第 6 项：Discord 频道免 @ 响应

默认优先自动解析，不先问用户要 `serverId`。

固定思路：
1. 从会话元数据拿 `channelId`
2. 从配置读取 Discord token
3. 通过 Discord channels API 正查 `guild_id`
4. 通过 guild channels API 反查校验
5. 成功后直接把 `guild_id` 作为 `serverId`

只有以下情况才向用户补问：
- 当前会话不是 Discord 频道
- 拿不到 `channelId`
- API 校验失败且无法自动分流处理

## 第 8 项：Discord 审批按钮

若开启：
- 补问 Discord 审批人用户 ID（可多个）

## 第 9 项：飞书限额优化

若开启：
- 只需确认用户当前是否真的在用 Feishu 渠道
- 若答“否”，直接跳过，不做文件修改

## 第 12 项：新增渠道接入

具体平台步骤、回传模板、配对码处理，统一按 `references/layer4-onboarding.md` 执行。

这里只负责收集最少必要参数：
- `discord`：`token`，可选 `accountId`
- `telegram`：`botToken`，可选 `accountId`
- `feishu`：`appId`、`appSecret`

注意：
- Telegram `/start` 或任意消息只用于触发 pairing code，不等于接入完成
- 收到 pairing code 后，必须继续执行 `openclaw pairing approve <channel> <code>`
- 凭据回显必须脱敏
