```yaml
lens: code-affected-set
outcome: reported
```

**Per change, the set found**

1. Removed `Deps.ConflictReroutes`, `Deps.LargeMergeLands`, `taskMerger`, `WriteOverlapNote`, `OverlapNotePath`, `overlapNoteInput` and darn's `ReviewInput` override: no remaining reference in code, docs, recipes or stencils; the only hit outside `_lym/` and `docs/research/` is `internal/landingshed/finalize_merge_test.go:117-118`, an intended absence assertion. Covered.
2. `ArbiterSeam.JudgesFirst(producer, reason)`: every assigner and caller is updated (`internal/loomcli/drivewiring.go:134`, `internal/sheddriver/drive.go:719`, `drive_test.go:766`), docs updated (`internal/sheddriver/doc.go:38`, `:60`). Covered.
3. `haltReason(row, escalateJudgedFirst)`: callers `finalize.go:686`, `parentalign.go:393`, `parentalign.go:420` all pass `EscalateJudgedFirst`. Covered.
4. `EscalateStopPrefix` and `EscalateHalt`: readers are `loomcli/arm.go`, `loomcli/drivewiring.go` and `battencli/wire.go`; docs updated in `landingshed`, `arbiterstop`, `battenshed`, `docs/overview.md`, the loom status spec, the landing-fixer report spec and the orch stencil. Covered.
5. `restep_hold`: writers are `loop.go`, `loopenvelope.go`, `status.go`; `shedverbs/doc.go` carries the key list and count; driver stencil, guide and status spec updated; key-set tests in the diff. Covered.
6. Finalize no longer reroutes on a conflict or large merge: recipe comments, orch stencil, `docs/overview.md`, `landingshed` doc, Code-Review rubric and `loomRerouteCapWayForward` updated. Covered.
7. Pawl keeps `LandingStopPrefix` with no `lym loom decision add`: `haltReason` and the orch stencil cover it. Covered.
8. `PATTERN.md` `PATTERN-shed-stop-chain` text matches the declared text. Covered.

**Findings**

- **LOW, consistency.** `internal/landingshed/doc.go:114-117` is the HandsBack paragraph. It says a spent or over-cap fix at Finalize "routes back through the recipe's reroute edge" and "the halt goes to the orch". It was not updated to say an escalate on loom or darn is the exception. The new escalate paragraph at `:122-125` covers it, so this is a minor omission, not a wrong statement.
- **LOW, consistency.** `contracts/stencils/orch/orch-template-role.md:128` says "A landing halt reaches you with the status marked `hand_back`, and no driver, batten or arbiter routes it to another row." The next bullet (`:133`) describes the escalate halt, which is not a landing halt. Consistent; a nit, no change needed.

No other member of any affected set was left wrong. The driver stencil adds `restep_hold` without naming a recipe or reading reason text.

**Tool calls (4 of 6)**
1. grep for `JudgesFirst` and stale large-merge phrases across the repo.
2. `git diff` of `contracts/stencils`, `contracts/specs` and `docs/overview.md`, plus a grep for `hand_back`.
3. grep for `status_moved` and "escalat" in docs and contracts, and a read of `landingshed/doc.go:112-126`.
4. grep for "reroute" in landingshed, `landingdeps.go`, `loomcli/doc.go` and `loomrecipe`.
