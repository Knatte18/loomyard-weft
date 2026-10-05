---
format: 5
approved: true
first_card: 15
language: go
---

# Plan: board labels and skills — rework round 1

The built plan gave board labels descriptions, taught the config machinery open maps, closed the board CLI's gaps and brought `lyx config` and `lyx selfreport` under the CLI tree invariant.
PR review rejected two things.
D9 deleted the interactive `lyx config` picker on auto-assumption A1, which no operator decided: the picker returns as `lyx config menu`, while bare `lyx config` keeps its cobra listing.
And the editor fallback opens vi, which the operator cannot use: after `$VISUAL` and `$EDITOR` it now tries `code --wait`, then `nano`, and `vi` only when neither exists, with notepad still the Windows fallback.
Nothing else in the PR changes.

## Card Index

15 — editor-fallback — after $VISUAL and $EDITOR, prefer code --wait, then nano, then vi; notepad stays the Windows fallback
16 — config-menu — restore the module picker as lyx config menu, listed by bare lyx config

## Shared Decisions

- D9 stands except for the picker's removal: bare `lyx config` lists, an unknown argument refuses before cwd resolution, and the picker lives on as the `menu` subcommand rather than as bare `lyx config`.
- A1 is withdrawn.
- `code` is tried on every OS before the OS fallback, because the order the operator gave puts it first; on Windows the fallback after `code` is notepad, elsewhere it is `nano`, then `vi`.
- Docs land with the card that changes the behaviour: `configengine` and `configcli` `doc.go`, the `lyx config` help and `docs/overview.md`.
- No live `_lyx/config/` file is edited.

## verify:

```bash
go build ./...
go test ./...
go test -tags integration ./internal/configengine/... ./internal/configcli/... ./cmd/lyx/...
go vet -tags smoke ./...
```
