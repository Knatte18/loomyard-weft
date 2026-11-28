---
dispositions:
  - id: F1
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed-differently
    reason: "ImplementerSeam.LiveStrands is filled from the replayed reed as proposed. The batten child seams are not: in production they read or resume the child pair's reed through battencli's own wiring over a resolved task worktree (internal/battencli/wire.go:1374), which envkit cannot reach, so the Design decisions say why they keep their fixed answers."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed-differently
    reason: "A copy of the recorded paste with its Stdin replaced would write new content into a recording, which pattern/PATTERN-test-stand-ins.md forbids. The coverage is a round trip in record_test.go instead: the call tmuxrec's own placeholding holds is replayed under another hub and must answer."
  - id: F7
    disposition: fixed-differently
    reason: "newSession makes its hub and loads the reed config in one step, so a test cannot run configengine.Set in that hub first without a new hook. The test sets LYM_REED_SHELL, which the reed template itself reads, and so writes no config file."
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed-differently
    reason: "The Blast radius line takes one word (internal/summaryparser/description.go:143), so it stays testkit and the wider reach is stated in sentences below it."
---

# Fixer report, code round 1

## What changed

- **F1** (`2a4584bd3`): the mail rows take Mode, Origin, TaskHead and MailAgent from `landingDeps` over the real pair.
  `pull_request` is set through `configengine.Set` on beam's spec, and the pair's origin URL through git.
- **F3** (`2a4584bd3`, `700fe8242`): the rows check that nothing is pushed, compare the whole record, bound every sleep and check that no parent lock is held.
  The wait mark is not checked on these rows; `700fe8242` carries its `Loosened-Check` line.
- **F4** (`e81a66f3c`, fixed differently): `reedengine.LiveStrandNames` is the one reader of live strand names, used by loom's wiring and by `envkit`.
  `TestSessionReads` asserts it.
  The batten child seams keep their answers, as the Design decisions now say.
- **F5** (`5be7de4e6`, `73f8c06f3`): a paste's text holds only the hub directory as a placeholder, and the package doc says so.
- **F6** (`5be7de4e6`, fixed differently): the placeholding test replays the held paste under another hub.
- **F7** (`832904d2e`, fixed differently): `session_test.go` pins the bare shell name on a host without the shell.
- **F8** (`32d7d95d6`): the two inventory rows mark the removed stand-ins done.
- **F9** (fixed differently): `_lym/landing/summary.md` states the wider reach under Merge danger.

A last commit breaks the comment lines this round added in `envkit.go`, `record.go` and `record_test.go` at sentence boundaries,
with each line under the length limit.
It changes comments only, and tier 0 passed again over `internal/testkit/envkit` and `internal/testkit/tmuxrec`.

Tests: `lym gate test --tier 0,1` passed over the changed packages and the packages built on `envkit`.
After the self-check fixes it passed again over `internal/loomcli` and `internal/testkit/tmuxrec`.
Tier 2 was not run.

## Self-check

- fix-code-correctness, LOW, expected head read from the seam under test: fixed (`700fe8242`).
- fix-proposal, LOW, stale package doc and literal machine values in a paste's text: fixed differently.
  The package doc is updated (`73f8c06f3`) and the Design decisions record the literal values.
  Replacing those values again would bring back the defect F5 names.
- fix-consistency, LOW, stale package doc: fixed (`73f8c06f3`), merged with the finding above.
- fix-consistency, LOW, stale Design decision on paste text: fixed in `_lym/landing/summary.md`.
- fix-affected-set, LOW, sleep bounds, wait mark and parent lock checks not restored: fixed differently (`700fe8242`).
  Sleep bounds and the parent lock are checked.
  A wait mark check needs a recording closure in `VerifyWaitMark`, a new stand-in for the status pane;
  the rows staying in `TestPRReview_Wait` check the mark.
- fix-affected-set, LOW, `DriverAlive` and `ArbiterInFlight` not named: fixed in `_lym/landing/summary.md`.

## Disputed

None.

## Deferred

None.

## Rulings

None.
