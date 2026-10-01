# Card 58 — fresh-and-plan-path-evidence

**Edit:**
- `internal/websterengine/runlevel.go`
- `internal/websterengine/auditledger.go`
- `internal/websterengine/suspect.go`
- `internal/websterengine/planbaseline.go`
- `internal/websterengine/doc.go`
- `internal/webstercli/run.go`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/suspect_test.go`

**Uses:**
- `internal/websterengine/fingerprint.go`
- `internal/websterengine/gitwrap.go`

**Intent:**
Fix the `--fresh` half of PR #321 round-4 finding A and the rest of finding D's second and third items.

- `freshPendingDrop` checks only that the suspect paths are restored in the worktree, so HEAD may keep every unaudited commit while `--fresh` drops the findings.
- It also leaves plan paths out of its check, so a card Master or a fork rewrote is dropped with the findings and becomes the new run's plan with no evidence.
- Every differing-path clause names a git restore, which cannot restore a plan file (card 57).

In `planbaseline.go`, `planBaselineHas(websterDir, hash string) (bool, error)` reports whether the store holds a copy for `hash`.

In `suspect.go`, `planPathClause(next string) string` is the way forward for differing plan paths: `restore them with "lyx webster restore-plan", or accept an edit to a card no batch has begun with "lyx webster rebaseline --card NN"; then re-run <next>`.
`checkRecoveredSuspects`' plan reason appends `; way forward: ` plus `planPathClause("lyx webster recover-batch NN")`, NN filled from the batch.

In `auditledger.go`, `AcceptPendingAudit` splits its differing paths into plan paths and the rest.
Plan paths get `planPathClause(`"lyx webster accept-audit"`)` and never a `git checkout`;
the rest keep the git clause.

In `runlevel.go`:

- `pendingAuditFindingsError(pending []PendingAuditFinding, geom Geometry) error` and the run-exit stuck reason name plan paths separately with `planPathClause`, the git clause covering only the other paths.
  A link-resolution error while sorting them is returned as is.
- `freshPendingDrop`, once there is anything to drop:
  - When a start commit is known, the worktree's HEAD must equal it.
    Otherwise it refuses, wrapping `ErrPendingAuditFindings`, before anything is archived: `--fresh would drop pending audit findings while HEAD <head> is not the run's start commit <start>; way forward: reset the branch to <start> with git, then re-run "lyx webster run --fresh"`.
  - Plan paths go through `checkSuspectPaths` with the rest.
    A differing plan path refuses with `planPathClause(`"lyx webster run --fresh"`)` when the store holds the copy `restore-plan` needs (a recorded file) or none is needed (a file the run never recorded);
    one whose recorded copy is missing cannot be restored by any verb, so it is dropped with the archived state like an unverifiable path, and its drop warning says so.
  - The doc comment drops "a plan file differs by design" and states both rules.
- `doc.go`'s run-entry paragraph states the HEAD rule and the plan-path rule.

`internal/webstercli/run.go`: the `--fresh` flag usage and `Long` add that it refuses while HEAD is not the run's start commit or a pending plan path differs from the plan the run recorded.

Tests:

- `runlevel_test.go`, over the existing `--fresh` fixture:
  - `TestRun_FreshRefusesCommitPastStart`: the suspect file restored in the worktree but HEAD one commit past the start → `ErrPendingAuditFindings` naming the start commit, `state.json` and the reports dir not archived.
  - `TestRun_FreshRefusesDifferingPlanPath`: a pending finding naming only an edited card file, HEAD at the start → `ErrPendingAuditFindings` naming `lyx webster restore-plan` and not `git checkout`;
    after `RestorePlan` the next `Run` with `Fresh` re-initialises and drops the finding.
  - `TestRun_FreshDropsPlanPathWithoutCopy`: the same with the store emptied → the run re-initialises, and the drop warning names the card.
  - `TestRun_PendingPlanPathNamesRestorePlan`: run entry without `--fresh` over a pending plan-path finding → the refusal names `lyx webster restore-plan` and not `git checkout`.
- `suspect_test.go`: `TestAcceptPendingAudit_PlanPathNamesRestorePlan`: a pending finding naming an edited card → `ErrAuditNotAcceptable` whose text names `lyx webster restore-plan` and `rebaseline --card` and does not contain `git checkout`.

**ImpactSummary:** run --fresh refuses while HEAD is not the run's start commit or a pending plan path differs from the recorded plan, and every plan-path way forward names restore-plan or rebaseline --card instead of a git checkout that cannot work.
