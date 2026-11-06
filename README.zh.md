# dsh-coding-skills

面向 DeepSeek Harness（DSH）的编程工作流技能包，与语言、框架、工具链无关：怎么审代码、怎么写提交、怎么规划重构和迁移、怎么定位问题、该测什么、怎么查性能、怎么设计接口、怎么发版。

每个技能就是一个 Markdown 文件，结构固定：什么时候用、需要你提供什么、它会做哪几步、产出什么、以及它明确不做什么。全部是流程纪律，不绑定具体技术栈。

## 技能列表

| 技能 | 做什么 | 典型触发 |
| --- | --- | --- |
| `code-review` | 审查 diff / commit / PR，按 Blocker → Major → Minor 分级，每条意见都落在 `路径:行号` 上并说明具体后果。 | 审查代码 / 看看这个 PR |
| `commit-and-pr` | 基于真实 diff 生成 Conventional Commit 信息和 PR 描述，提交前先过一遍卫生检查（误入文件、调试残留、凭证、混装改动）。 | 提交 / 写提交信息 |
| `refactor-plan` | 把大改动拆成每步可独立合并、可独立回滚、可验证、≤300 行的序列。 | 重构 / 拆分这个类 |
| `migration-plan` | 规划依赖 / 运行时 / 框架 / schema / 平台迁移：兼容矩阵、可回滚阶段、数据回填与验证。 | 迁移 / 升级大版本 |
| `debug-repro` | 从原始报错 → 最小复现 → 二分定位 → 修复 → 补一个修前失败、修后通过的回归测试。 | 报错 / 为什么不工作了 |
| `test-strategy` | 决定测什么、在哪一层测、测到什么程度算够，以及覆盖率与 flaky 处理。 | 该写哪些测试 / 单测还是集成 |
| `perf-investigate` | 先测量再优化：指标与预算、可重复基线、热点排序、一次改一处并给出前后对比。 | 太慢了 / 内存占用高 |
| `api-design` | 接口与 SDK 表面设计：命名、形状、校验、错误结构、分页、兼容规则。 | 接口设计 / 错误码 |
| `release-notes` | 从真实提交区间生成变更说明、破坏性变更升级须知与语义化版本判定。 | 发版 / 写更新日志 |

## 在 DeepSeek Harness 里安装

```bash
dsh plugin install github:devwangxiaogang/dsh-coding-skills#v0.3.1
```

也可以直接让 agent `install_bundle` 这个 spec。安装后技能出现在目录里，按需加载。

Windows 上升级时若 pnpm 报 `ERR_PNPM_EPERM`（重命名已有安装目录失败），先卸载再装新 tag：

```
plugin_manager remove_bundle  target=dsh-coding-skills
plugin_manager install_bundle target=github:devwangxiaogang/dsh-coding-skills#v0.3.1
```

如果这台机器访问不了 `https://github.com`（`git ls-remote` 报 `Recv failure: Connection was reset`，或安装时报 `connection to github.com timed out after 5000ms`），`github:` spec 会在探测阶段就失败。改用 SSH 形式安装，profile 里仍会记成规范的 `github:` spec：

```
plugin_manager install_bundle target=git+ssh://git@github.com/devwangxiaogang/dsh-coding-skills.git#v0.3.1
```

## 在其它 AI 工具里使用

同一份 Markdown 可以直接用在 Claude Code、Cursor、Windsurf、Cline/Roo、GitHub Copilot、Codex CLI（`AGENTS.md`）、Gemini CLI、Aider、Continue，以及任何聊天窗口。先把文件拿到手，再导出成各工具的原生格式：

```bash
git clone --depth 1 --branch v0.3.1 https://github.com/devwangxiaogang/dsh-coding-skills.git /tmp/dcs
REPO=/tmp/dcs
# 或者直接复用 DSH 已装好的那份：
#   REPO="$DSH_HOME/profiles/desktop/node_modules/dsh-coding-skills"

node "$REPO/scripts/export-rules.mjs" --list
```

| 工具 | 命令（在目标仓库根目录执行） | 产出 |
| --- | --- | --- |
| Claude Code | `--target claude --out .` | `.claude/skills/<name>/SKILL.md`，逐字节复制；个人机器用 `--out "$HOME"` |
| Cursor | `--target cursor` | `.cursor/rules/<name>.mdc`，带 `description` / `globs` / `alwaysApply: false` |
| Windsurf | `--target windsurf` | `.windsurf/rules/<name>.md`，`trigger: model_decision` |
| Cline / Roo | `--target cline` | `.clinerules/<name>.md`（Roo 改放 `.roo/rules/`） |
| GitHub Copilot | `--target copilot` | `.github/copilot-instructions.md`；要按路径命中就拆成 `.github/instructions/*.instructions.md` + `applyTo` |
| Codex CLI / OpenCode / Amp | `--target agents` | `AGENTS.md` |
| Gemini CLI | `--target gemini` | 瘦身版 `GEMINI.md`，用 `@` 引入 `skills/<name>/SKILL.md` |
| Aider | `--target aider` | `CONVENTIONS.md`（配合 `.aider.conf.yml` 的 `read: [CONVENTIONS.md]`） |
| Continue | `--target cline` 后移动 | `.continue/rules/*.md` |
| 全部 | `--target all --out .exported` | 40 个文件，先看再拷 |

逐工具的安装步骤、项目级与全局路径、以及验证规则真生效的三步法：`docs/using-with-other-ai-tools.md`。

比格式更重要的两条：生成物是构建产物，要改就改 `skills/<name>/SKILL.md` 再重新导出；能用「按需命中」就不要用「始终生效」，九份常驻规则会把代码从上下文窗口里挤出去。

## 新增一个技能

1. 建 `skills/<name>/SKILL.md`，目录名必须等于 frontmatter 里的 `name`。
2. frontmatter 只有两个字段：

```yaml
---
name: my-skill
description: 一句话说明它做什么。Use when <英文触发词>, or 中文触发词。
---
```

3. 正文结构：先 `## 使用方法`（你会怎么说 / 我需要什么 / 我会跑 / 产出 + 边界），再编号的 `## 1.` 步骤，一段围栏示例输出，最后 `## 边界` 说明它不覆盖什么。
4. 如果需要按文件模式命中（Cursor/Windsurf），在 `scripts/export-rules.mjs` 的 `GLOBS` 里补一条。
5. 升版本号，同步两份 README 与 `locale/*.json`，打 tag，推送。

## 许可

MIT，见 [LICENSE](LICENSE)。
