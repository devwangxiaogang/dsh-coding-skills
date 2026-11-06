# dsh-coding-skills

与语言无关的软件工程流程技能包，以 DSH skill bundle 形式提供。它补的是 `dsh-android-skills` 这类领域包不覆盖的部分——**每天都要做、且换任何技术栈都不变**的那些环节。

## 包含的技能

| 技能 | 什么时候用 |
|---|---|
| `code-review` | 审查 diff、单个 commit 或 PR，合并前 |
| `commit-and-pr` | 写 commit message、PR/MR 描述、提交前卫生检查 |
| `refactor-plan` | 改动大到一步做不完、拆大类、迁移接口 |
| `debug-repro` | 出问题要最小复现、定位根因、补回归测试 |
| `test-strategy` | 决定测什么、在哪一层测、测到什么程度算够 |
| `perf-investigate` | 变慢、内存高、启动慢——先测量再优化 |
| `release-notes` | 发版、写 changelog、决定下一个版本号 |

## 怎么用

技能不靠命令调用，靠你说的话。匹配只看 frontmatter 里的 `description`，所以正常描述需求就行，中英文都可以。

| 技能 | 你会怎么说 | 最好一起给我 | 你会拿到 |
|---|---|---|---|
| `code-review` | "审一下我还没提交的改动"、"看看 commit 3f2a1c"、"这个 PR 能合吗" | 范围（工作区 / sha / base...HEAD / PR 号）、关注点（并发、安全） | 先给结论，再按 必须修 / 应该修 / 可以讨论 分级，每条带 `路径:行号` 和具体后果 |
| `commit-and-pr` | "提交这些改动"、"帮我写 PR 描述" | 哪些文件属于这次改动、目标分支、关联 issue | 先展示 message 或 PR 正文等你确认，附带提交前卫生检查报告 |
| `refactor-plan` | "这个类太大了怎么拆"、"把 A 迁移到 B 怎么做" | 痛点、不能变的契约、能接受几步 | 基线 + Step 1..N，每步可独立合并、回滚、验证 |
| `debug-repro` | "这个命令报错了"、"这个测试偶发失败" | 逐字报错、复现入口、环境、最近改动 | 现象 / 最小复现 / 根因 / 修复 / 验证，并给出修复前失败、修复后通过的测试 |
| `test-strategy` | "这块逻辑该怎么测"、"CI 太慢/老 flaky" | 被测模块、它历史上的故障、CI 时长预算 | 分层配比表 + 按性价比排好的"先写这 N 个测试"清单 |
| `perf-investigate` | "这个接口太慢了"、"启动要 4 秒" | 哪个指标、现状与目标、测量条件 | 基线表、按占比排序的热点（带证据）、改前后对比、是否超出噪声 |
| `release-notes` | "准备发版"、"版本号怎么定" | 区间（不给就自己读上一个 tag）、受众 | 带版本判定的更新说明，破坏性变更写清"变了什么/谁受影响/怎么升级" |

七个技能共用一套纪律，这才是它们真正的价值：

- 意见和说明必须带 `路径:行号` 与具体后果；不收品味型意见，不写"修复了 bug"
- commit 与 PR 只能描述 diff 里真实存在的东西，"验证方式"只能写真跑过的命令
- 重构必须先有绿的基线和特征测试，才能动结构
- 调 bug 必须有"修复前失败、修复后通过"的复现；"未复现"是可以说、但必须明说的结论
- 测试能失败才算测试；覆盖率是"没跑到的代码清单"，不是分数
- 性能必须有数字：没有 profile 就不给优化清单，低于噪声的改动直接回退
- 发版说明里存疑的改动一律按破坏性变更处理，并给出升级步骤

每个 `skills/<name>/SKILL.md` 开头都有 `## 使用方法` 一节：触发说法、需要你提供什么、实际会跑的命令、产出什么。

## 安装

通过 DSH 插件管理器，按 GitHub tag 安装：

```
plugin_manager install_bundle target=github:devwangxiaogang/dsh-coding-skills#v0.2.0
```

或直接写进 profile：

```json
{
  "dependencies": {
    "dsh-coding-skills": "github:devwangxiaogang/dsh-coding-skills#v0.2.0"
  },
  "dsh": { "profile": { "bundles": ["dsh-coding-skills"] } }
}
```

本 bundle 会插入一行 `@deepseek-ai/dsh-skill-filesystem`，`providerName: coding-skills`，指向包内的 `skills/` 目录。

### Windows 上升级

pnpm 无法把包目录 rename 覆盖到一个已存在的目录上，原地换版本会失败：

```
[ERR_PNPM_EPERM] [importPackage ...\node_modules\dsh-coding-skills] EPERM: operation not permitted,
rename '...\dsh-coding-skills_tmp_<pid>_<n>' -> '...\dsh-coding-skills'
```

先卸载再装：

```
plugin_manager remove_bundle target=dsh-coding-skills
plugin_manager install_bundle target=github:devwangxiaogang/dsh-coding-skills#v0.2.0
```

这是文件系统 rename 的限制，不是权限问题——不要去改目录 ACL。

## 新增技能

在 `skills/<name>/SKILL.md` 建文件，frontmatter 写 `name` 与 `description`——`description` 是**唯一**参与匹配的文本，要用用户真会说的词。行配置里开了 `watch: true`，改完不用重装。

## 许可

MIT，见 [LICENSE](./LICENSE)。
