---
format: 5
approved: true
language: go
---

# Plan: the board becomes the roadmap

One store, `board.json`, replaces `tasks.json` and `notes.json`; every entry carries a numeric `tier` and a closed-set `type`, and the README renders one section per tier like `manifest/roadmap.md`.
The store migrates itself from the legacy files on load, persists on the first write, and folds a pre-upgrade binary's done marks, so a long-running old driver keeps working until `retire-legacy`.
Agents get `promote`, `prune`, `find`, a `--text` listing, full payload help, and a `ly:board` skill.

## Card Index

1 — entry-schema — add tier and type, rename the recipe field, retire deferred, and enforce the tier dependency rule
2 — store-ops — add promote, prune and find on the in-memory store
3 — legacy-migration — pure migration and done-mark fold from the legacy files
4 — board-json-store — switch to one board.json store with migration on load and collapse the two-store facade
5 — board-verbs — expose promote, prune, find and retire-legacy on the Board facade
6 — render-roadmap — render the README per tier and give design docs a header
7 — cli-verbs — add the new CLI verbs and --text, and turn notes and promote-note into hidden aliases
8 — cli-help — document every payload verb's keys in its --help
9 — board-skill — add the ly:board skill and list it in the ly index
10 — overview-doc — update the board entry in docs/overview.md

## Shared Decisions

### Decision: the record keeps the name Task

The stored record stays the Go type `Task`, the facade methods keep their names (`UpsertTask`, `GetTask`, `SetStatus`, `ListTasksBrief`), and the JSON envelope keys stay `task`/`tasks`.
External callers (`internal/loomcli`'s `MarkTaskDone`, `internal/battencli`'s `ReadBoardType`, `internal/ideengine`'s menu) and the discussion stencil's `{"task": ...}` read stay unchanged apart from the recipe field.

### Decision: tier numbers map to words only in the renderer

The data holds `tier` as 1, 2 or 3; the names Planned, Next Up and Someday and their meaning lines are declared once, in `render.go`.
The skill and help text describe the meanings in prose but never become a second lookup table in code.

### Decision: board.lock stays the write lock

Every write, the fold included, runs under `board.lock`, the same lock a pre-upgrade binary takes before writing a legacy file, so an old-binary done mark and a new-binary fold serialize.

### Decision: tests build boards in temp dirs

No test touches the real `_board`; engine tests use temp dirs, CLI tests reuse `seedCwd`, and hub-level tests build through `internal/hubforge`.

## verify:

```
go build ./...
go test ./...
go test -tags integration ./internal/boardengine/... ./internal/boardcli/... ./internal/battencli/... ./internal/battenshed/... ./internal/loomcli/... ./cmd/lyx/...
go vet -tags smoke ./internal/boardengine/... ./internal/boardcli/... ./internal/battencli/... ./internal/battenshed/... ./internal/loomcli/... ./cmd/lyx/...
```
