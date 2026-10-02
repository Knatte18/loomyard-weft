---
format: 5
approved: false
language: go
---

# Plan: operator-surface-fixes

Deploying lyx and opening worktrees in VS Code must leave every worktree runnable and clean.
A present config file missing a template key loads at the template default instead of refusing (#339);
`lyx reed add --unless-name orch` keeps the folder-open chain from collapsing a live orch pane (#340);
and `lyx ide spawn` regenerates `tasks.json` and keeps `.vscode/` out of git through `info/exclude`, never a tracked `.gitignore` (#334).

## Card Index

1 — yaml-fill-missing — add `yamlengine.FillMissing`, the in-memory merge of a file's missing template keys
2 — config-load-fills — `configengine`'s shared load fills missing keys and logs them instead of refusing
3 — reconcile-hints — rewrite adopt-a-new-key reconcile hints to the default-until-reconcile wording
4 — reed-unless-engine — engine add that is skipped while a named strand is live
5 — reed-add-unless-flag — `lyx reed add --unless-name` with a skip envelope and a real-tmux test
6 — ide-spawn-clean — `--unless-name orch` in the chain, `tasks.json` regenerated, `.vscode/` excluded via `info/exclude`
7 — gitignore-pkg-delete — delete the now-unused `internal/gitignore`
8 — repo-housekeeping — plain section for the repo `.gitignore`'s mill block, drop the fixed CLAUDE.md workarounds

## Shared Decisions

### Decision: fill-never-writes

- **Decision:** the config fill happens only in memory inside `configengine`; `lyx config reconcile` stays the only writer of config files.
- **Rationale:** a load that wrote files would dirty the weft on every lyx call after a deploy.
- **Applies to:** cards 1, 2, 3

### Decision: warp-ignores-via-exclude

- **Decision:** every ignore lyx needs in a warp worktree goes to `.git/info/exclude` through `fabricengine`; no lyx code writes a warp `.gitignore`.
- **Rationale:** a tracked `.gitignore` edit dirties the pair and rides into its commits.
- **Applies to:** cards 6, 7

## verify:

go build ./...
go vet ./...
go test ./...
go test -tags integration ./...
go vet -tags smoke ./...
