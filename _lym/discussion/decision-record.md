## Goal

Every module under `internal/` falls under a board module label, so an entry's label exists when the entry is written.

## Scope

- In: one new label per uncovered package group under `internal/`, each description naming its packages.
- Out: any change to the repository's code, template or tests.
  Labels live in the hub's `_board/_lym/config/board.yaml`, outside this worktree, and the repository template ships `labels: {}`.
- The parent applies the labels with `lym config board --set`; this run edits no file.

## Decisions

### D1 — Labels are added by the parent, not this run

Rationale: the label list is hub config outside the worktree, and the parent chose to apply it from a list this run sends.

### D2 — One label per package group, named for the module

Rationale: groups follow the packages' shared concern, such as `tender` for `internal/tendercli`, `tenderengine` and `pauserecord`.
Packages already covered by an existing label get no new one.

## Constraints

- No edit outside this worktree.
- A label description names its packages, as the existing ones do.

## Auto-mode assumptions

- The grouping of small support packages into `cwd`, `fsutil`, `cli` and `process` is this run's judgment; the operator may split or rename them.

## Open risks

- A package added later has no label until someone adds one.

## Acceptance criteria

- D1: the parent receives the full label list and applies it; this run changes no file outside its worktree.
- D2: each uncovered package under `internal/` appears in exactly one new label's description.
