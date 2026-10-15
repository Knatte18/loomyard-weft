---
verdict: APPROVED
---

## Measure

Board entry `seed-ide-interactive-chain` with its operator decision of 2026-10-15; the decision record holds no `Added after Discussion` entry, so the board body is the measure.
Diff: pull request 635 at `d6fc5b7c28ca37cf816de86b74577d3ab1431ea7` against `main`, one commit.

## Alignment

- The folderOpen chain is gone: `internal/vscode/config.go` deletes `TaskChain`, `TaskChainInteractive`, `TaskChainAttachOnly`, the `reed up` and `reed add claude` tasks and the `Start Claude` folderOpen entry; nothing in the generated config runs `reed add claude` from VS Code.
- One seeding shared by every caller: `ideengine.Spawn` and `ideengine.SeedTask` both call `writeWorktreeConfig`, and `WriteConfig` writes one tasks.json shape; no duplicated seeding code is added.
- The operator's view on a `lyx reed attach` shortcut is met by one on-demand `reed attach` task with no `runOptions`.
- Docs follow in the same commit: `docs/overview.md`'s ide row and `internal/loomcli/doc.go`'s operator-session paragraph no longer describe the chain; a grep of the PR head finds no `folderOpen`, `TaskChain`, `Start Claude` or `writeInteractiveConfig` left in Go, Markdown or YAML.
- Nothing outside the entry is changed: reed's `add` flags, keybindings seeding, the hub workspace file and batten's rows are untouched.

## Open risks carried over

- A worktree already holding the old chain keeps it until its next `lyx ide spawn` or Seed-IDE.
  It names no parent correction, and the entry asks for nothing beyond the regenerate-on-write that the change keeps.

## Overlaps

- Peer `shed-tackler` (row 15): both change `docs/overview.md` and `internal/loomcli/doc.go`.
  Its decision record moves the Go driver and tackler into `internal/sheddriver` and says nothing about VS Code seeding, the ide row or the operator-session paragraph, so the overlap is text-only and the designs do not conflict.
  Landing order: this run (`own_row` 22) is further along and lands first; `shed-tackler` takes these changes at Merge-In.
- No parent commits are listed.

## Superseded

None: no other board entry is made obsolete by this run.
