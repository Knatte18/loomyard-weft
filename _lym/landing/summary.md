# Merge main into the codex-engine beam

Brings `origin/main`'s nine commits since the fork point `7f0f65b3` into the `codex-engine` beam, so the beam's landing merges cleanly.
The merge adds no new behaviour.

## What came in from main

- `loomcli`: a pre-push check of origin and owners before a run commits or pushes (`prepush.go`), wired into `loomCommitStatusDeps` in `internal/loomcli/wiring.go`.
- hibernate: the new hub-wide `hibernate.yaml` config (`internal/hibernateconfig`), registered in `configreg`, and lifting the hub pause on wake only when that config allows it.
- batten and dobby: a stray dir reads as a gone pair, a beam landing keeps its mark, and a paused run whose work already reached its parent is finished.
- `beam create` clears a done motif run under the same slug.
- `fabric remove` deletes worktree dirs refilled with ignored files.
- `gatecli` ends a tier whose shell exits while an orphan holds its pipe.
- `loom status --watch` stops printing a line per poll.
- Way-forward texts name a `lym board remove` command the verb accepts.

## How the two sides met

Git merged every file without a conflict.
Four files changed on both sides, and each merged result keeps both changes:

- `internal/loomcli/wiring.go`: main's pre-push guards on `Commit`, `Spawn` and `Push` sit beside the beam's `DefaultEngine` fields and the assistant reviewer's segment bounds and settings.
- `internal/configreg/configreg_test.go` and `cmd/lym/configstrictness_test.go`: main's `hibernate` module rows; the beam's edits to these files touch other lines.
- `docs/overview.md`: main's hibernate module row and section, and the beam's `codexengine` text in the shuttle section.

`PATTERN.md` carries main's `PATTERN-owner-machine` change beside the beam's `PATTERN-engine-blocks` entry and its `CreateFileLink` addition to `PATTERN-fs-links`.
Main changed no stencil the beam moved into an engine block, and none of the beam's code that main's commits read changed under it.

## Verification

`go build ./cmd/lym` and `go vet` over every package either side changed since the fork point both pass on the merged tree.
