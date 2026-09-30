---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:44
    summary: The once-per-approval resume leaves the marker-vs-Spawn order unstated and gives no distinct signal or recovery when the one resume fails to move the child, so an exempt awaiting wait can run unbounded on a consumed approval
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:76
    summary: The archive-tag abort sits at the weft-branch deletion, which Remove reaches only after portal, launchers and both worktrees are gone, so an abort leaves a half-torn pair; the manual recovery, the remote-false and offline case, and whether force bypasses it are unstated
  - id: F3
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:54
    summary: Done-wait Stuck bounces are counted against the 1440 budget the running watch already spent, so a long child run can block batten with bounce-budget-exhausted on a child that is already done
  - id: F4
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:114
    summary: The grace-elapsed path names the halt but not the resulting batten state or the operator recovery, and the refusal batten surfaces tells the operator to use --force, which discards the uncommitted drive report
  - id: F5
    severity: LOW
    location: _lyx/discussion/decision-record.md:69
    summary: The records pathspec says the friction tree but not whether it covers the timestamped archive siblings Reflect renames the notes into, which would otherwise sit untracked in the weft and make a non-force Remove refuse
  - id: F6
    severity: LOW
    location: _lyx/discussion/decision-record.md:77
    summary: Every weft-branch deletion is broader than the decision means; read literally it also puts an archive tag push, and its abort, inside Add's rollback of a weft branch Add itself just created
  - id: F7
    severity: LOW
    location: _lyx/discussion/decision-record.md:116
    summary: The in-flight-at-deploy risk names the split but not its consequence, that notes left under .lyx/loom/friction/ are never reflected, committed or archived and vanish at teardown; the matching auto-mode assumption only repeats Scope Out
---

### [MEDIUM] Resume-once plus exempt budget has no bound and no signal when the resume fails

**Location:** `_lyx/discussion/decision-record.md:44` (Decision "Batten resumes an approved awaiting child, once per approval"), read with `:49` ("Awaiting-wait bounces do not consume the watch budget").

**Issue:** The Decision says `Run-Shed` "calls `deps.Spawn` … and records that approval's identity", but not in which order, or what a `Spawn` error does to the marker.
Today `Spawn` returning an error is a hard error and resuming the batten run retries it (`internal/battenshed/innerrun.go`, the `spawnErr` arm).
If the marker is written before or regardless of the `Spawn` result, the resumed batten run sees a consumed approval and never retries.
Second case: `Spawn` succeeds but the launched driver dies before its first step.
The child stays `awaiting` at Publish, the marker holds the current identity, every later bounce is exempt, and the Stuck reason still carries the child's old awaiting reason plus the generic `lyx loom approve` hand-off, which does not tell the operator the approval was already acted on.
The wait has no bound and no signal that distinguishes "awaiting approval" from "approved, resume did not take".
The recovery exists but is unstated: `lyx loom approve` accepts a run still awaiting at Publish and writes a fresh `approved_at` (`internal/loomcli/approve.go`), so re-approving yields a new identity and triggers another resume.

**Fix:** State in the Decision that the marker is written only after `Spawn` returns success, and a `Spawn` error stays the existing hard error, retried on resume.
State that when the marker already holds the current approval identity and the child is still `awaiting`, the Stuck reason says the approval at `approved_at` was acted on and names the recovery: re-run `lyx loom approve` in the task worktree, which records a new identity and triggers another resume.
Add the matching acceptance criteria.

### [MEDIUM] Archive-tag abort point leaves a half-torn pair, and its reach beyond batten is not fully stated

**Location:** `_lyx/discussion/decision-record.md:76` (Decision "Every weft-branch deletion archives the weft tip first").

**Issue:** Extending the archive to manual `lyx fabric remove`, `RemovePairBranch` and prune is decided and in scope: the Decision lists them, and the support log's rejected alternative "Batten's teardown commits the child's records" rejects a batten-only route for exactly that reason.
The unreachable-origin case is covered in general terms ("a failed tag push aborts the deletion … surfaces as an error").
What the record leaves open is where the abort lands and what it leaves behind.
`Topology.Remove` (`internal/fabricengine/remove.go`) removes the portal, the launchers and the warp worktree before `removeWeftWorktree`, which removes the weft worktree and only then deletes the weft branch (`internal/fabricengine/weftwiring.go`).
An abort "before any branch is removed" therefore leaves both branches but neither worktree.
Remove also tolerates a weft-teardown error today whenever the weft worktree is already gone, so "surfaces as an error" reverses an existing contract without saying so.
Batten recovers, since its resumed teardown's already-gone arm calls `RemovePairBranch` (`internal/battencli/wire.go`), which archives too.
A manual `lyx fabric remove` re-run fails at `worktree … not found`, and the record names no recovery.
The tag push also runs regardless of `remote`, so a `lyx fabric remove` without `--remote`, or prune, now needs a reachable origin where it succeeded offline before; the record does not say whether `--force` bypasses the archive, or record this as a risk.

**Fix:** State that `Remove` creates and pushes the archive tag after its refusals and before its first mutation, so a failed push leaves the pair whole and a plain re-run works.
State whether `force` bypasses the archive step.
Add an Open risk: pair removal and prune now fail while the weft origin exists but cannot be reached.

### [MEDIUM] Done-wait bounces are counted against a budget the running watch may already have spent

**Location:** `_lyx/discussion/decision-record.md:54` (Decision "A `done` child is finished only after its driver has ended").

**Issue:** The done-wait returns Stuck "(not exempt)".
`episodeStuckCount` counts every Stuck by `Run-Shed` since its last Done (`internal/shedengine/run.go`), and `Run-Shed` has not returned Done during the child run, so done-wait bounces share one episode with the running watch.
The grace window at the default is 900 s / 30 s = 30 bounces.
A child that ran close to 12 hours leaves fewer than 30, and the done-wait then blocks batten with `ReasonBounceBudgetExhausted` on a child that is already `done`.
The record's rationale for the budget ("running time keeps its 12-hour budget") does not account for this, and no acceptance criterion covers it.
The done-wait already has its own bound, the grace window, so counting it against the running budget adds nothing.

**Fix:** Mark the done-wait Stuck budget-exempt as well, bounded by `driver_exit_grace_s`, and change "`Run-Shed` sets the flag only for an `awaiting` child" to cover both arms.
Add an acceptance criterion: a `done` child's grace wait neither consumes nor is blocked by the running budget.

### [MEDIUM] Grace-elapsed path: batten state and operator recovery unstated, and the surfaced remedy discards the report

**Location:** `_lyx/discussion/decision-record.md:114` (Open risk "Driver never exits after `done`"), with `:58`.

**Issue:** The risk says teardown ends "halting batten with a failed teardown".
The halt is concrete: `Worktree-Teardown` has no `on_stuck` (`contracts/recipes/batten-recipe.yaml`), so a `Remove` error is a Stuck that blocks the batten run with reason "worktree removal failed (session shutdown already succeeded): weft worktree has uncommitted changes; run \"lyx fabric sync\" or use --force" (`internal/battenshed/teardown.go`, `internal/fabricengine/remove.go`).
Remove has also already torn down the portal and launchers before its dirtiness refusal.
The record never states the recovery.
The one remedy the operator sees, `--force`, deletes exactly the drive report this task exists to keep.
The intended recovery is `lyx loom commit-records` in the task worktree, then resuming the batten run.

**Fix:** In the Decision at `:58`, or in the risk, state the resulting state (batten blocked at `Worktree-Teardown`, portal and launchers already removed, pair otherwise intact) and the recovery: run `lyx loom commit-records` in the task worktree, then resume the batten run.
Require the teardown's Stuck reason for a dirty weft on this path to name that recovery instead of `--force`.

### [LOW] Records pathspec ambiguous about friction archive siblings

**Location:** `_lyx/discussion/decision-record.md:69` (Decision "The per-transition status commit stages the records"), with `:62`.

**Issue:** `LoomFrictionArchivePrefix` is `LoomFrictionDir + "-"` (`internal/loomengine/config.go`), and `frictionengine.Reflect` renames the friction directory to a timestamped sibling after a successful reflection.
The Decision moves the archive prefix onto `_lyx/loom/`, so archives become `_lyx/loom/friction-<ts>/` beside, not inside, `_lyx/loom/friction/`.
"The `_lyx/loom/` friction tree" does not say whether the pathspec (and `commit-records`, which reuses it) covers those siblings.
If it covers only `friction/`, a reflected run leaves untracked files in the weft, and the non-force `Remove` refuses the dirty weft at teardown.

**Fix:** State that the records pathspec covers the friction directory and its archive siblings, and add it to the `statusCommitPathspec` acceptance criterion.

### [LOW] "Every weft-branch deletion" reaches Add's rollback

**Location:** `_lyx/discussion/decision-record.md:77`.

**Issue:** The Decision's title and first sentence say every path that deletes a pair's weft branch archives first, and a failed push aborts.
`Topology.Add`'s rollback also deletes a weft branch, one it created moments earlier (`internal/fabricengine/add.go`, `removeWeftWorktree` in the rollback).
That branch carries no run records, and an abort there would turn a failed add into a stranded half-created pair.
The acceptance criteria name only `Remove`, `RemovePairBranch` and prune, so the intent is narrower than the wording.

**Fix:** Scope the Decision to deleting the weft branch of an existing pair at teardown or cleanup, and exclude Add's rollback of a branch it just created by name.

### [LOW] In-flight split-run consequence unstated; duplicated assumption

**Location:** `_lyx/discussion/decision-record.md:116` (Open risk "In-flight runs at deploy"), with `:106`.

**Issue:** After the move, every friction consumer reads only `_lyx/loom/friction/`, including reflection (`internal/loomcli/run.go`, `frictionengine.Reflect` over `c.frictionDir`) and the hub loop's operator-gated filing, which reads the notes the child committed.
Notes an old-binary run left under `.lyx/loom/friction/` are therefore never reflected, never committed and never archived, and teardown deletes them with the worktree.
The risk names the split but not this silent loss, so the record neither accepts it nor mitigates it.
The auto-mode assumption at `:106` restates Scope Out's "Migrating notes or reports already written under the old `.lyx` paths" without adding anything; it is the wrong place for a scope decision already made.

**Fix:** Extend the Open risk: old-path notes of a run in flight at deploy are not read by reflection or the hub loop and are lost at teardown, accepted because deploy is operator-timed and can wait for in-flight runs to finish.
Remove the assumption at `:106`; Scope Out already records the decision.

## Focus directive verdicts

- Focus 1 (resume-once plus exempt budget): a defect, F1.
- Focus 2 (archive-tag reach and unreachable origin): extending to manual remove, `RemovePairBranch` and prune is decided and in scope, and the unreachable-origin failure is stated in general; the abort point, the manual recovery and the offline/`force` behaviour are open, F2.
- Focus 3 (grace-elapsed path and budget): both halves are defects, F3 and F4.
- Focus 4 (relocation pass): the auto-mode assumptions on the `start.go` verification and the driver-strand liveness key carry plan requirements.
  They already sit in `decision-record.md`, which Plan-Write reads, and each is correctly framed as an assumption to verify, not a settled decision, so neither needs lifting or moving.
  The remaining assumptions each record an auto-pick, which is what that section is for.
  Open risks and Notes for the plan writer carry no deliberation that belongs in `support-log.md`.
  The one exclusion candidate is the assumption at `:106`, folded into F7.
- Focus 5 (split friction notes): the consequence is unrecorded, F7.
