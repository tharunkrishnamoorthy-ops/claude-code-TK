# Contributing

Thanks for your interest in contributing to `claude-code-TK`. This repo bundles
official Claude Code plugins and the GitHub-issue automation that supports
them. The notes below cover how to add a plugin, work on the issue scripts,
and submit changes.

## Before you contribute

- **Security issues** — do **not** open a GitHub issue or PR. Report through
  HackerOne per [`SECURITY.md`](./SECURITY.md).
- **Bugs / feature requests** — use the templates under
  [`.github/ISSUE_TEMPLATE/`](./.github/ISSUE_TEMPLATE/).

## Adding a plugin

1. Create `plugins/<your-plugin>/` following the structure documented in
   [`plugins/README.md`](./plugins/README.md#plugin-structure).
2. Include a `README.md` describing what the plugin does, the commands /
   agents / skills / hooks it ships, and a usage example.
3. Add `.claude-plugin/plugin.json` with the plugin metadata (match the shape
   of an existing plugin like `plugins/code-review/.claude-plugin/plugin.json`).
4. Register the plugin in [`.claude-plugin/marketplace.json`](./.claude-plugin/marketplace.json)
   with `name`, `description`, `version`, `author`, `source`, and `category`.
   Follow the shape of existing entries.
5. Update the plugin table in [`plugins/README.md`](./plugins/README.md).

## Working on the issue-automation scripts

The scripts under `scripts/` are TypeScript executed by [Bun](https://bun.sh).
GitHub Actions invokes them via `oven-sh/setup-bun@v2` then `bun run`.

### Local setup

```bash
curl -fsSL https://bun.sh/install | bash   # install Bun
bun install                                 # @types/bun + typescript
bun run typecheck                           # tsc --noEmit
```

### Running a script locally

The TS scripts read three env vars:

| Var | Purpose |
|---|---|
| `GITHUB_TOKEN` | PAT or app token with the scopes the script needs |
| `GITHUB_REPOSITORY_OWNER` | Repo owner (e.g. `anthropics`) |
| `GITHUB_REPOSITORY_NAME`  | Repo name (e.g. `claude-code`) |

Always start with a dry run when the script supports it:

```bash
GITHUB_TOKEN=$YOUR_TOKEN \
GITHUB_REPOSITORY_OWNER=anthropics \
GITHUB_REPOSITORY_NAME=claude-code \
  bun run sweep:dry
```

Available `package.json` scripts: `sweep`, `sweep:dry`,
`auto-close-duplicates`, `backfill-duplicate-comments`, `typecheck`.

### Code style for scripts

- Use Bun-native APIs (`fetch`, top-level `await`, `process.env`). Don't pull
  in `axios`, `node-fetch`, `dotenv`, etc.
- Keep types explicit at GitHub-API boundaries (see `scripts/auto-close-duplicates.ts`
  for the existing pattern).
- Run `bun run typecheck` before opening a PR.

## Branching and pull requests

- Branch from `main`. Use a short, descriptive name; when working through
  Claude Code on the web, follow the `claude/<topic>-<id>` convention.
- One logical change per PR. Reference the relevant issue.
- Push with `git push -u origin <branch>`.
- Never force-push to `main`.
- The PR description should cover: **what changed**, **why**, and **how it
  was tested** (dry-run logs, before/after for plugin UX, etc.).

## Markdown style

- GitHub-flavored markdown.
- LF line endings (enforced by `.gitattributes`).
- No trailing whitespace.

## Questions

Open a discussion or ping in [Claude Developers Discord](https://anthropic.com/discord).
