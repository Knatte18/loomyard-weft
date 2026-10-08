# Card 11 — reset-targets

**Create:**
- `plan:internal/websterengine#ResetToReportHead` -> `const ResetToReportHead ResetTarget = "report-head"`
- `plan:internal/websterengine#ResetToLastBatchHead` -> `const ResetToLastBatchHead ResetTarget = "last-batch-head"`
- `plan:internal/websterengine#ResetToBatchStart` -> `const ResetToBatchStart ResetTarget = "batch-start"`

**Edit:**
- `internal/websterengine/reset.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/reset_test.go`
- `internal/websterengine/reset_integration_test.go`
- `internal/webstercli#websterCLI.resetCmd`
- `internal/webstercli/reset_test.go`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/websterengine#runEvidenceBases`
- `internal/websterengine#ReportFileName`
- `internal/websterengine#ParseReport`
- `internal/websterengine#StrandLive`
- `internal/websterengine#RunActive`

**Intent:** `lyx webster reset --to` gains three targets, each a commit the run recorded, so every recovery site can name a lyx verb instead of a raw `git reset --keep` (D1).
`PlanReset` takes the batch number beside the target (zero when none) and validates the pairing first: `--batch` is required for `report-head` and `batch-start` and refused for `start`, `pre-fix` and `last-batch-head`, each refusal naming the correct spelling.
The new targets resolve:
- `report-head --batch NN`: the `head_sha` of batch NN's report on disk (`ReportFileName`, `ParseReport`).
  Accepted only when batch NN is begun and not terminal, its report is on disk and parses, its head descends from or equals the batch's recorded `StartSHA`, no batch after NN in the recorded partition's order has a record, and no recovery strand of batch NN is live in reed (`ResetDeps` gains the reed seam; `StrandLive` answers).
  It is the one target allowed while the run lock is held, because Master calls it inside its run between the fork's report and record-batch; every other target keeps the `ErrRunBusy` refusal.
- `last-batch-head`: `runEvidenceBases`' last recorded batch head, the commit accept-audit picks by git ancestry.
- `batch-start --batch NN`: batch NN's recorded `StartSHA`; refused when any batch after NN in the partition's order recorded a start, so it never discards another batch's commits.
`last-batch-head` and `batch-start` refuse when the state recorded no such commit, naming `lyx webster reset --to start`; `batch-start`'s later-start refusal names `--to start` too.
Each `report-head` refusal names its way forward (a terminal batch: `--to last-batch-head`; no report: wait for the fork's report or run `lyx webster recover-batch NN`; a head that does not descend from the start or a later begun batch: `--to start`; a live recovery strand: wait for its `recover-batch` to finish, then re-run).
Every target keeps `PlanReset`'s existing guards after resolution (merge in progress, foreign branch, missing commit, target not an ancestor of HEAD, foreign dirty paths) and the pair reset through fabric's `ResetPairCode`, so `report-head` moves the remote task branch under that function's own bound (only the task branch, only dropped commits reachable from HEAD, a lease on the remote tip).
Bound, `report-head` under a held lock: the three checks above rule out every writer lyx can see; a Master that spawned an in-session fork out of order stays unseen, the residue the decision record's open risks name.
`resetCmd` gains `--batch NN`, accepts the five targets in its `--to` validation and usage line, passes the reed engine, and its long help states each target, the `--batch` pairing and the run-lock exception.
`reset_test.go` gains one table test over the new targets' resolutions and every new refusal (including the `--batch` pairing and the held-lock acceptance of `report-head` and refusal of the others); `reset_integration_test.go` moves a real HEAD for each target; `webstercli/reset_test.go` covers the flag pairing and the envelope's `target`.
`refusal-spec.md` gains a row per new refusal, and the package doc's reset section lists the five targets.

**ImpactSummary:** Extends PlanReset's signature and the reset verb's flags; start and pre-fix behave as before, and the one caller is the reset verb.
