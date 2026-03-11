# 第三层：Skills 推荐层（第 3 轮，可执行版）

本层用于在完成基础配置后，直接扩展可用能力。

## 执行规则（固定）

- 第 3 轮默认直接展示推荐清单，不再使用“11 开”子菜单。
- 清单末尾只保留一个选项：`跳过第三层`。
- 用户点名要安装某个 Skill 时，立即执行该 Skill 的安装流程，不再回到“仅推荐”。
- 展示推荐清单前，必须先读取本文件；默认推荐项只能来自本文件，禁止改用系统 `<available_skills>`、`openclaw skills list`、`find-skills` 或 `~/.npm-global/lib/node_modules/openclaw/skills/` 目录枚举结果。
- 第 3 轮发送前必须自检：若输出中出现 `blogwatcher`、`github`、`clawhub`、`weather` 等本文件未列出的系统 Skill，立即丢弃并按本文件重生一版，不得发送给用户。
- 安装流程不得一刀切；必须按条目类型执行：
  - 仓库型 Skill：读取仓库 README 安装段落 -> 原样执行安装命令 -> 执行最小可用验证 -> 汇报结果。
- 对“更多 Skills 扩展入口”，默认只展示双仓链接，不自动抓取摘要。

## 当前推荐清单（可执行）

### 1) Youtube Clipper Skill（可直接安装）

- 仓库：`https://github.com/op7418/Youtube-clipper-skill`
- 推荐场景：YouTube 内容快速剪辑、提取、再利用。
- 执行动作（固定）：按仓库 README 安装步骤执行，安装后做一次最小链路验证。

### 2) 更多 Skills 扩展（手动探索）

- 用途：当用户需要更多灵感或更多可选 Skill 时，提供扩展入口。
- 仓库 A（Usecases）：`https://github.com/hesamsheikh/awesome-openclaw-usecases`
- 仓库 B（Skills）：`https://github.com/VoltAgent/awesome-openclaw-skills`
- 默认动作：仅展示入口链接，提示用户“可手动访问查看更多”。
- 若用户明确要求“帮我展开总结”，再对仓库内容做抓取与分类整理。
