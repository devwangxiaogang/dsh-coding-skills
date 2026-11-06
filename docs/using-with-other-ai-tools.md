# 在其它 AI 工具里使用这些技能

这些技能是**纯 Markdown + YAML frontmatter**，没有私有格式。DSH 直接加载 `skills/<name>/SKILL.md`；换到别的工具，差别只有"放在哪"和"多篇还是一个文件"。

一句话结论：**能用"按需命中"就不要用"始终生效"**——九篇规则常驻会挤掉真正的代码上下文。

## 一分钟选型

| 工具 | 放哪 | 形态 | 自动导出 |
|---|---|---|---|
| Claude Code | `.claude/skills/<name>/SKILL.md`（项目或 `~/.claude/skills/`） | 多篇，**格式与本仓库完全一致** | `--target claude` |
| Codex CLI / OpenCode / Amp | `AGENTS.md`（可放子目录，就近生效） | 单文件聚合 | `--target agents` |
| GitHub Copilot | `.github/copilot-instructions.md`；按路径细分用 `.github/instructions/*.instructions.md` + `applyTo` | 聚合，可拆分 | `--target copilot` |
| Cursor | `.cursor/rules/<name>.mdc`（`alwaysApply: false`） | 多篇 | `--target cursor` |
| Windsurf | `.windsurf/rules/<name>.md`（`trigger: model_decision`） | 多篇 | `--target windsurf` |
| Cline / Roo Code | `.clinerules/*.md`（Roo 用 `.roo/rules/`） | 多篇 | `--target cline` |
| Gemini CLI | `GEMINI.md`（可用 `@path` 引入其它文件） | 单文件聚合 | `--target gemini` |
| Aider | `CONVENTIONS.md` + `--read CONVENTIONS.md` | 单文件聚合 | `--target aider` |
| 任意聊天工具（ChatGPT / DeepSeek / 豆包 …） | 不开文件，直接粘贴单篇正文 | 手动 | — |

> 各工具的文件名与字段随版本变化。落地后**用 §4 的三步验证一次**，不要只信本页。

## 1. 全自动：导出脚本

仓库自带 `scripts/export-rules.mjs`，只依赖 Node，不带第三方包：

```bash
node scripts/export-rules.mjs --target claude --out .
node scripts/export-rules.mjs --target cursor --out .
node scripts/export-rules.mjs --target copilot --out .
node scripts/export-rules.mjs --target all --out /tmp/rules   # 一次全出，先看差异再拷
```

`--target claude` 会生成 `.claude/skills/<name>/SKILL.md`，与 DSH 里的目录结构完全一致；其余目标生成对应的规则文件。脚本只读 `skills/`，不修改源文件。

## 2. 逐工具要点

### Claude Code

技能机制与本仓库一致（`name` + `description` 的 frontmatter），所以**直接复制目录**即可：

```bash
node scripts/export-rules.mjs --target claude --out .
# → .claude/skills/<name>/SKILL.md （9 个）
```

- `description` 是唯一触发依据：写成"通用编程助手提示"会让它永不命中；写成"审代码、CR、看看这段改动有没有问题"才会在用户真这么说时加载
- 不要把长技能塞进根目录 `CLAUDE.md`：那是常驻记忆，九篇全文会把代码上下文挤掉
- 团队共享时把 `.claude/skills/` 提交进仓库；个人偏好放 `~/.claude/skills/`

### Codex CLI / AGENTS.md 系（OpenCode、Amp 等）

```bash
node scripts/export-rules.mjs --target agents --out .
# → ./AGENTS.md
```

- 单文件聚合，模型每次都会读，所以**只放索引 + 触发条件**的瘦身版更划算；需要细节时让它去读 `skills/<name>/SKILL.md`
- 支持分层：仓库根放通用规则，子目录再放更具体的（如 `services/billing/AGENTS.md`）
- 若工具支持"指令文件引用"，把正文留在 `skills/`，聚合文件里只写路径

### GitHub Copilot

```bash
node scripts/export-rules.mjs --target copilot --out .
# → .github/copilot-instructions.md
```

- 仓库级常驻，占用每次补全/对话的上下文；建议只保留纪律条目，不保留完整流程
- 按路径细分：`.github/instructions/*.instructions.md`，frontmatter 用 `applyTo: "src/api/**"`，把 `api-design` 只挂到接口目录
- Copilot 偏补全场景，`code-review`、`api-design` 这类"判断型"技能收益最大；`release-notes` 这类"全流程"技能收益较小

### Cursor

```bash
node scripts/export-rules.mjs --target cursor --out .
# → .cursor/rules/<name>.mdc
```

- 关键字段：`description`（何时适用）、`globs`（按文件匹配）、`alwaysApply`
- **不要**把九篇都设成 `alwaysApply: true`：那是每轮对话的固定成本，且会与代码本身竞争注意力
- 推荐：`api-design` 用 `globs: ["**/api/**", "**/*controller*", "**/*route*"]`；`test-strategy` 用 `globs: ["**/*test*", "**/__tests__/**"]`；其余用 `description` + `alwaysApply: false` 由模型自行判断
- Cursor 有 4 类规则（Always / Auto Attached / Agent Requested / Manual），本脚本生成的是 Agent Requested 形态

### Windsurf

```bash
node scripts/export-rules.mjs --target windsurf --out .
# → .windsurf/rules/<name>.md
```

- 用 frontmatter 的 `trigger: model_decision` 让模型按 `description` 决定何时加载
- 若用 `trigger: always_on`，请先删掉正文里的长清单，只留 5~10 行硬规则
- 单文件上限（字符数）按编辑器提示为准，超限会被截断，表现为"后半部分不生效"

### Cline / Roo Code

```bash
node scripts/export-rules.mjs --target cline --out .
# → .clinerules/<name>.md      （Roo Code 可改放 .roo/rules/）
```

- 目录形态优于单文件：一篇一个主题，便于按需引用
- Cline 读 `.clinerules`（文件）或 `.clinerules/`（目录），两者不要并存

### Gemini CLI

```bash
node scripts/export-rules.mjs --target gemini --out .
# → ./GEMINI.md
```

- 可用 `@skills/code-review/SKILL.md` 形式的引入，把正文留在原处，`GEMINI.md` 只做索引
- 常驻上下文，同样建议只放纪律条目

### Aider / Continue

```bash
node scripts/export-rules.mjs --target aider --out .
# → ./CONVENTIONS.md
```

- Aider 需显式读入：`aider --read CONVENTIONS.md`；可写进 `.aider.conf.yml`
- Continue 用 `.continue/rules/*.md`（新版本）或 `config.yaml` 的 rules 字段，一篇一个文件

### 纯聊天工具（ChatGPT / DeepSeek / 豆包 / Kimi …）

没有文件机制，靠粘贴：

1. 打开目标技能文件，**整篇**复制（frontmatter 可以去掉，正文必须完整）
2. 前面加一句定位：`按下面这套流程做，不要跳步，输出按它的模板`
3. 长任务换主题时重新粘一次，否则模型会遗忘分级口径

半自动：把九篇拼成一个文件（`--target agents` 的输出即可）上传为知识库/附件，让它按需检索。

## 3. 通用适配原则

- **只搬 `description` 与正文**；多余 frontmatter 字段在严格工具里可能导致整篇被忽略
- **一篇文章一件事**；聚合文件用 `---` 和一级标题分隔，方便模型定位
- **优先按需加载**（skills / `model_decision` / `globs` / Agent Requested），常驻是最贵的选择
- **保持中文原文**：翻译会把"必须修 / 应该修 / 可以讨论"这类分级词变成模糊表达，分级正是这些技能的价值
- **不要合并技能**：`debug-repro` 与 `test-strategy` 合并后触发词互相污染，反而不命中

## 4. 怎么验证规则真的生效（三步）

1. **复述测**：问"列出 code-review 的必须修清单前三条，不要展开"。复述不出 = 没加载。
2. **触发测**：说"帮我审一下没提交的改动"，看它是否先给结论、再按 必须修 / 应该修 分级、每条带 `路径:行号`。
3. **反例测**：贴一段有并发写入的代码，看它能否指出"并发下会写坏文件"这种具体后果，而不是"建议加锁优化"。

三步都过，才算配置成功；只做第 2 步容易得到"看起来像在遵守"的假阳性。

## 5. 已知坑

- **上下文挤压**：九篇全文约 1000 行，常驻会让模型后半段"忘记"前面的规则——按需加载不是优化，是正确性
- **字段严格**：某些工具只认 `description`，遇到未知字段整篇跳过；导出脚本已按各目标白名单生成
- **换行符**：本仓库 `.gitattributes` 强制 LF；某些 Windows 工具对 CRLF 敏感，跨工具复制时别让编辑器改写
- **版本漂移**：文件名、字段名会随工具大版本变化；升级工具体后重跑一次 §4
- **同一规则两处生效**（如同时有 `CLAUDE.md` 和 `.claude/skills/`）会造成重复指令，模型可能执行两遍；选一处

## 6. 这些规则与工具自带能力的分工

| 你想要的 | 用技能 | 用工具自带 |
|---|---|---|
| 判断"这段改动能不能合" | `code-review` | — |
| 生成符合仓库风格的 commit | `commit-and-pr` | 补全模型 |
| 自动跑测试 | — | CI / 测试运行器 |
| 知道项目怎么构建 | — | `README` / `AGENTS.md` 里的环境说明 |
| 决定测什么 | `test-strategy` | 覆盖率工具只提供数据 |

技能负责**流程与判断口径**，不替代构建、测试、静态检查这些可执行工具。
