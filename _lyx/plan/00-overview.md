---
format: 5
approved: false
language: go
---

# Plan: board model — tasks and notes with labels, and GitHub intake

The board becomes the operator's one list.
Every entry is a task or a note (`kind` replaces `tier`), classified by `labels` validated against two lists in `board.yaml` (replacing `type` and bracketed brief prefixes).
`list` and `find` filter by label, the README groups notes by type label, `upsert` reads a body from a file or stdin, and old `board.json` entries migrate on load.
A new `lyx board intake` group lists open inbox issues not yet on the board, imports one as a note or folds it into an entry and closes it with a pointer, and closes noise with a stated reason; the hidden `notes` and `promote-note` aliases go.

## Card Index

1 — board-label-vocabulary — board.yaml gains the types and labels lists, read through a Vocabulary
2 — board-alias-removal — delete the hidden notes group and promote-note verb
3 — board-kind-and-labels — kind and labels replace tier and type across the model, render, listing and promote
4 — board-load-migration — old-shape board.json entries convert on load and persist on the next write
5 — board-label-filter — list and find take a repeatable --label filter
6 — board-upsert-body-file — upsert reads its body from --body-file or stdin
7 — inbox-github-calls — selfreportengine lists, fetches, comments on and closes inbox issues
8 — board-issue-records — the issues field and the store-side import and fold
9 — board-intake-verbs — the lyx board intake list, import and close verbs
10 — board-model-docs — package doc, overview, skill and sandbox docs, and the D13 label list

## Shared Decisions

### Decision: one vocabulary, carried by the Board

- **Decision:** `Vocabulary` (card 1) is the one answer to "is this a type label" and "is this label configured".
  `Board` copies it from its `Config` and sets it on every store it builds; a store without one exists only in store-level tests.
  The renderer gets the type order through `Outputs.Types`.
- **Rationale:** keeps `boardengine` a strict config caller that never reads config files itself, and keeps label rules testable on a bare `Store`.
- **Applies to:** cards 1, 3, 4, 5, 8

### Decision: a path-only Board is the --board-path shape

- **Decision:** "path-only" means a `Board` whose outputs are not configured, the predicate `boardCriticalSection` already uses to refuse writes.
  It never validates labels: it cannot write, and its `--label` filter is not checked.
- **Rationale:** D3's and D6's bound in one predicate, with no second notion of "built from config".
- **Applies to:** cards 3, 5

### Decision: every refusal names the way forward

- **Decision:** a label refusal names the entry, the label and `board.yaml`; a kind refusal names `lyx board set-deps` or `lyx board promote`; an intake refusal names the payload change that works.
- **Rationale:** the decision record asks it of every refusal; an agent reading the error must be able to act on it.
- **Applies to:** cards 3, 5, 6, 8, 9

## verify:

go build ./...
go test ./...
go test -tags integration ./internal/boardengine/... ./internal/boardcli/... ./internal/selfreportengine/... ./internal/loomcli/... ./internal/battencli/... ./cmd/lyx/...
go test -tags integration -run '^$' -bench . -benchtime 1x ./internal/boardengine/boardtest/
go vet -tags smoke ./internal/loomcli/...
