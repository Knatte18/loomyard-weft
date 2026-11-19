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
  - id: F5
    disposition: fixed
---

## What changed

- F1: `internal/stencilstore/skew.go` no longer returns shipped bytes.
  `refuseNewerBoardCopy` returns a `*NewerBoardCopyError` naming the stencil, the writer's revision, the running revision and the way forward.
  `Read` (`reconcile.go`) and `readInclude`'s engine-block path (`engineblock.go`) propagate it.
  The registry is only compared against; `newerCopyWarned` and the Warn log are gone.
  The `Read` doc, `doc.go` and the `skew.go` header describe the refusal.
- F2: `skew_test.go` is now an external-package table test over a `t.TempDir()` board, through `Read`, `ReadForEngine` and `ReadSet`.
  `export_test.go` adds `SetSkewStateForTest`; the test names its process-global state and is not parallel.
  Rows cover older, equal-time, no-writer, shipped-body, no-commit-time and unshipped copies, plus the ordering rows from F4.
  Every row asserts the board file is unchanged.
- F3: the refusal applies to edited and untouched copies alike; the board file is never written.
- F4: `RegisterOrdering` takes the refresh's `Older`; `cmd/lym/stencilseed.go` registers `StencilSource(...).Older`.
  Without an ordering the read path falls back to commit time.
- F5: the `Registry` doc in `contracts/stencils/stencils.go` states that the package registers with stencilstore at init.

Commits: the findings share `skew.go`, `reconcile.go` and the test, so they landed in one commit, with a second for the self-check fixes.
A third commit repairs the gate's comment-style findings:
the wrapped or 120-character comment lines in `contracts/stencils/stencils.go`, `doc.go`, `reconcile.go`, `skew.go` and `skew_test.go`
were re-broken at sentence boundaries, one sentence per line.
A fourth commit re-words the two `skew.go` comments (`RegisterOrdering`, `refuseNewerBoardCopy`) the gate still flagged as wrapped,
so each sentence stands on its own line.

## Self-check

- fix-code-correctness, MEDIUM, git work ran before the cheap body comparison: fixed (comparison first).
- fix-code-correctness, LOW, ordering registered only behind the seed pass's gates: disputed, every fix worse (see Disputed).
- fix-consistency, LOW, "Init" capitalised in the `Registry` doc: fixed.
- fix-consistency, LOW, `doc.go` never says `RegisterOrdering` shares `Source`'s `Older`: fixed.
- fix-proposal, MEDIUM, ordering before body hash: duplicate of the first line, fixed.
- fix-proposal, MEDIUM, way forward wrong for a fresh older binary: fixed differently from the review's wording; the error now also says to update the binary first if it is older than the board.
- fix-proposal, LOW, registration behind the gates: same as above, disputed.
- fix-affected-set, LOW, registration behind the gates: same as above, disputed.

## Disputed

- Ordering registered only in `seedStencils` (fork findings from three lenses), ground `every fix worse`.
  Registering ahead of the gates needs the worktree path, which means resolving geometry with `git rev-parse` in processes that opt out for exactly that reason.
  Evidence: `cmd/lym/stencilseed.go:43-50`.
  Those processes still fall back to commit-time ordering, and the affected-set fork found no stencil-reading driver among the opt-outs.

## Deferred

None.

## Rulings

None.
