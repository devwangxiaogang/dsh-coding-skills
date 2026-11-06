# dsh-coding-skills

Language-agnostic software-engineering skills for the DeepSeek Harness (DSH) agent: how to review a change, write a commit, plan a refactor or a migration, debug a failure, decide what to test, investigate performance, design an API, and cut a release.

Each skill is one Markdown file with a fixed shape: when to use it, what it needs from you, the steps it will take, the output it produces, and what it deliberately does not do. They are plain process discipline — no language, framework, or tool lock-in.

## Skills

| Skill | What it does | Typical trigger |
| --- | --- | --- |
| `code-review` | Reviews a diff / commit / PR, graded Blocker → Major → Minor, every finding anchored to `path:line` with a concrete consequence. | 审查代码 / review this PR |
| `commit-and-pr` | Turns the real diff into a Conventional Commit message and a PR body, after a pre-commit hygiene pass (stray files, debug leftovers, secrets, mixed concerns). | 提交 / write a commit message |
| `refactor-plan` | Splits a large risky change into steps that are each mergeable, revertible, verifiable, and ≤300 lines. | 重构 / split this god class |
| `migration-plan` | Plans dependency / runtime / framework / schema / platform migrations: compatibility matrix, rollback-able phases, backfill and verification. | 迁移 / upgrade to v5 |
| `debug-repro` | From verbatim error → minimal repro → bisect → fix → regression test that fails before and passes after. | 报错 / why is this failing |
| `test-strategy` | Decides what to test, at which layer, and how much is enough; coverage and flaky-test policy. | 该写哪些测试 / unit or e2e |
| `perf-investigate` | Measure before optimizing: named metric, budget, reproducible baseline, hot-spot ranking, one change at a time with a before/after table. | 太慢了 / 内存占用高 |
| `api-design` | Endpoint and SDK surface design: naming, shape, validation, error envelope, pagination, compatibility rules. | 接口设计 / error codes |
| `release-notes` | Builds release notes from the actual commit range, plus breaking-change upgrade notes and a semver verdict. | 发版 / changelog |

## Install in DeepSeek Harness

```bash
dsh plugin install github:devwangxiaogang/dsh-coding-skills#v0.3.1
```

Or ask the agent to `install_bundle` this spec. Skills appear in the catalog and load on demand.

Upgrading on Windows: if pnpm reports `ERR_PNPM_EPERM` when renaming an existing install directory, remove the bundle first and then install the new tag:

```
plugin_manager remove_bundle  target=dsh-coding-skills
plugin_manager install_bundle target=github:devwangxiaogang/dsh-coding-skills#v0.3.1
```

If the host cannot reach `https://github.com` (`git ls-remote` → `Recv failure: Connection was reset`, or an install that fails with `connection to github.com timed out after 5000ms`), the `github:` spec dies at its host probe. Install over SSH instead — the profile still records the canonical `github:` spec:

```
plugin_manager install_bundle target=git+ssh://git@github.com/devwangxiaogang/dsh-coding-skills.git#v0.3.1
```

## Using with other AI tools

The bundle is plain Markdown, so the same files work in Claude Code, Cursor, Windsurf, Cline/Roo, GitHub Copilot, Codex CLI (`AGENTS.md`), Gemini CLI, Aider, Continue, or a plain chat window. Get the files once, then export them into that tool's native format:

```bash
git clone --depth 1 --branch v0.3.1 https://github.com/devwangxiaogang/dsh-coding-skills.git /tmp/dcs
REPO=/tmp/dcs
# or reuse the copy DSH already installed:
#   REPO="$DSH_HOME/profiles/desktop/node_modules/dsh-coding-skills"

node "$REPO/scripts/export-rules.mjs" --list
```

| Tool | Command (run in your project) | Writes |
| --- | --- | --- |
| Claude Code | `--target claude --out .` | `.claude/skills/<name>/SKILL.md`, verbatim; `--out "$HOME"` for the personal scope |
| Cursor | `--target cursor` | `.cursor/rules/<name>.mdc` with `description` / `globs` / `alwaysApply: false` |
| Windsurf | `--target windsurf` | `.windsurf/rules/<name>.md` with `trigger: model_decision` |
| Cline / Roo | `--target cline` | `.clinerules/<name>.md` (`mv` to `.roo/rules/` for Roo) |
| GitHub Copilot | `--target copilot` | `.github/copilot-instructions.md`; split into `.github/instructions/*.instructions.md` + `applyTo` for path scoping |
| Codex CLI / OpenCode / Amp | `--target agents` | `AGENTS.md` |
| Gemini CLI | `--target gemini` | slim `GEMINI.md` that `@`-imports `skills/<name>/SKILL.md` |
| Aider | `--target aider` | `CONVENTIONS.md` (+ `read: [CONVENTIONS.md]` in `.aider.conf.yml`) |
| Continue | `--target cline` then move | `.continue/rules/*.md` |
| everything | `--target all --out .exported` | 40 files; review before copying into the project |

Per-tool install steps, project-level vs global paths, and a three-step way to verify a rule actually fires: `docs/using-with-other-ai-tools.md`.

Two things that matter more than the file format: generated files are build outputs, so edit `skills/<name>/SKILL.md` and re-export; and prefer on-demand rules over always-on ones — nine always-on rule files crowd the code out of the context window.

## Adding a skill

1. Create `skills/<name>/SKILL.md`. The directory name must equal the frontmatter `name`.
2. Frontmatter is exactly two fields:

```yaml
---
name: my-skill
description: 一句话说明它做什么。Use when <English triggers>, or 中文触发词。
---
```

3. Body sections: `## 使用方法` (你会怎么说 / 我需要什么 / 我会跑 / 产出 + 边界) first, then numbered `## 1.` steps, a fenced example of the output, and `## 边界` listing what it does not cover.
4. Add the name to `GLOBS` in `scripts/export-rules.mjs` if it should attach to specific file patterns in Cursor/Windsurf.
5. Bump the version, update both READMEs and `locale/*.json`, tag, push.

## License

MIT. See [LICENSE](LICENSE).
