---
name: commit-and-pr
description: 根据真实 diff 生成规范的 commit message 和 PR/MR 描述，并检查提交前的卫生问题。Use when asked to commit, write a commit message, open a PR, describe changes, or 提交、写提交信息、发 PR、写变更说明。
---

# Commit 与 PR 描述

产物必须**只描述真实发生过的改动**：先取 diff，再写文字。禁止凭记忆编造"已修复 X"。

## 使用方法

**你会怎么说**
- `提交这些改动` / `帮我写 commit message`
- `生成这个 PR 的描述`
- `看看我这次提交有没有不该进来的东西`

**我需要什么**（缺了先问，不猜）
- 哪些改动属于这次提交（工作区可能混着无关文件）
- 目标分支、关联 issue 号
- 偏好：中文还是英文、要不要 scope 前缀（没有偏好就跟随 `git log` 既有风格）

**我会跑**
```bash
git status --short
git diff --stat && git diff && git diff --staged
git log --oneline -10            # 既有 message 风格
git log --oneline origin/HEAD..  # 本分支已有的提交
```

**产出**：先展示 commit message（或 PR 正文），你确认后我才执行 `git commit` / 创建 PR

**边界**：不用 `git add -A`、不 `--no-verify`、不 force push；卫生检查发现问题先报给你

## 1. 取事实

```bash
git status --short
git diff --stat                 # 范围与规模
git diff                        # 未暂存
git diff --staged               # 已暂存
git log --oneline -10           # 仓库既有 message 风格
```

按第 4 步核对后，确认仓库惯例：语言（中文/英文）、是否有 scope、是否用 emoji、是否有 issue 前缀。**跟随既有风格**，不要引入新风格。

## 2. 提交前卫生检查（发现问题先说，别直接提交）

- 误入文件：`node_modules/`、`dist/`、`*.log`、`.env`、本地路径配置、IDE 目录、大二进制
- 调试残留：`console.log`、`print(`、`debugger`、临时注释、被注释掉的整段代码
- 凭证：token/key/密码/cookie、内网域名、真实用户数据
- 副作用：意外修改的锁文件、格式化整个文件导致的大 diff（应拆成独立提交）
- 混装：一次提交里同时有重构 + 功能 + 格式化 → 建议拆分

## 3. Commit message

默认 Conventional Commits，除非仓库另有约定：

```
<type>(<scope>): <祈使句，≤ 50 字，说做了什么>

<body：为什么这么改，1-3 行；说明取舍或影响面>
```

- type：`feat|fix|refactor|perf|test|docs|build|ci|chore`
- 标题说**结果**而非过程："修复并发下重复写入" 好于 "修改了 lock 的判断"
- 破坏性变更：`feat(api)!:` 并在 body 写迁移方式
- 不写"update files"、"fix bug"、"优化代码" 这类空话

## 4. PR / MR 描述

```markdown
## 变更内容
- <按模块列 2-5 条，讲清行为变化，不是文件清单>

## 原因
<解决什么问题；关联 issue：#123>

## 影响面 / 风险
- 影响：<受影响的接口、数据、配置、平台>
- 风险：<可能出问题的地方 + 触发条件>
- 回滚：<怎么退>

## 验证方式
- [ ] <实际跑过的命令与结果：单测/构建/手动路径>
- [ ] <未覆盖的部分，明确写出>

## 截图 / 产物
<UI 改动附截图；无则删掉本节>
```

规则：

- "验证方式"只写**真的执行过**的；没跑的写"未验证：缺 X 环境"
- 不写"已全面测试"这类无法证伪的话
- 大 PR 在开头加一句 30 秒能读完的摘要

## 5. 执行边界

- 提交信息写好后**先展示**，等用户确认再 `git commit`
- 不用 `--no-verify` 跳过钩子；钩子失败就修问题
- 不 force push、不改写已推送历史，除非用户明确要求
- 不自动 `git add -A`；按文件显式暂存，逐条说明为何纳入
