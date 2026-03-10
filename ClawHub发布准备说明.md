# ClawHub 发布准备说明

本文档用于整理 `easy-openclaw` 后续发布到 OpenClaw 官方公共 Skills 注册表 `ClawHub` 的准备事项。

当前阶段的目标不是立刻发布，而是先把下面几件事说清楚：
- 官方现在支持的发布路径到底是什么
- 我们这个仓库现在是否已经满足最低要求
- 如果后续要正式发布，还建议补哪些元数据
- 到真正发布那一天，应该按什么顺序操作

本文档基于 2026-03-10 查到的 OpenClaw 官方文档整理。

## 1. 结论先说

当前可以明确确认的结论有这些：

1. `easy-openclaw` 可以走官方公开发布路径，也就是 `ClawHub`。
2. 当前官方公开文档里，面向用户的安装命令是 `clawhub install <skill-slug>`。
3. 我目前没有查到官方把 `openclaw skill install <skill-slug>` 作为 ClawHub 的标准安装入口来写进文档，所以现阶段不要把这个当成对外承诺。
4. 当前仓库已经满足 Skill 的最低结构要求：有 `SKILL.md`，而且包含合法的 YAML frontmatter。
5. 但如果要让它在 ClawHub / Skills UI 里表现得更完整、更适合被发现，后续仍建议补充 `homepage` 和 `metadata.openclaw`。
6. 现阶段最合理的做法不是立刻去改发布元数据，而是先把功能和文案继续收尾，最后再一次性补齐发布所需信息。

## 2. 官方发布路径是什么

根据 OpenClaw 官方文档，ClawHub 是公开的 Skill 注册表，支持：
- 搜索 Skills
- 安装 Skills
- 更新 Skills
- 发布 Skills
- 同步本地 Skills 到注册表

官方文档当前明确给出的常用命令是：

```bash
clawhub search "calendar"
clawhub install <skill-slug>
clawhub update --all
clawhub publish <path>
clawhub sync --all
```

也就是说，如果以后我们把 `easy-openclaw` 正式发布到 ClawHub，标准用户路径会更像这样：

```bash
clawhub install easy-openclaw
```

而不是：

```bash
openclaw skill install easy-openclaw
```

至少按我目前查到的官方资料，后者不是 ClawHub 这一条链路的标准公开说法。

## 3. 官方对 Skill 的最低要求

OpenClaw 的官方 Creating Skills 文档写得比较直接：

一个 Skill 本质上就是一个目录，里面至少有一个 `SKILL.md` 文件。这个文件用 YAML frontmatter 写元信息，用 Markdown 写说明和规则。

最低要求可以整理成下面这几条：

1. 必须是一个 Skill 目录。
2. 目录里必须有 `SKILL.md`。
3. `SKILL.md` 必须带 YAML frontmatter。
4. frontmatter 至少要有：
   - `name`
   - `description`

另外，Skills 文档还给了几个很重要的解析约束：

1. frontmatter 的 key 需要保持单行写法。
2. `metadata` 需要写成单行 JSON 对象。
3. 如果要给 Skill 补更多 OpenClaw 专用元数据，写在 `metadata.openclaw` 下面。

## 4. 当前仓库是否已经满足最低要求

就现在这个仓库来说，答案是：满足最低要求，可以继续往发布方向准备。

当前已具备：

1. 有独立的 Skill 根目录。
2. 有 `SKILL.md`。
3. `SKILL.md` 有合法 YAML frontmatter。
4. frontmatter 已包含 `name` 和 `description`。
5. 有 README。
6. 有 `references/` 作为支撑文档。

当前还没有补，或者说还没有必要现在就补的，是这些：

1. `homepage` frontmatter
2. `metadata.openclaw`
3. 规范化的发布版本号方案
4. 对外发布时使用的 changelog 文案
5. 确认最终 slug 是否固定为 `easy-openclaw`

## 5. `metadata.openclaw` 是什么

这是 OpenClaw 用来描述 Skill 可发现性、依赖、安装要求和 UI 展示信息的一组元数据。

官方文档里明确提到，ClawHub 和 Skills UI 会利用这些元数据做这些事情：
- 判断 Skill 是否 eligible
- 判断缺少哪些 requirement
- 展示安装入口
- 展示网站链接和图标
- 做搜索与发现

也就是说：

- 没有 `metadata.openclaw`，Skill 依然可以被加载。
- 但如果没有它，ClawHub 和 Skills UI 对这个 Skill 的理解会更弱，展示和引导也会更有限。

官方文档还明确写到：
- 如果没有 `metadata.openclaw`，这个 Skill 默认仍会被视为 eligible
- 除非它被显式禁用，或者属于 bundled skill 且又被 allowlist 阻止

这个点对我们很重要，因为 `easy-openclaw` 目前本质上是一个“流程引导型 / 文档型 Skill”，不是一个强依赖某个单一二进制的工具 Skill。

## 6. 官方当前支持的关键字段

根据官方 Skills 文档，目前与我们最相关的字段主要有这些。

### 顶层 frontmatter

可直接写在 frontmatter 顶层的字段：

- `name`
- `description`
- `homepage`
- `metadata`

其中 `homepage` 官方文档明确支持，并且会显示在 macOS Skills UI 里。

### `metadata.openclaw`

当前官方文档明确列出的字段包括：

- `always`
- `emoji`
- `homepage`
- `os`
- `requires.bins`
- `requires.anyBins`
- `requires.env`
- `requires.config`
- `primaryEnv`
- `install`

这些字段分别大致意味着：

- `always: true`
  - 始终包含该 Skill，跳过其他 eligibility 判断
- `emoji`
  - 给 UI 展示用的小图标
- `homepage`
  - 展示给用户看的主页链接
- `os`
  - 限制支持的平台，如 `darwin`、`linux`、`win32`
- `requires.bins`
  - 要求某些命令必须存在于 PATH
- `requires.anyBins`
  - 这些命令里至少存在一个
- `requires.env`
  - 需要环境变量
- `requires.config`
  - 要求某些 `openclaw.json` 配置路径为真
- `primaryEnv`
  - 让 `skills.entries.<skillKey>.apiKey` 能自动映射到某个环境变量
- `install`
  - 定义安装器规格，供 Skills UI 调用

## 7. `install` 元数据到底有什么用

这个点值得单独说，因为它很容易被误解。

`metadata.openclaw.install` 不是“发布必填项”。

它的主要作用是：
- 给 Skills UI 提供安装方式
- 让网关知道如何在目标主机上安装这个 Skill 依赖

官方当前列出的安装器类型包括：
- `brew`
- `node`
- `go`
- `uv`
- `download`

但这并不意味着每个 Skill 都应该写 `install`。

对于 `easy-openclaw` 来说，当前不建议把 `install` 作为第一优先级，原因很简单：

1. 它不是一个“先装某个 CLI 才能使用”的单体工具。
2. 它更像一个“配置优化向导 + 文档规则 Skill”。
3. 如果现在为了追求发布完整度，硬塞一个安装器，反而容易误导用户，以为必须先装某个依赖，这和我们的产品定位不一致。

所以当前建议是：

1. 第一版发布时，可以不写 `metadata.openclaw.install`。
2. 后面如果我们把某个子能力做成明确的安装型依赖，再补这一段也不迟。

## 8. 针对 `easy-openclaw` 的元数据建议

这里不是现在就改，只是先把建议定下来，等正式发布前一次性处理。

### 建议目标

我们后续补元数据时，目标应该是：

1. 让这个 Skill 在 ClawHub 里容易被发现。
2. 让用户在 UI 里能看到正确的项目主页。
3. 不要错误地把它限制到某个平台。
4. 不要错误地声明一个过强的二进制依赖。
5. 保持它“文档型引导 Skill”的本质。

### 建议最小方案

我建议正式发布前，至少补下面这类信息：

```yaml
---
name: easy-openclaw
description: OpenClaw 配置优化向导，帮助用户完成初始化、权限模式、渠道增强、Skills 推荐和新增渠道接入。
homepage: https://github.com/daheiai/easy-openclaw
metadata: {"openclaw":{"emoji":"🦞","homepage":"https://github.com/daheiai/easy-openclaw","os":["darwin","linux","win32"]}}
---
```

这套最小方案的考虑是：

1. `homepage`
   - 指向 GitHub 仓库即可，最稳定，也最符合现在的对外入口。
2. `emoji`
   - 不是必须，但展示更友好。
3. `os`
   - 这个 Skill 本身不是平台绑定型工具，建议直接覆盖三平台。
4. 不建议现在就写 `requires.bins`
   - 因为 `easy-openclaw` 自身不是某个单一命令行工具。
   - 它内部会按用户选项决定是否需要某些依赖。
   - 如果在 Skill 层面过早写死，会导致本来可运行的引导 Skill 被错误判成不可用。

## 9. 为什么暂时不建议给这个 Skill 加严格依赖门槛

这是一个很实际的问题。

如果我们给 `easy-openclaw` 加上类似：

```json
{"requires":{"bins":["docker","jq","python3"]}}
```

那么只要用户当前机器缺其中一个命令，这个 Skill 就可能直接被判断为不 eligible。

但 `easy-openclaw` 的产品目标并不是“只有环境完美时才允许用户进入”。

它的目标恰恰相反：
- 先让用户进入引导流程
- 再根据具体选项判断要不要修依赖
- 能修就给修复建议
- 不能修就给替代路径

所以从产品逻辑上看：
- 严格 `requires.bins` 更适合某些工具型 Skill
- 不太适合 `easy-openclaw` 这种向导型 Skill

这也是我建议它第一版发布尽量保持轻元数据的原因。

## 10. 仓库当前与“适合发布到 ClawHub”之间还差什么

如果把“最低可发布”和“发布质量较好”分开看，会更清楚。

### 已达到“最低可发布”

1. Skill 结构成立
2. `SKILL.md` frontmatter 成立
3. 名称和描述存在
4. 有完整仓库和 README 支撑

### 距离“发布质量较好”还建议补的项

1. `homepage` frontmatter
2. `metadata.openclaw`
3. 统一对外描述文案
4. 版本号策略
5. changelog 文案
6. 发布前敏感信息检查
7. 发布后安装路径说明

## 11. 发布命名建议

这里先定建议，不现在执行。

### slug

推荐继续使用：

```text
easy-openclaw
```

理由：
- 与仓库名一致
- 好记
- 不需要额外教育用户
- 后续 GitHub、README、ClawHub 三者能保持统一

### display name

推荐二选一：

```text
easy-openclaw
```

或：

```text
Easy OpenClaw
```

如果你更在意统一性，我更推荐直接继续用：

```text
easy-openclaw
```

### 初始版本号

建议二选一：

```text
0.1.0
```

或：

```text
1.0.0
```

我的建议是：

- 如果你认为现在仍在快速调整流程和规则，用 `0.1.0`
- 如果你认为主流程已经稳定，只剩闭环测试，用 `1.0.0`

按当前项目状态，我更倾向于：

```text
0.1.0
```

因为我们还没有把所有渠道审批闭环完全做完。

## 12. 到时候真正发布时怎么操作

如果后续决定正式发布，建议按下面顺序来。

### 第一步：安装 ClawHub CLI

```bash
npm i -g clawhub
```

或：

```bash
pnpm add -g clawhub
```

### 第二步：登录

```bash
clawhub login
```

或者用 token：

```bash
clawhub login --token <token>
```

### 第三步：先做一次本地检查

如果只是想先看当前仓库会不会被扫描到，可以先跑：

```bash
clawhub sync --dry-run
```

这个命令比较适合作为“预检查”，因为它不会真的推到线上。

### 第四步：正式发布

到时可以从这个命令模板开始：

```bash
clawhub publish ./easy-openclaw \
  --slug easy-openclaw \
  --name "easy-openclaw" \
  --version 0.1.0 \
  --changelog "Initial public release" \
  --tags latest
```

如果那时你已经决定它足够稳定，也可以把版本号换成：

```bash
1.0.0
```

## 13. 官方发布门槛

这一点官方写得很明确：

- GitHub 账号至少满一周，才能发布。

这条规则不是仓库要求，而是发布账号要求。

所以如果后续发布失败，先不要急着怀疑仓库结构，先检查：
- 当前登录的 GitHub 账号年龄是否达标
- ClawHub 登录绑定的是否就是那个账号

## 14. 发布前检查清单

建议正式发布前，按下面清单过一遍。

### 文件层

- [ ] `SKILL.md` frontmatter 合法
- [ ] `name` 与仓库意图一致
- [ ] `description` 足够清晰
- [ ] `metadata` 如果要补，必须保持单行 JSON
- [ ] `homepage` 已改成 `https://github.com/daheiai/easy-openclaw`
- [ ] README 不包含错误或过时安装命令
- [ ] 根目录没有不该公开的临时文件

### 信息层

- [ ] 不包含 token
- [ ] 不包含 app secret
- [ ] 不包含 pairing code
- [ ] 不包含私有账号凭证
- [ ] 不包含真实姓名（如果你仍希望避免）
- [ ] 对外链接统一为新 GitHub 地址

### 发布层

- [ ] GitHub 账号年龄满足要求
- [ ] 已完成 `clawhub login`
- [ ] 确定最终 slug
- [ ] 确定最终 version
- [ ] 确定 changelog 文案
- [ ] 先执行过一次 `clawhub sync --dry-run`

## 15. 关于 `openclaw skill install` 这件事的判断

这部分单独强调一下，避免未来 README 写错。

你之前的目标，是希望像官方仓库那样，用户可以非常自然地一行命令安装。

就我目前查到的官方资料来说：
- ClawHub 这一条官方路径，对外明确写的是 `clawhub install <skill-slug>`
- 我没有查到官方把 `openclaw skill install <skill-slug>` 作为等价的标准说法

因此当前最稳的策略是：

1. 对外文档里，如果是 ClawHub 路线，就写 `clawhub install easy-openclaw`
2. 如果以后官方文档更新，明确支持 `openclaw skill install`，那时再跟进改文案
3. 在没有官方依据前，不要把 `openclaw skill install` 写成这个项目的发布承诺

## 16. 现阶段最推荐的节奏

结合当前项目进度，我建议按这个顺序推进：

1. 继续完成 Skill 自身的功能收尾和实测闭环
2. 等主流程稳定后，再补 `homepage` 和 `metadata.openclaw`
3. 做一次发布前敏感信息检查
4. 本地先跑 `clawhub sync --dry-run`
5. 最后再正式执行 `clawhub publish`

这样做的好处是：
- 不会因为现在还在迭代而频繁重发版本
- 不会过早写死 metadata 导致返工
- 可以把最终对外文案一次性定稳

## 17. 当前阶段的建议动作

如果只看“现在下一步该做什么”，我的建议非常简单：

现在先不要改 `SKILL.md` 的发布元数据，保留这份说明文档即可。

等到下面这些条件基本满足后，再进入真正的发布动作：
- 审批闭环实测完成
- 关键文案稳定
- README 不再频繁改动
- 你决定最终的版本号是 `0.1.0` 还是 `1.0.0`

## 18. 本文档对应的官方参考

我这次整理主要参考了 OpenClaw 官方文档里的这些页面：

1. ClawHub
   - https://docs.openclaw.ai/tools/clawhub
2. Skills
   - https://docs.openclaw.ai/skills
3. Creating Skills
   - https://docs.openclaw.ai/tools/creating-skills
4. macOS Skills
   - https://docs.openclaw.ai/platforms/mac/skills

如果后续官方文档更新，优先以这些页面的新内容为准。
