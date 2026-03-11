# 统一执行、重启与收口

本文件用于“用户点击确认执行之后”的统一动作。

## 执行纪律

开始执行前，先在内部列一份执行清单，至少包含：
- 备份
- 配置深度合并
- 每日记忆归档 cron（若第 2 项为 `记忆增强+每天归档`）
- 审批配置（若第 7 项开启）
- 渠道增强项
- 第 3 轮已选 Skill 安装
- 第 4 轮新增渠道接入
- 重启
- 验收

规则：
- 每完成一项就标记完成。
- 未完成项不得在最终总结里宣告成功。
- 用户中途插话后，要回到清单继续，不得漏项。

## 0. 严格预检（按已选项）

按用户实际开启项检查依赖：
- 联网搜索：`curl`
- Youtube Clipper：`yt-dlp`、`ffmpeg`、`python3`
- Agent Reach：`python3`、`pip`
- OpenClaw Backup / 渠道验证 / 重启：`openclaw`

自动修复规则：
- 用户在第 3 轮点名安装某个 Skill，视为已同意该 Skill 的最小必要依赖补齐。
- 优先按上游 README / install 文档补齐依赖。
- 文档没给时，再尝试通用补齐：
  - `python3 -m ensurepip --upgrade`
  - 必要时 `get-pip.py`
- 禁止向用户索要“服务器密码 / 终端密码”。
- 若当前环境确实无权限自动补齐，才标记为“待人工处理”。

## 1. 备份（如已选）

执行：

```bash
BACKUP_DIR="$HOME/openclaw-backups"
mkdir -p "$BACKUP_DIR" 2>/dev/null || BACKUP_DIR="$HOME/.openclaw/backups"
mkdir -p "$BACKUP_DIR"
cd ~ && zip -r "$BACKUP_DIR/backup-openclaw-all-$(date +%Y%m%d-%H%M%S).zip" .openclaw/ -x ".openclaw/backups/*"
```

规则：
- 备份目录优先放在 `~/.openclaw/` 之外，避免“滚雪球”。

## 2. 读取并深度合并配置

先读取：

```bash
~/.openclaw/openclaw.json
```

然后按用户选择一次性深度合并。

### 第 1 轮写入

具体字段统一按 `references/layer1-base.md`：
- 流式消息：默认必须写 `"partial"`，不能写 `true/false`
- 记忆增强：`memoryFlush.enabled=true`、`softThresholdTokens=40000`
- 记忆增强+每天归档：除 `memoryFlush` 外，还必须创建 cron 并手动触发验收
- 消息回执
- 联网搜索：`browser.defaultProfile="openclaw"`，并更新 `~/.openclaw/workspace/TOOLS.md`
- 权限模式：`coding / full / minimal`

### 第 2 轮写入

具体字段统一按 `references/layer2-channels.md`：
- Exec 审批
- Discord 免 @
- Discord 审批按钮
- Feishu 24h 缓存

审批额外规则：
- `exec-approvals.json` 统一用 `security=allowlist + ask=on-miss + askFallback=deny`
- allowlist 采用“实用优先”的宽放行策略，具体命令集按 `references/layer1-base.md`
- 审批测试不要用 `curl/cat/ls/pip3` 这类默认放行命令做判据
- 若用户开启第 7 项审批，除了 `approvals.exec.*` 与 `exec-approvals.json` 外，还必须同步写入真正的执行路径：
  - `tools.exec.host="gateway"`
  - `tools.exec.security="allowlist"`
  - 测试闭环阶段优先 `tools.exec.ask="always"`，先确保审批链路真正走通
- 若后续用户明确要求“降低打扰，只在未命中 allowlist 时再审批”，再把 `tools.exec.ask` 从 `always` 调回 `on-miss`

### 第 3 轮执行

已选 Skill 的安装与最小验证，统一按 `references/layer3-skills.md`。

强制：
- 只安装用户明确选中的编号
- 不得顺带安装未选条目
- 每项完成后更新状态，再继续下一项

### 第 4 轮执行

新增渠道接入统一按 `references/layer4-onboarding.md`。

强制：
- 只处理用户明确新增的渠道
- 不覆盖当前已可用渠道
- Telegram/Discord/Feishu 的 pairing 规则必须按文档走完，未完成不能宣告接入成功

## 3. 每日记忆归档（若启用）

若第 2 项为 `记忆增强+每天归档`：
- 必须创建 OpenClaw Cron
- 必须手动触发一次验收
- 触发前必须先解析真实 `job id`
- 不能只写 `memoryFlush` 就算完成

## 4. 重启

默认只允许一次最终重启。

唯一例外：
- Feishu 首次接入允许一次前置连接验证重启

最终重启执行与判定，统一按 `references/troubleshooting.md` 的“重启 Gateway（可验证流程）”。

硬规则：
- 未拿到完成状态前，不得宣告“重启成功”
- 不能把 Discord 频道重连当成 Gateway 全量重启

## 5. 验收

按本轮实际启用项做验收；具体口径统一按 `references/troubleshooting.md`：
- 权限模式
- Exec 审批
- Discord 免 @
- 记忆功能
- 联网搜索
- 新增渠道接入

若某项失败：
- 明确指出失败项
- 给修复建议
- 不影响其他已完成项的结论

若第 7 项开启（Exec 审批），必须做“双验证”：
- 配置层：确认 `tools.exec.host=gateway`、`tools.exec.security=allowlist`、`tools.exec.ask=always` 已生效，同时 `approvals.exec.*` 与 `exec-approvals.json` 已写入
- 行为层：再触发一次**控制性的高风险 exec 命令**确认确实进入审批链路；不要只拿“删文件有没有弹窗”当唯一判据

推荐的控制性高风险验收方式：
1. 先创建一个临时测试文件，例如工作区下的 `.approval-smoke-test`
2. 再执行删除这个临时文件的命令，确认出现审批
3. 审批通过后，再确认命令继续执行

这样既能验证“高风险动作会被拦住”，又不会碰用户真实文件。

## 6. 收尾总结

最终输出至少包含：
- 本次实际生效的项目清单
- 备份文件路径（如有）
- 第 3 轮已安装 Skill 结果
- 第 4 轮新增渠道结果（如有）
- 若跳过重启，明确提醒“重启前配置不会生效”

## 7. 结束后止损

本轮总结发出后，流程视为结束。

后续如果用户发送：
- `hi`
- `hello`
- `你好`
- `nihao`
- `test`
- `测一下`
- `现在如何了`
- `还在吗`

这些都只能当在线验收消息处理：
- 正常回复
- 不再读配置
- 不再写配置
- 不再重启
- 不再重新进入统一执行阶段

只有当用户明确表达“继续改配置 / 再调一下 / 重新执行 / 继续修复 / 重装某项”时，才开启下一轮配置动作。
