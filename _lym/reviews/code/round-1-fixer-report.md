---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
---

# Fixer-report, round 1

## Changes

All four findings fixed as proposed, committed as `1fb2825a` and `1ba9c922` (the four fixes touch shared files, so F1-F4 share the first commit).

- F1: `stepListAndFindText` asserts the exact `list --text --label bug` output, with no P0 row.
- F2: `withRootProjectRow` also matches a done entry, through the new `BriefTask.IsDone` in `internal/boardengine/store.go`; its doc comment, the list help text, `doc.go` and the test comment are reworded.
  `stepListAndFindText` marks every entry done and asserts the P0 row stands ahead of them.
- F3: under `--sort` the list command appends the root row last; the help text and `doc.go` say so, and the test asserts P0 is the last line under `--sort age`.
- F4: the `stepProjects` doc comment says `parent_beam` is root under a grouping project.

The gate (`lym gate test ./internal/boardcli ./internal/boardengine`) passed.

## Self-check

- fix-consistency, LOW: `doc.go:68` kept the old placement wording. Fixed as proposed.
- fix-consistency, LOW: stale comment at `verbs_test.go:346`. Fixed as proposed.
- fix-affected-set, LOW: same `doc.go` twin as fix-consistency, merged with it. Fixed, and the `--sort` rule added there.
- fix-code-correctness: no findings.
- fix-proposal: no findings.

## Disputed

None.

## Deferred

Nothing deferred.

## Rulings

None.
