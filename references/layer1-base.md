# 第一层：基础推荐层（第 1 轮）

## 备份

备份目标：
- `~/.openclaw/openclaw.json`
- `~/.openclaw/workspace/`
- `~/.openclaw/agents/`

命名规则：`backup-YYYYMMDD-HHMMSS.zip`

执行命令：
```bash
cd ~/.openclaw && zip -r backup-$(date +%Y%m%d-%H%M%S).zip openclaw.json workspace/ agents/
```

## 1) 流式消息

推荐写法：
```json
"channels": {
  "discord": {
    "streaming": "partial"
  }
}
```

合法值：
- `"off"`
- `"partial"`（推荐）
- `"block"`
- `"progress"`

可配合：
```json
"agents": {
  "defaults": {
    "blockStreamingDefault": "on",
    "blockStreamingBreak": "text_end"
  }
}
```

## 2) 记忆功能（本地 embedding）

```json
"agents": {
  "defaults": {
    "compaction": {
      "memoryFlush": {
        "enabled": true,
        "softThresholdTokens": 40000
      }
    },
    "memorySearch": {
      "enabled": true,
      "provider": "local",
      "local": {
        "modelPath": "hf:ggml-org/embeddinggemma-300m-qat-q8_0-GGUF/embeddinggemma-300m-qat-Q8_0.gguf"
      },
      "fallback": "none"
    }
  }
}
```

注意：
- `compaction` 必须深度合并，保留原有字段。
- 首次执行 memory index/search 时会自动下载约 329MB 本地模型文件。

## 3) 消息回执

```json
"messages": {
  "ackReactionScope": "all"
}
```

## 4) 联网搜索

目标：不给用户强绑 Brave API，也不要求本地部署搜索服务；优先用网页正文读取服务，再用隔离浏览器兜底抓取页面内容。

推荐策略（按优先级）：
1) `https://defuddle.md/https://目标网址`（主力正文提取）
2) `https://r.jina.ai/http://目标网址`（备用正文提取）
3) 浏览器隔离抓取兜底（`browser.defaultProfile="openclaw"`）

`openclaw.json` 建议写入（浏览器兜底）：

```json
"browser": {
  "enabled": true,
  "defaultProfile": "openclaw"
}
```

说明：
- `defaultProfile: "chrome"` 会走扩展 relay 和用户日常浏览器上下文，不适合作为默认自动化通道。
- `defaultProfile: "openclaw"` 使用 OpenClaw 托管隔离 profile，更安全也更符合开箱路径。
- 若环境是 Docker 容器、无桌面 VPS、纯命令行虚拟机等无图形界面场景，浏览器兜底可能不可用；此时只依赖前两层正文提取。

写入 `~/.openclaw/workspace/TOOLS.md`（告知 agent 的操作偏好）：

```markdown
## 搜索服务（默认）
- 默认不部署本地搜索服务，也不强依赖 Brave API。
- 第 1 优先：`https://defuddle.md/https://目标网址`
- 第 2 优先：`https://r.jina.ai/http://目标网址`
- 第 3 优先：若前两者失败，再用 `browser`（profile=`openclaw`）打开网页并 snapshot。
- 若环境无图形界面，浏览器兜底可能不可用，不作为联网搜索成功前提。
```

正文提取连通性检查：

```bash
curl -L -s "https://defuddle.md/https://example.com" | head
curl -L -s "https://r.jina.ai/http://example.com" | head
```

## 5) 权限模式（可选）

目标：不再默认只做“安全收紧”，而是先探测当前 OpenClaw 权限状态，再由用户选择要维持现状、完全开放，还是最小安全。

推荐顺序：
1) 维持现状（默认，推荐）
2) 完全开放（`full`，高风险）
3) 最小安全（`minimal` + 沙箱，最严格）

探测命令（固定）：

```bash
openclaw config get tools.profile
openclaw config get agents.defaults.sandbox.mode
command -v docker || echo docker_missing
```

判定：
- 若当前是 `tools.profile="coding"`：可直接继续执行本 skill，大多数基础功能不受影响。
- 若当前是 `tools.profile="minimal"` 或其他明显受限状态：先提示用户手动执行 `openclaw config set tools.profile coding`，再继续本 skill。
- 若当前已是 `tools.profile="full"`：说明环境处于完全放开模式，应额外提醒用户风险。

### A. 维持现状（默认）

推荐给大多数用户：

```json
"tools": {
  "profile": "coding"
},
"agents": {
  "defaults": {
    "sandbox": {
      "mode": "off"
    }
  }
}
```

说明：
- `coding` 适合当前这套 skill 的原生体验。
- 能读写配置、执行常规命令、完成大部分自动化修改。
- 默认不额外改动用户现有权限配置；若探测到已是 `coding`，可直接维持。

若当前权限太低，提示用户执行：

```bash
openclaw config set tools.profile coding
```

### B. 完全开放（`full`，高风险）

适用场景：追求“贾维斯狂奔模式”，希望 agent 尽量少受限制。

```json
"tools": {
  "profile": "full"
},
"agents": {
  "defaults": {
    "sandbox": {
      "mode": "off"
    }
  }
}
```

强提醒：
- 这是完全放开模式，风险最高。
- 不建议在生产环境、主力电脑、重要账号环境下长期启用。
- 必须明确提醒用户：当前 agent 的执行自由度明显更高。

### C. 最小安全（`minimal` + 沙箱）

适用场景：把 OpenClaw 尽量收回到“偏聊天机器人”的状态。

```json
"tools": {
  "profile": "minimal"
},
"agents": {
  "defaults": {
    "sandbox": {
      "mode": "all"
    }
  }
}
```

关键前提：
- 只有系统已安装 Docker，才允许写入 `sandbox.mode="all"`。
- 若未安装 Docker，直接开启沙箱会导致 agent 启动失败，并出现 `Sandbox mode requires Docker` / `docker command was not found in PATH`。

处理规则：
- 若用户选择最小安全，先检查 `docker` 是否可用。
- 若 `docker` 缺失：先提示安装 Docker；安装前不得宣告“最小安全模式已完成”。
- 若环境可自动修复（如 Debian/Ubuntu 且有 `sudo`）：可在用户同意后先安装 Docker。
- 若无法自动修复：明确告知用户先装 Docker，随后再开启 `minimal + sandbox`。

### 审批联动（仅最小安全模式相关）

若用户明确要“最小安全 + 审批联动”，使用以下默认策略：

`exec-approvals.json` 默认策略：

```json
{
  "version": 1,
  "socket": {},
  "defaults": {
    "security": "allowlist",
    "ask": "always",
    "askFallback": "deny"
  },
  "agents": {}
}
```

推荐命令（避免手写 JSON 出错）：

```bash
printf '%s' '{"version":1,"socket":{},"defaults":{"security":"allowlist","ask":"always","askFallback":"deny"},"agents":{}}' | openclaw approvals set --stdin --json
```

审批命令：
- `/approve <id> allow-once`
- `/approve <id> allow-always`
- `/approve <id> deny`

注意：
- OpenClaw `2026.2.24` 下，`tools.exec.askFallback` 不是合法键，`askFallback` 要放在 `exec-approvals.json`。
- 若启用 `sandbox.mode="all"` 后出现 `spawn docker ENOENT` 或 `Sandbox mode requires Docker`，应立即改回 `"off"` 并重启 Gateway。
