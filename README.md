# dsh-coding-skills

Language-agnostic software-engineering skills for [DeepSeek Harness](https://github.com/deepseek-ai), packaged as a DSH skill bundle. Complements domain bundles such as `dsh-android-skills`: this one covers the parts of daily work that are the same in every stack.

## Skills

| Skill | Use it when |
|---|---|
| `code-review` | Reviewing a diff, commit, or PR; before merging |
| `commit-and-pr` | Writing a commit message, PR/MR description, pre-commit hygiene check |
| `refactor-plan` | A change is too big for one step; splitting a god class; migrating an API |
| `debug-repro` | Something fails and you need a minimal reproduction, root cause, and regression test |
| `release-notes` | Cutting a release, writing a changelog, deciding the next version number |

Each skill is a `SKILL.md` under `skills/<name>/`. All five are language- and repository-agnostic; they describe process, output shape, and the checks that catch real defects.

Distinctive rules baked in:

- Reviews must cite `path:line` and the concrete consequence of each finding; no preference-only nitpicks.
- Commits and PR bodies may only describe changes that actually exist in the diff, and "verification" may only list commands that were really run.
- Refactors must establish a green baseline and a characterization test before touching structure.
- Debugging requires a reproduction that fails before the fix and passes after; "cannot reproduce" is a valid, stated outcome.
- Release notes treat anything ambiguous as a breaking change and must state how to upgrade.

## Install

From the DSH plugin manager, by GitHub tag:

```
plugin_manager install_bundle target=github:devwangxiaogang/dsh-coding-skills#v0.1.0
```

Or add it to a profile directly:

```json
{
  "dependencies": {
    "dsh-coding-skills": "github:devwangxiaogang/dsh-coding-skills#v0.1.0"
  },
  "dsh": { "profile": { "bundles": ["dsh-coding-skills"] } }
}
```

The bundle inserts one `@deepseek-ai/dsh-skill-filesystem` row with `providerName: coding-skills`, pointing at its own `skills/` directory.

## Adding a skill

Create `skills/<name>/SKILL.md` with `name` and `description` in the frontmatter — `description` is the only text used for matching, so write the words a user would actually type. `watch: true` is set in the row, so edits apply without reinstalling.

## License

MIT. See [LICENSE](./LICENSE).
