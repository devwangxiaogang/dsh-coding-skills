# 在其它 AI 工具里使用这些技能

这些技能是**纯 Markdown + YAML frontmatter**，没有私有格式。DSH 直接加载 `skills/<name>/SKILL.md`；换到别的工具，差别只有三件：文件放哪、多篇还是一个文件、怎么让它按需命中。

一句话结论：**能用"按需命中"就不要用"始终生效"**——九篇规则常驻会挤掉真正的代码上下文。

## 0. 先把文件拿到手

这套文件共 9 个技能，各自独立、互不依赖：`api-design`（接口设计）、`code-review`（代码审查）、`commit-and-pr`（提交与 PR）、`debug-repro`（问题定位）、`migration-plan`（迁移计划）、`perf-investigate`（性能排查）、`refactor-plan`（重构计划）、`release-notes`（发版说明）、`test-strategy`（测试策略）。可以只装其中几个——按目录复制即可，缺哪个不影响其余。

三种来源，选一种即可。下面用 `$REPO` 指代你选的那份路径。

**A. 已经装进 DSH**（最省事，版本和 DSH 里一致）

```bash
# Windows / Git Bash
REPO="$DSH_HOME/profiles/desktop/node_modules/dsh-coding-skills"
# macOS / Linux 同理，把 $DSH_HOME 换成你的 dsh-data 目录
ls "$REPO/skills"     # 应看到 9 个目录
```

**B. 从 GitHub 取指定版本**

```bash
git clone --depth 1 --branch v0.3.1 https://github.com/devwangxiaogang/dsh-coding-skills.git /tmp/dcs
REPO=/tmp/dcs
```

> 该机器上如果 `https://github.com` 被重置，用 SSH：`git clone --depth 1 -b v0.3.1 git@github.com:devwangxiaogang/dsh-coding-skills.git /tmp/dcs`，并确认用的是有权限的那把 key（`ssh -T git@github.com`）。

**C. 下载 tag 压缩包**，解压后同样得到 `$REPO`。

拿到后先确认导出脚本能跑（只依赖 Node ≥ 18，无第三方包）：

```bash
node "$REPO/scripts/export-rules.mjs" --list
```

输出九行 `name + description` 即正常。

## 1. 一分钟选型

| 工具 | 放哪 | 形态 | 自动导出 |
|---|---|---|---|
| Claude Code | `.claude/skills/<name>/SKILL.md`（项目）或 `~/.claude/skills/`（个人） | 多篇，**格式与本仓库完全一致** | `--target claude` |
| Codex CLI / OpenCode / Amp | `AGENTS.md`（可放子目录，就近生效） | 单文件聚合 | `--target agents` |
| GitHub Copilot | `.github/copilot-instructions.md`；按路径细分用 `.github/instructions/*.instructions.md` + `applyTo` | 聚合，可拆分 | `--target copilot` |
| Cursor | `.cursor/rules/<name>.mdc`（`alwaysApply: false`） | 多篇 | `--target cursor` |
| Windsurf | `.windsurf/rules/<name>.md`（`trigger: model_decision`） | 多篇 | `--target windsurf` |
| Cline / Roo Code | `.clinerules/*.md`（Roo 用 `.roo/rules/`） | 多篇 | `--target cline` |
| Gemini CLI | `GEMINI.md`（可用 `@path` 引入其它文件） | 单文件聚合 | `--target gemini` |
| Aider | `CONVENTIONS.md` + `read:` 配置 | 单文件聚合 | `--target aider` |
| Continue | `.continue/rules/*.md` | 多篇 | `--target cline` 后改名 |
| 任意聊天工具（ChatGPT / DeepSeek / 豆包 …） | 不开文件，直接粘贴单篇正文 | 手动 | — |

> 各工具的文件名与字段随版本变化。落地后**用 §5 的三步验证一次**，不要只信本页。

## 2. 逐工具安装

每个工具给两条路：**项目级**（跟着仓库走，团队共享，推荐）和**全局**（个人机器上所有仓库生效）。命令都在仓库根目录执行。

### Claude Code

```bash
# 项目级：进仓库根目录
node "$REPO/scripts/export-rules.mjs" --target claude --out .
git status --short .claude        # 应看到 .claude/skills/<name>/SKILL.md × 9

# 全局（个人机器，所有仓库可用）
node "$REPO/scripts/export-rules.mjs" --target claude --out "$HOME"
ls ~/.claude/skills
```

- 格式与本仓库一致，脚本是**逐字节复制**，不需要改字段
- `description` 是唯一触发依据：写成"通用编程助手提示"永不命中；写成"审代码、CR、看看这段改动有没有问题"才会在用户真这么说时加载
- **不要**把长技能塞进根目录 `CLAUDE.md`：那是常驻记忆，九篇全文会把代码上下文挤掉

### Cursor

```bash
node "$REPO/scripts/export-rules.mjs" --target cursor --out .
# → .cursor/rules/<name>.mdc × 9，含 description / globs / alwaysApply: false
```

- 项目级规则就是这些 `.mdc` 文件，提交进仓库即可共享
- 全局规则没有目录形态：Cursor 设置 → Rules → **User Rules** 里粘贴聚合版（`--target agents` 的输出挑几条纪律即可，别贴九篇全文）
- 关键字段：`description`（何时适用）、`globs`（按文件匹配）、`alwaysApply`
- 推荐 `api-design` 用 `globs: **/api/**,**/*controller*,**/*route*`；`test-strategy` 用 `**/*test*,**/__tests__/**`；其余靠 `description` 判断
- 本脚本产出的是 Agent Requested 形态（模型自己决定要不要读）

### Windsurf

```bash
node "$REPO/scripts/export-rules.mjs" --target windsurf --out .
# → .windsurf/rules/<name>.md × 9，trigger: model_decision
```

- 全局规则走 Cascade 面板里的 **Global Rules**（单个编辑框），同样只放纪律条目
- 单文件有字符上限，超限会被静默截断，表现为"后半部分不生效"——先看编辑器提示
- 若改成 `trigger: always_on`，请先删掉正文里的长清单，只留 5~10 行硬规则

### Cline / Roo Code

```bash
node "$REPO/scripts/export-rules.mjs" --target cline --out .
# → .clinerules/<name>.md × 9      （Roo Code 改成 .roo/rules/）
```

- 目录形态优于单文件：一篇一个主题，便于按需引用
- Cline 读 `.clinerules`（文件）或 `.clinerules/`（目录），**两者不要并存**
- 全局规则在 Cline 设置的 Rules 面板里维护，别和仓库里的重复

### Codex CLI / OpenCode / Amp（AGENTS.md 系）

```bash
node "$REPO/scripts/export-rules.mjs" --target agents --out .
# → ./AGENTS.md（约 24k 字符，含九篇全文）
```

- 单文件聚合、每轮都会读：建议**先瘦身**——只留索引与触发条件，细节让模型自己去读 `skills/<name>/SKILL.md`
- 支持分层：仓库根放通用规则，子目录再放更具体的（如 `services/billing/AGENTS.md`）
- 全局：把聚合文件放到用户级配置目录（Codex 为 `~/.codex/AGENTS.md`），内容只保留纪律条目
- 若工具支持文件引用语法，就让聚合文件只写路径

### GitHub Copilot

```bash
node "$REPO/scripts/export-rules.mjs" --target copilot --out .
# → .github/copilot-instructions.md（约 24k 字符）
```

- 仓库级常驻，占用每次补全/对话的上下文；建议只保留纪律条目，不保留完整流程
- 按路径细分（推荐做法）：`.github/instructions/<topic>.instructions.md`，frontmatter 写 `applyTo: "src/api/**"`，把 `api-design` 只挂到接口目录
- 个人级指令：VS Code 设置 `github.copilot.chat.codeGeneration.instructions`（可指向文件），别和仓库级重复
- Copilot 偏补全场景，`code-review`、`api-design` 这类"判断型"技能收益最大；`release-notes` 这类"全流程"技能收益较小

### Gemini CLI

```bash
node "$REPO/scripts/export-rules.mjs" --target gemini --out .
# → ./GEMINI.md（索引 + @skills/<name>/SKILL.md 引入，约 2k 字符）
```

- 脚本刻意做成瘦身版：正文留在 `skills/`，`GEMINI.md` 只做索引和 `@` 引入；**`skills/` 目录必须一起提交**，否则引入失效
- 全局：用户目录下的 `GEMINI.md`（`~/.gemini/GEMINI.md`），同样只放索引
- 常驻上下文，别把九篇全文塞进去

### Aider

```bash
node "$REPO/scripts/export-rules.mjs" --target aider --out .
# → ./CONVENTIONS.md

# 写进 .aider.conf.yml，之后每次自动加载
cat >> .aider.conf.yml <<'EOF'
read: [CONVENTIONS.md]
EOF
```

- 也可以临时用：`aider --read CONVENTIONS.md`
- 聚合文件较大，建议只在"要审代码/要发版"这类会话里带；日常聊天不必加

### Continue

```bash
node "$REPO/scripts/export-rules.mjs" --target cline --out .
mkdir -p .continue/rules
mv .clinerules/*.md .continue/rules/
```

- 新版 Continue 用 `.continue/rules/*.md`，一篇一个文件；老版本走 `config.yaml` 的 `rules` 字段
- 全局规则放在 Continue 的用户级 rules 目录，内容同样只留纪律

### 纯聊天工具（ChatGPT / DeepSeek / 豆包 / Kimi …）

没有文件机制，靠粘贴：

1. 打开目标技能文件，**整篇**复制（frontmatter 可以去掉，正文必须完整）
2. 前面加一句定位：`按下面这套流程做，不要跳步，输出按它的模板`
3. 长任务换主题时重新粘一次，否则模型会遗忘分级口径

半自动：把九篇拼成一个文件（`--target agents` 的输出即可）上传为知识库/附件，让它按需检索。

## 3. 一次装到多个工具

```bash
node "$REPO/scripts/export-rules.mjs" --target all --out /tmp/rules
node "$REPO/scripts/export-rules.mjs" --target all --out .
git status --short | head -50
```

`--target all` 一次生成 `.claude/`、`.cursor/`、`.windsurf/`、`.clinerules/`、`.github/copilot-instructions.md`、`AGENTS.md`、`GEMINI.md`、`CONVENTIONS.md`（40 个文件：9×4 篇 + 4 个聚合文件）。**不要无脑全提交**：同一份规则在两个机制里同时生效（例如既有 `CLAUDE.md` 又有 `.claude/skills/`）会造成重复指令，模型可能执行两遍。选一个主工具，其余放 `.gitignore`。

## 4. 通用适配原则

- **只搬 `description` 与正文**；多余 frontmatter 字段在严格工具里可能导致整篇被忽略
- **一篇文章一件事**；聚合文件用 `---` 和一级标题分隔，方便模型定位
- **优先按需加载**（skills / `model_decision` / `globs` / Agent Requested），常驻是最贵的选择
- **保持中文原文**：翻译会把"必须修 / 应该修 / 可以讨论"这类分级词变成模糊表达，分级正是这些技能的价值
- **不要合并技能**：`debug-repro` 与 `test-strategy` 合并后触发词互相污染，反而不命中
- **升级时重跑导出**：这些文件是生成物，手改会在下次导出时丢失；改源头 `skills/<name>/SKILL.md`

## 5. 怎么验证规则真的生效（三步）

1. **复述测**：问"列出 code-review 的必须修清单前三条，不要展开"。复述不出 = 没加载。
2. **触发测**：说"帮我审一下没提交的改动"，看它是否先给结论、再按 必须修 / 应该修 分级、每条带 `路径:行号`。
3. **反例测**：贴一段有并发写入的代码，看它能否指出"并发下会写坏文件"这种具体后果，而不是"建议加锁优化"。

三步都过，才算配置成功；只做第 2 步容易得到"看起来像在遵守"的假阳性。

## 6. 已知坑

- **上下文挤压**：九篇全文约 1000 行，常驻会让模型后半段"忘记"前面的规则——按需加载不是优化，是正确性
- **字段严格**：某些工具只认 `description`，遇到未知字段整篇跳过；导出脚本已按各目标白名单生成
- **换行符**：本仓库 `.gitattributes` 强制 LF；某些 Windows 工具对 CRLF 敏感，跨工具复制时别让编辑器改写
- **版本漂移**：文件名、字段名会随工具大版本变化；升级工具后重跑一次 §5
- **同一规则两处生效**（如同时有 `CLAUDE.md` 和 `.claude/skills/`）会造成重复指令，模型可能执行两遍；选一处
- **全局与项目级重复**：两处都有时取并集，成本翻倍；全局只留纪律，项目级放完整流程

## 7. 这些规则与工具自带能力的分工

| 你想要的 | 用技能 | 用工具自带 |
|---|---|---|
| 判断"这段改动能不能合" | `code-review` | — |
| 生成符合仓库风格的 commit | `commit-and-pr` | 补全模型 |
| 自动跑测试 | — | CI / 测试运行器 |
| 知道项目怎么构建 | — | `README` / `AGENTS.md` 里的环境说明 |
| 决定测什么 | `test-strategy` | 覆盖率工具只提供数据 |

技能负责**流程与判断口径**，不替代构建、测试、静态检查这些可执行工具。
