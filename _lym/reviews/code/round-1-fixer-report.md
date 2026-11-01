# Fixer-report, round 1

## Changes

- **F1** fixed as proposed, with one deviation noted below.
  `FixerAttempts.Accepted` became `AcceptedHead`.
  `AcceptFixReview` takes a `taskHead` function and records the HEAD at accept time; `loomcli/circling.go` passes the task worktree's HEAD.
  `judgeFixerChange` proceeds only when `AcceptedHead` equals the current head, and a stale acceptance is reset with its rounds and findings so the fix reviews run afresh.
  Deviation: the proposal says to take the HEAD after the description commit moves it.
  The description is committed to the weft repository, not the task branch, so that commit never moves the task HEAD, and the head is read directly.
  Tests: a seeded accepted record with a failing then passing gate and a new fixer change runs the fix review; acceptance at an earlier HEAD is reviewed; a rejection after a stale acceptance goes back to the fixer, which fails if the reset is removed.
- **F2** fixed as proposed: a seeded pending-`FixReview` resume row (fixer spawned on that path over a passing gate) and a row where the review fails to start, asserting no pending `FixReview` stays on disk.
- **F3** fixed as proposed in `contracts/specs/loom-plan-spec.md`.
- **F4** fixed as proposed: retry row (heading present), missing-description row, head-read-failure row, and the no-verdict finding asserted on the fix review end table.
- **F5** fixed with a different placement.
  The named file `tapestryrecipe_test.go` does not exist, so the assertion is a new `TestRecipe_FinalizeDeclaresNoReroute` in `internal/loomrecipe/recipe_test.go`, which also requires the tapestry recipe to have a Finalize row.
- **F6** fixed differently from the proposal.
  The proposal's Webster swap made the fakes say a Webster halt is handed back and a Finalize halt is not, the reverse of production (`HandsBack` hands back a Finalize stop opening with `LandingStopPrefix`).
  The fakes in `battenshed`, `shedverbs` and `sheddriver` now model a Finalize `landing stop: ` halt, which `HandsBack` still produces, with Webster as the negative row.
- **F7** fixed as proposed in `docs/overview.md`.
- **F8** fixed as proposed: the second call of the spent-cap test is dropped and the test renamed.
- Doc comments on `gateAndFix` and `doc.go` now name the HEAD the acceptance is bound to (self-check).

## Self-check

- fix-consistency: no findings.
- fix-code-correctness, LOW, reset in `judgeFixerChange` untested: fixed as proposed (new rejection row).
- fix-code-correctness, LOW, stale `gateAndFix` doc: fixed as proposed.
- fix-code-correctness, LOW, a parent move between accept and resume discards the acceptance: disputed, `every fix worse`; see Disputed.
- fix-affected-set, LOW, stale `doc.go:96` and `finalize.go:620` comments: fixed as proposed.
- fix-proposal, MEDIUM, F6 fakes invert production: fixed with the own change above.
- fix-proposal, LOW, head-failure subtest does not assert the record: fixed as proposed.
- fix-proposal, LOW, F5 assertion in a Webster-named test: fixed as proposed (own test).
- fix-proposal, LOW, F1 acceptance lost when a resume moves HEAD, and the old `accepted` key: disputed, same ground as above.

## Disputed

- **Acceptance discarded when a resume merges a moved parent** (fix-code-correctness 3, fix-proposal 4), ground `every fix worse`.
  The result is more fix review rounds, never an unreviewed landing.
  `Reviewed == head` in the same function has the same property, and any rule that tolerates a changed HEAD would let a changed tree land on a stale acceptance, the defect F1 reports.
  A record written with the old `accepted` key loses its acceptance the same safe way.

## Deferred

None.
