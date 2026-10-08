# Card 11 — reset-targets

**Create:**
- `internal/websterengine#ResetToReportHead`
- `internal/websterengine#ResetToLastBatchHead`
- `internal/websterengine#ResetToBatchStart`

**Edit:**
- `internal/websterengine#ResetDeps`
- `internal/websterengine#PlanReset` -> `func PlanReset(deps ResetDeps, to ResetTarget, batch int) (ResetPlan, error)`
- `internal/websterengine#resolveResetSHA` -> `func resolveResetSHA(geom Geometry, st *State, bases evidenceBases, to ResetTarget, batch int) (string, error)`
- `internal/websterengine/doc.go`
- `internal/websterengine/reset_test.go`
- `internal/websterengine/reset_integration_test.go`
- `internal/webstercli#websterCLI.resetCmd`
- `internal/webstercli#Command`
- `internal/webstercli/reset_test.go`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/websterengine#runEvidenceBases`
- `internal/websterengine#ReportFileName`
- `internal/websterengine#ParseReport`
- `internal/websterengine#StrandLive`
- `internal/websterengine#RunActive`

**Intent:** `lyx webster reset --to` gains three targets, each a commit the run recorded, so every recovery site can name a lyx verb instead of a raw `git reset --keep` (D1).
`PlanReset` takes the batch number beside the target (zero when none) and validates the pairing first: `--batch` is required for `report-head` and `batch-start` and refused for `start`, `pre-fix` and `last-batch-head`, each refusal naming the correct spelling.
The new targets resolve in `resolveResetSHA`, and any helper they need sits beside it in `reset.go`:
- `report-head --batch NN`: the `head_sha` of batch NN's report on disk (`ReportFileName`, `ParseReport`).
  Accepted only when batch NN is begun and not terminal, its report is on disk and parses, its head descends from or equals the batch's recorded `StartSHA`, no batch after NN in the recorded partition's order has a record, and no recovery strand of batch NN is live in reed (`ResetDeps` gains the reed seam; `StrandLive` answers).
  It is the one target allowed while the run lock is held, because Master calls it inside its run between the fork's report and record-batch; every other target keeps the `ErrRunBusy` refusal.
- `last-batch-head`: `runEvidenceBases`' last recorded batch head, the commit accept-audit picks by git ancestry.
- `batch-start --batch NN`: batch NN's recorded `StartSHA`; refused when any batch after NN in the partition's order recorded a start, so it never discards another batch's commits.
`last-batch-head` and `batch-start` refuse when the state recorded no such commit, naming `lyx webster reset --to start`; `batch-start`'s later-start refusal names `--to start` too.
Each `report-head` refusal names its way forward (a terminal batch: `--to last-batch-head`; no report: wait for the fork's report or run `lyx webster recover-batch NN`; a head that does not descend from the start or a later begun batch: `--to start`; a live recovery strand: wait for its `recover-batch` to finish, then re-run).
Every target keeps `PlanReset`'s existing guards after resolution (merge in progress, foreign branch, missing commit, target not an ancestor of HEAD, foreign dirty paths) and the pair reset through fabric's `ResetPairCode`, so `report-head` moves the remote task branch under that function's own bound (only the task branch, only dropped commits reachable from HEAD, a lease on the remote tip).
Bound, `report-head` under a held lock: the three checks above rule out every writer lyx can see; a Master that spawned an in-session fork out of order stays unseen, the residue the decision record's open risks name.
`resolveResetSHA` takes the geometry in place of the bare worktree path, since `report-head` reads the reports dir, and the batch number its two batch targets resolve against.
`resetCmd` gains `--batch NN`, accepts the five targets in its `--to` validation and usage line, passes the reed engine, and its long help states each target, the `--batch` pairing and the run-lock exception.
Its `Short` ("move the task branch back to the run's start commit or the verify gate's pre-fix head") becomes a summary that holds for every target, a commit the run recorded, and the `--to` flag description ("start or pre-fix (required)") names the five targets and points at `--batch` for the two that take it.
The webster group help's `Verbs:` line in `Command`'s `Long` names the five targets in place of `--to start|pre-fix`.
Each behavior is tested once, at the surface that owns it, in the lowest tier that holds it:
- `reset_test.go` (untagged, `PlanReset` over the package's `fakeGit`, a reports dir under `t.TempDir()` and a fake reed seam), whose existing tests each pin one pre-run refusal, gains `TestPlanReset_Resolution`, a table asserting the returned plan or refusal, with rows for the `--batch` pairing refusals, which `PlanReset` checks before any git read, and for every refusal that reads only state, report files, the reed seam or a fake `Git`: terminal batch, no report, a head not descending from the start, a later begun batch, a live recovery strand, no recorded commit and a later recorded start; `test-budget.yaml` raises `internal/websterengine` by one for it, and cards 14 and 15 add their rows to it;
- `TestPlanReset` in `reset_integration_test.go` gains steps for each new target's resolved `plan.SHA` over a real repository and the existing ancestry and dirty-path guards reached through a new target, since `PlanReset` is read-only and asserts on the returned plan or error;
- `TestResetCmd` in `webstercli/reset_test.go` gains a step per new target asserting HEAD, files and `mutations` after the move and the `target` and `sha` envelope keys, which also shows `--batch` reaching `PlanReset`, and the held-lock steps: `report-head` moves HEAD while the run lock is held, and `start` is refused under the same lock.
`refusal-spec.md` gains a row per new refusal above (the two `--batch` pairing refusals, each `report-head` refusal, and the `last-batch-head`/`batch-start` no-recorded-commit and later-recorded-start refusals) and rewrites three existing rows: `reset: unknown --to` names the five targets and the `--batch` pairing, `reset: run busy` states the `report-head` exemption, and `reset: target not an ancestor of HEAD` extends its "for `pre-fix`, run `lyx webster reset --to start`" clause to the three new targets, which reach that guard after resolution.
`reset: target commit missing`, `reset: dirty tracked path the run did not write` and the other `reset:` guard rows name `lyx webster reset` without a target and already cover the new targets, so they stay; cards 14 and 15 rewrite the `start` and standalone cases among them.
The package doc's reset section lists the five targets.

**ImpactSummary:** Extends PlanReset's signature and the reset verb's flags; start and pre-fix behave as before, and the one caller is the reset verb.
