---
format: 5
approved: true
language: go
---

# Plan: no run step edits its own task's board entry

No step of a task's run is asked to change that task's own board entry's body or scope, which the run's board lock refuses.
The documentation-lifecycle wording stops implying the building task cuts its own entry, and the Discussion, Plan and darn writers, with their reviewing rubrics, each state and check the rule; the board lock and `lyx board upsert` stay unchanged.

## Card Index

1 — doc-lifecycle-wording — `CLAUDE.md` and `docs/overview.md` say the building task leaves its own entry to the landing and prune
2 — discussion-own-entry-rule — the Discussion stencil and its rubric forbid and flag a decision that edits the own entry
3 — plan-own-entry-rule — the Plan stencil and its rubric forbid such a card and route a decided own-entry step's drop through the parent
4 — darn-own-entry-rule — the darn write stencil and its rubric forbid such a step and route its omission through the parent

## Shared Decisions

Every new rule says "body or scope" of the running task's own entry: the landing's status write, a `run_settings` write, a write to another entry and a change to board-module code are neither forbidden nor flagged.

## verify:

go build ./... && go vet -tags integration ./... && go vet -tags tmux ./... && go vet -tags llm ./... && go test ./... && go test -tags integration ./...
