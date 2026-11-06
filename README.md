# dsh-coding-skills

Language-agnostic software-engineering skills for [DeepSeek Harness](https://github.com/deepseek-ai), packaged as a DSH skill bundle. Complements domain bundles such as `dsh-android-skills`: this one covers the parts of daily work that are the same in every stack.

## Skills

| Skill | Use it when |
|---|---|
| `code-review` | Reviewing a diff, commit, or PR; before merging |
| `commit-and-pr` | Writing a commit message, PR/MR description, pre-commit hygiene check |
| `refactor-plan` | A change is too big for one step; splitting a god class; migrating an API |
| `debug-repro` | Something fails and you need a minimal reproduction, root cause, and regression test |
| `test-strategy` | Deciding what to test, at which layer, and how much is enough |
| `perf-investigate` | Something is slow, memory-heavy, or slow to start — measure before optimizing |
| `release-notes` | Cutting a release, writing a changelog, deciding the next version number |

## How to use

Each skill is triggered by what you say, not by a command. The `description` in its frontmatter is the only text used for matching, so plain phrasing works — in Chinese or English.

| Skill | Say something like | Best if you also give | You get back |
|---|---|---|---|
| `code-review` | "review my uncommitted changes", "check commit 3f2a1c", "can this PR merge?" | the range (working tree / sha / base...HEAD / PR) and any focus (concurrency, security) | verdict first, then Blocker / Major / Minor findings, each with `path:line` and the concrete consequence |
| `commit-and-pr` | "commit this", "write the PR description" | which files belong to this change, target branch, issue number | message or PR body shown for confirmation, plus a pre-commit hygiene report |
| `refactor-plan` | "this class is too big", "how do I migrate A to B" | the pain point, the contracts that must not change, how many steps you can ship | baseline + Step 1..N, each independently mergeable, revertible, and verifiable |
| `debug-repro` | "this command fails", "this test is flaky" | the verbatim error, how to trigger it, environment, recent commits | phenomenon / minimal repro / root cause / fix / verification, with a test that fails before and passes after |
| `test-strategy` | "how should I test this", "our CI is slow/flaky" | the module, its historical bugs, CI time budget | layer allocation table plus an ordered "write these N tests first" list |
| `perf-investigate` | "this endpoint is slow", "startup takes 4s" | which metric, current vs target, measurement conditions | baseline table, ranked hotspots with evidence, before/after comparison, and whether it cleared the noise |
| `release-notes` | "prep a release", "what version number" | the range (or let it read the last tag) and the audience | changelog with a version verdict, breaking changes spelled out as what/who/how to upgrade |

Every skill follows the same discipline, which is the actual value here:

- Findings and notes must cite `path:line` and the concrete consequence; no preference-only nitpicks, no "fixed a bug".
- Commit and PR text may only describe changes that exist in the diff, and "verification" may only list commands that were really run.
- Refactors need a green baseline and a characterization test before structure moves.
- Debugging needs a reproduction that fails before the fix and passes after; "cannot reproduce" is a valid, stated outcome.
- Tests count only if they can fail; coverage is a list of unexecuted code, not a score.
- Performance work starts from a number: no profile, no optimization list; changes below the noise floor are reverted.
- Anything ambiguous in a release is treated as breaking, and must state how to upgrade.

Each `skills/<name>/SKILL.md` opens with a `## 使用方法` section: the phrases that trigger it, what input it needs, the actual commands it runs, and what it produces.

## Install

From the DSH plugin manager, by GitHub tag:

```
plugin_manager install_bundle target=github:devwangxiaogang/dsh-coding-skills#v0.2.0
```

Or add it to a profile directly:

```json
{
  "dependencies": {
    "dsh-coding-skills": "github:devwangxiaogang/dsh-coding-skills#v0.2.0"
  },
  "dsh": { "profile": { "bundles": ["dsh-coding-skills"] } }
}
```

The bundle inserts one `@deepseek-ai/dsh-skill-filesystem` row with `providerName: coding-skills`, pointing at its own `skills/` directory.

### Upgrading on Windows

pnpm cannot rename a package directory over an existing one, so switching versions in place fails with:

```
[ERR_PNPM_EPERM] [importPackage ...\node_modules\dsh-coding-skills] EPERM: operation not permitted,
rename '...\dsh-coding-skills_tmp_<pid>_<n>' -> '...\dsh-coding-skills'
```

Remove first, then install:

```
plugin_manager remove_bundle target=dsh-coding-skills
plugin_manager install_bundle target=github:devwangxiaogang/dsh-coding-skills#v0.2.0
```

This is a filesystem rename limitation, not a permissions problem — do not go changing directory ACLs.

## Adding a skill

Create `skills/<name>/SKILL.md` with `name` and `description` in the frontmatter — `description` is the only text used for matching, so write the words a user would actually type. `watch: true` is set in the row, so edits apply without reinstalling.

## License

MIT. See [LICENSE](./LICENSE).
