---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: handler
    location: internal/battencli/cleanup.go:135
    summary: The board entry asks that a finished run's task branch be removed with its run directory, and the diff removes only records and scratch, so a later start under the slug can still be refused by the leftover task branch.
    fix: Make the done-run clear also remove the run's task branch when the run is done and its landing is counted, locally and on origin with the archive tag the existing task-branch discard path writes (see internal/fabricengine/taskbranchdiscard.go and the discard seam in internal/battencli/cancel.go), reached from battencli.ClearDoneRun through fabric and under the remote-refs lock per PATTERN-gitrepo-client-boundary. A branch failure must leave the run's records in place so the next tick retries. If the operator means to keep the branch, the parent must record that narrowing with lym loom decision add, and then doneSlugRefusal in internal/battencli/arm.go and the commit message must say the branch stays. Add a test row that pins the branch removal.
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: internal/beamcli/doc.go:59
    summary: Docs still say a done run other than a motif run stays and refuses a later start naming lym batten cleanup, which the dobby's done-run clear makes stale.
    fix: In internal/beamcli/doc.go line 59 and docs/overview.md line 816, say that the parent's dobby normally removes a done run once its landing is counted and its notice decided, and that the refusal naming lym batten cleanup --apply applies only until it has. In docs/overview.md line 1617, add the dobby's done-run clear to the description of lym batten cleanup.
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/dobbyengine/tidy.go:117
    summary: Nothing pins that the tick calls clearDoneRuns after the landing count, or that countLandings sets d.landed, so removing either leaves every test green while the uncounted-landing gate stops working.
    fix: In internal/dobbyengine/donerun_test.go add one test that drives a tick or countLandings then clearDoneRuns over a fake Deps.Landings, with a done run that landed and is not yet counted because a tidy agent is in flight, and asserts the clear seam is not called, then called once the tidy settles. Reuse the existing daemon fixture.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/dobbyengine/donerun.go:58
    summary: When the state update that drops a cleared slug from CountedLandings fails, only a warning is logged and the stale entry stays, so a later run under the same slug has its landing treated as already counted.
    fix: In clearDoneRuns, drop CountedLandings entries in the same UpdateState call for every slug that is neither in d.landed nor listed as a run, on each tick that runs the clear, so a missed deletion is repaired at the next tick. Keep the warning for the update failure.
  - id: F5
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/battencli/cleanup.go:197
    summary: The commit message branch for a done run with no child-recipe record has no assertion, so a regression to a double space would pass.
    fix: In the rows of TestClearDoneRun_RemovesADoneRunTheRecipeFilterAdmits in internal/battencli/cleanup_test.go, assert with the stubs commits record that the commit message reads batten, then remove the done run of the slug, for the no-recipe row, and remove the done darn run of the slug for the darn row.
---

### [BLOCKING] Task branch is not removed

**Location:** internal/battencli/cleanup.go:135, internal/dobbyengine/donerun.go:25
**Issue:** The entry's `## Goal` says the fix is that teardown removes the finished run's directory "(and its task branch), not only that merge-parent picks a fresh slug".
The diff removes records and scratch through `ClearDoneRun` and nothing else; no commit in the range touches a branch.
The repository's own text says that is not enough to free the slug: `doneSlugRefusal` in `internal/battencli/arm.go:852` says a torn-down pair keeps its task branch locally and on the remote, so deleting only the run directory leads into the create row's leftover-branch refusal on the re-run, and `freeMergeTaskSlug` in `internal/beamcli/mergeparent.go:110` says a leftover task branch refuses a start under its slug.
The commit message claims "its slug is free again"; with the branch left, the diff's own evidence says it is not.
The half the diff makes (records) is sound and covered; the other half the entry names is missing.

### [MEDIUM] Docs describe the old done-run refusal

**Location:** internal/beamcli/doc.go:59, docs/overview.md:816, docs/overview.md:1617
**Issue:** `internal/beamcli/doc.go:59` says "Any other done run stays, and the first pass refuses it naming `lym batten cleanup --apply`".
`docs/overview.md:816` says "any other done run is still refused, naming `lym batten cleanup --apply`".
`docs/overview.md:1617` describes `lym batten cleanup` without the dobby's clear.
All three are read against the diff, which now removes every done run in the parent's dobby; the diff updated the dobby doc, the `cleanup.go` help and the `cli.go` help but left these.

### [LOW] Tick placement and the d.landed link are untested

**Location:** internal/dobbyengine/tidy.go:117, internal/dobbyengine/loop.go:697
**Issue:** `TestClearDoneRuns` calls `clearDoneRuns` directly and sets `d.landed` by hand.
Deleting `d.landed = landed` in `countLandings`, or the `d.clearDoneRuns(now)` call in `tick`, fails no test.
The first would let a done run be cleared before its landing is counted, losing its board-tidy count; the `doc.go` claim that the step is skipped while an anchor change stands is unpinned too.
Per test economy this is a behavior the diff adds with no coverage.

### [LOW] A failed CountedLandings update leaves a stale entry

**Location:** internal/dobbyengine/donerun.go:58
**Issue:** After a successful clear, `UpdateState` deletes the slug from `CountedLandings`; if it fails, a warning is logged and nothing retries.
A later run under the same slug that lands is then read as already counted, so its board-tidy count is lost without a trace.
Only the tidy count is at risk, hence LOW.

### [LOW] No assertion on the commit message of a no-recipe run

**Location:** internal/battencli/cleanup.go:197
**Issue:** The new `run` variable builds `"run"` for an empty recipe and `recipe + " run"` otherwise.
The new rows assert commit and spawn counts and the restored child recipe, not the message, so a malformed message passes.

## Rejected

- Lens code-correctness F1, a done run landed on another machine is cleared by this machine's dobby: the dobby loop ends when another machine owns its beam (`loop` doc, `probeOwner`), and `DobbyLandings` deliberately never counts another machine's landing, so clearing it loses no count this dobby would have made.
- Lens code-affected-set F2, the help text names fewer preconditions than the doc: the help is a user-facing summary, and the doc holds the exact conditions.
- Lens code-tests findings 3, 4, 5 and 6 (nil seam, unreadable state record, failed count update, episode logging): each is a short guard or a log call whose absence from a test is not a behavior a caller relies on; the update failure is carried by F4.
- Lens code-tests findings 8 and 9 (no test of `ClearDoneRun` or of the seam wiring): `ClearMotifRun` and its wiring follow the same untested thin shape, and the engine behavior is pinned through `clearDoneRun`.
- Focus question on gating and staleness of `d.landed`: a failed `Landings()` ends the tick before the clear, a nil `Landings` leaves nothing to count, and a tidy agent in flight leaves the slug uncounted so the clear waits; not a defect.
- Focus question on `ClearDoneRun` for a non-prime dobby: `ResolveBattenPrime` returns the prime for the prime location, the seam is bound to it, and `hubDobbyDeps` is the only production `Deps` literal; not a defect.
- Redundancy report: all three new tests are listed unique, so no test-economy finding.
