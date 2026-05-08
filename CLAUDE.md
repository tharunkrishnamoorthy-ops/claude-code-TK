# CLAUDE.md

Project memory for Claude Code working in this repository.

## Repo purpose

`claude-code-TK` is a Claude Code **plugin marketplace** plus a small set of
**GitHub-issue automation scripts**. It bundles 13 first-party plugins under
`plugins/` and registers them through `.claude-plugin/marketplace.json`. The
scripts under `scripts/` run on a schedule via GitHub Actions to triage,
deduplicate, and lifecycle issues.

## Layout

```
.
├── .claude/commands/             # Repo-level slash commands
├── .claude-plugin/marketplace.json   # Marketplace registration for the 13 plugins
├── .github/
│   ├── ISSUE_TEMPLATE/           # 5 issue forms
│   └── workflows/                # 12 GitHub Actions
├── examples/{hooks,settings}/    # Reference snippets
├── plugins/                      # 13 bundled plugins (see plugins/README.md)
├── scripts/                      # Bun + TS automation invoked by workflows
├── Script/                       # PowerShell helper for devcontainer launch
├── README.md, SECURITY.md, LICENSE.md, CHANGELOG.md
├── CONTRIBUTING.md               # How to add plugins / work on scripts
├── package.json, tsconfig.json   # Bun script tooling
└── phone-organization-guide.md   # Standalone (unrelated to Claude Code)
```

## Plugins

The authoritative list and per-plugin description lives in
[`plugins/README.md`](./plugins/README.md). Do not duplicate that table.

Conventions every plugin must follow:
- Live at `plugins/<name>/`.
- Include `.claude-plugin/plugin.json` and a `README.md`.
- Be registered in `.claude-plugin/marketplace.json` with `name`,
  `description`, `version`, `author`, `source`, `category`.

Known parity gap: `plugins/security-guidance/` does not currently have a
`README.md` while every other plugin does. Add one if the user asks for it,
but otherwise leave alone.

## Scripts

Runtime: **Bun** (workflows use `oven-sh/setup-bun@v2`). Type-check with
`bun run typecheck` (calls `tsc --noEmit`).

| Script | Purpose | Workflow |
|---|---|---|
| `scripts/sweep.ts` | Mark stale issues; close ones whose lifecycle label has expired. Supports `--dry-run`. | `.github/workflows/sweep.yml` |
| `scripts/auto-close-duplicates.ts` | Close issues flagged as duplicates after a cooldown. | `.github/workflows/auto-close-duplicates.yml` |
| `scripts/backfill-duplicate-comments.ts` | Backfill duplicate-detection comments. | `.github/workflows/backfill-duplicate-comments.yml` |
| `scripts/issue-lifecycle.ts` | Lifecycle config (label thresholds) imported by other scripts. | — |
| `scripts/lifecycle-comment.ts` | Generates lifecycle comment bodies. | — |
| `scripts/comment-on-duplicates.sh` | Bash helper. | — |
| `scripts/edit-issue-labels.sh` | Bash helper. | — |
| `scripts/gh.sh` | Shared `gh` CLI wrapper. | — |

Required env vars for the TS scripts: `GITHUB_TOKEN`, `GITHUB_REPOSITORY_OWNER`,
`GITHUB_REPOSITORY_NAME`. Always test with `--dry-run` first when supported.

## Workflows

Under `.github/workflows/`:

- `claude.yml` — main `@claude` mention handler.
- `claude-dedupe-issues.yml` / `claude-issue-triage.yml` — Claude-driven triage.
- `sweep.yml`, `auto-close-duplicates.yml`, `backfill-duplicate-comments.yml` — run the TS scripts above.
- `issue-lifecycle-comment.yml`, `issue-opened-dispatch.yml`, `remove-autoclose-label.yml`, `lock-closed-issues.yml`, `log-issue-events.yml` — issue lifecycle plumbing.
- `non-write-users-check.yml` — gate workflows that should only run for trusted users.

## Slash commands

Repo-level commands live in `.claude/commands/`:
- `commit-push-pr.md`
- `dedupe.md`
- `triage-issue.md`

Plugin-level commands live under `plugins/<name>/commands/`.

## Branch convention

- Develop on `claude/<topic>-<id>` branches when working through Claude Code on the web.
- Never push directly to `main`.
- Push with `git push -u origin <branch>`. On network failure retry up to 4
  times with exponential backoff (2s, 4s, 8s, 16s).

## House rules for Claude

- Prefer editing existing files over creating new ones.
- Don't add a `CLAUDE.md` inside a plugin folder unless asked.
- Don't create docs (`*.md`) unless the user requests them.
- Don't run destructive git commands (`reset --hard`, force-push, branch
  deletion) without explicit user confirmation.
- Do not push to a branch other than the one specified in the task.

## Pointers

- [`plugins/README.md`](./plugins/README.md) — plugin catalog and structure.
- [`CONTRIBUTING.md`](./CONTRIBUTING.md) — how to add plugins / run scripts locally.
- [`SECURITY.md`](./SECURITY.md) — vulnerability disclosure (HackerOne).
- [`CHANGELOG.md`](./CHANGELOG.md) — Claude Code release history.
- [Plugin docs](https://docs.claude.com/en/docs/claude-code/plugins) — official plugin system reference.
