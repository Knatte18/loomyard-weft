# Card 29 — finalize-gate-and-fixer

**Create:**
- `plan:internal/landingshed#Finalize.gateAndFix` -> `func (fz *Finalize) gateAndFix(ctx context.Context) (shedengine.Outcome, shedengine.OutputPointer, error, bool)`
- `plan:internal/landingshed#TestFinalize_GateAndFixer` -> `func TestFinalize_GateAndFixer(t *testing.T)`

**Edit:**
- `internal/landingshed#Finalize`
- `internal/landingshed#NewFinalize`
- `internal/landingshed#Finalize.Call`
- `internal/landingshed#Deps`
- `internal/landingshed#TestFinalizeVerify_Fail`
- `internal/landingshed#TestFinalizeVerify_DirtyTreeHalts`
- `internal/loomcli#landingDeps`
- `internal/loomcli#intentionallyNil`
- `internal/testkit/envkit#LandingDeps`

**Uses:**
- `plan:internal/landingshed#newFixerRun`
- `plan:internal/landingshed#FixCap`
- `plan:internal/landingshed#ReadFixerAttempts`
- `plan:internal/landingshed#RemoveFixerAttempts`
- `plan:internal/landingshed#FixerAttemptsPath`
- `plan:internal/landingshed#LandingStopPrefix`
- `plan:internal/landingshed#fakeFixerSeats`

**Extends:**
- `internal/landingshed/doc.go`

**Intent:** Finalize runs the `landing` gate and, on a failure, the merge fixer in `finalize` mode (D10 step 4).
`gateAndFix` runs the `landing` gate through `verifytree.Verify`, which skips a tree it already verified, so a run that went right holds the lock for one merge, a skipped gate and the landing; a darn Finalize over a diff wholly under `tree_scope` runs the tree-scoped command, since darn's verify source already answers it.
A passing gate removes the landing failure record and a failing one writes it (D12).
On a failure it spawns card 11's fixer in `finalize` mode, which fixes failing tests only, on the attempts `landing.yaml`'s `finalize_fix_attempts` leaves the entry, reading the attempt record under `Deps.RowEntry("Finalize")`.
The pre-fix HEAD is the entry's: the first spawn writes the HEAD before the fixer's first turn into the attempt record's `Baseline`, every resume of the entry keeps it, and the fixer's whole change is judged by `FixCap` over `LineChange(Baseline, HEAD)` with `finalize_fix_max_lines` and the run's language matcher, so turns that each stay under the cap but sum past it are over it.
An attempt record that does not decode is a Stuck with Halt naming the file, with the way forward to remove the file by its path, which grants the entry a fresh cap, then `lyx loom resume`.
A spent cap, or a change over the cap, keeps the fixer's commits and reroutes through `rerouteReason`, naming the cause, so the way back reviews the fix.
Where the way back reaches no review, a spent cap or an over-cap change halts with `LandingStopPrefix` instead, naming the fixer's commits, with the way forward to review and fix on the task branch, then `lyx loom resume`.
Whether the way back holds a review is the new told `Deps.RerouteReviewed`: for loom, the Code-Review perch's effective budget, `code_max_bounces` or the `review_max_bounces` it falls back to when null, is above 0; for darn, Finalize's `on_reroute` is Darn-Bouncer, that is `darn.yaml`'s `review_max_bounces` is above 0, and not the budget-0 rewrite's `PR-Review`; `landingDeps` fills it per recipe from the config at call time; a nil seam reads as no review, so a struct-literal Finalize halts rather than reroutes an over-cap fix.
A `finalize`-mode `escalate` verdict keeps the fixer's commits and halts with `LandingStopPrefix`, naming the report, with the way forward `lyx loom decision add` in the task worktree, then `lyx loom resume`.
The fixer's other failure exits halt with card 11's reasons, and a failed spawn is an error stop that spends no attempt.
A Finalize reroute leaves the entry's attempt record, which its next entry clears; a Done clears it.
The size and kind caps are the bound on what an unreviewed Finalize fix lets through: a fix within them lands with no perch, behind only the `landing` gate it passed and the `finalize` mode's instruction to fix failing tests only.

`TestFinalize_GateAndFixer` drives `Call` with `fakeFixerSeats` and pins: a verified tree runs no gate and proceeds to the landing; a darn Finalize over a diff wholly under `tree_scope` runs the tree-scoped command its verify source answers, not the `landing` call; a failing gate spawns the `finalize` fixer and a `fixed` change within the caps proceeds; two turns of 30 lines each, each under `finalize_fix_max_lines` but 60 together, reroute as over the cap, the recorded `LineChange` `from` being the entry's `Baseline`; an undecodable attempt record halts naming the file and its way forward; a change past `finalize_fix_max_lines`, a spent `finalize_fix_attempts`, a 3-line change editing a `_test.go` file under a Go matcher and a `FooTests.cs` file under the darn union each reroute with the commits kept; under a `none` plan a small fix to any file reroutes; with `RerouteReviewed` false the over-cap change halts with `LandingStopPrefix` instead; `escalate` halts naming the report; after a reroute on a spent cap the next Finalize entry starts at zero spent attempts, while a resumed entry keeps its spent ones.
The verify tests assert the fixer spawn where they asserted a Stuck.

`doc.go`'s Finalize section describes the gate, the fixer, the caps and their bound.

**ImpactSummary:** Finalize's failed-gate Stuck becomes the fixer loop with its caps, and one told seam is added and filled by the landing assembly and the test kit.
