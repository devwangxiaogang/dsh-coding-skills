# dsh-coding-skills

与语言无关的软件工程流程技能包，以 DSH skill bundle 形式提供。它补的是 `dsh-android-skills` 这类领域包不覆盖的部分——**每天都要做、且换任何技术栈都不变**的那些环节。

## 包含的技能

| 技能 | 什么时候用 |
|---|---|
| `code-review` | 审查 diff、单个 commit 或 PR，合并前 |
| `commit-and-pr` | 写 commit message、PR/MR 描述、提交前卫生检查 |
| `refactor-plan` | 改动大到一步做不完、拆大类、迁移接口 |
| `debug-repro` | 出问题要最小复现、定位根因、补回归测试 |
| `release-notes` | 发版、写 changelog、决定下一个版本号 |

每个技能就是 `skills/<name>/SKILL.md`。五个都不绑定语言和仓库。

写进技能里的硬规则：

- 审查意见必须带 `路径:行号` 和具体后果，不收"品味型"意见
- commit 与 PR 只能描述 diff 里真实存在的东西，"验证方式"只能写真的跑过的命令
- 重构必须先有绿的基线和特征测试，才能动结构
- 调 bug 必须有"修复前失败、修复后通过"的复现；"未复现"是可以接受但要明说的结论
- 发版说明里，存疑的改动一律按破坏性变更处理，并给出升级步骤

## 安装

通过 DSH 插件管理器，按 GitHub tag 安装：

```
plugin_manager install_bundle target=github:devwangxiaogang/dsh-coding-skills#v0.1.0
```

或直接写进 profile：

```json
{
  "dependencies": {
    "dsh-coding-skills": "github:devwangxiaogang/dsh-coding-skills#v0.1.0"
  },
  "dsh": { "profile": { "bundles": ["dsh-coding-skills"] } }
}
```

本 bundle 会插入一行 `@deepseek-ai/dsh-skill-filesystem`，`providerName: coding-skills`，指向包内的 `skills/` 目录。

## 新增技能

在 `skills/<name>/SKILL.md` 建文件，frontmatter 写 `name` 与 `description`——`description` 是**唯一**参与匹配的文本，要用用户真会说的词。行配置里开了 `watch: true`，改完不用重装。

## 许可

MIT，见 [LICENSE](./LICENSE)。
