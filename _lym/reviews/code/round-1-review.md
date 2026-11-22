---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/battencli/arm.go:793
    summary: A fresh run never sends Worktree-Create through the commit seam, so Seed-Child meets no marker and still waits for a synchronous records push.
    fix: "In internal/battencli/arm.go, once the fresh-start seeding at line 793 has written the Worktree-Create running status and released the status lock, call the run's commit-status seam (battenCommitStatusSeam for the run) with (Worktree-Create, running) before the first producer runs, and return its error. A resumed run, whose status was already found, makes no such call. Then the fresh chain pushes once, at Worktree-Create, and Seed-Child through Run-Shed take startCore. Add a test that drives the fresh-start path through that call with fake Commit and Push closures and asserts the Worktree-Create commit pushes and a following (Seed-Child, running) commits without a push, instead of feeding the seam a synthetic Worktree-Create call. Correct _lym/landing/summary.md's claim that the chain pushes once at Worktree-Create if the fix takes another route."
    origin: assist:assist
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/battencli/commitstatus.go:264
    summary: A start-row push that failed with a non-owed error, or was cut off by a crash, is not retried on resume, so the rest of the start runs pushless with origin lacking the run's records until Land.
    fix: "In internal/battencli/commitstatus.go, give commitStatusMarker a durable push-pending field (JSON key push_pending, read tolerantly). The core's after-commit callback writes the marker with push_pending true; once core returns nil for a call that committed, rewrite the marker for the same pair with push_pending false. startCore's marker writes carry push_pending false. Take startCore at line 264 only when the marker's push_pending is false, so a resumed start after a failed or interrupted push goes through core and pushes. Add a TestNewCommitStatusSeam row whose Push fails with a non-owed error on the first start-chain push and succeeds after, and assert the next start row pushes."
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/battencli/commitstatus_test.go:53
    summary: TestNewCommitStatusSeam leaves four behaviours of the new start branch uncovered, so removing any of them passes every row.
    fix: "Add rows to TestNewCommitStatusSeam in internal/battencli/commitstatus_test.go, each with literal expected counts: (1) startCore rewrites the marker: {Worktree-Create running}, {Seed-Child running}, {Seed-Child running} expects 2 commits and 1 push; (2) Worktree-Create is never a start row: {Seed-Child running}, {Worktree-Create running} expects 2 commits and 2 pushes; (3) the marker producer must precede Run-Shed: {Run-Shed paused}, {Run-Shed running} expects 2 commits and 2 pushes; (4) the start core takes no gate admission: give the table's deps an Admit closure counting admissions, add a wantAdmits column, and assert StartRowsCommitWithoutPush takes 3 admissions, one per pushing transition."
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: decision
    location: internal/battencli/commitstatus.go:264
    summary: The diff narrows the decision's start-row exemption to an uninterrupted start, so a start row resumed after a pause or halt still pushes, and no Added after Discussion entry records that narrowing.
    fix: "Leave the code in place and defer this finding to the run's parent: recording or refusing the narrowing with lym loom decision add is the parent's and lies outside the fixer's write surface. If the parent refuses it, a resumed start row from Seed-Child to Run-Shed commits through startCore as well, and the ResumedStartRowPushes row and the uninterrupted wording in PATTERN.md and pattern/PATTERN-run-fsm.md change with it."
    origin: lens:code-scope
  - id: F5
    severity: LOW
    class: consistency
    location: PATTERN.md:28
    summary: The PATTERN-pair-remote entry and its background's opening still say both sides go before the run acts, with no start-row exception.
    fix: "In PATTERN.md's PATTERN-pair-remote entry and in the opening of pattern/PATTERN-pair-remote.md, append to the clause ending both before the run acts the exception bar a batten run's uninterrupted start from Seed-Child to Run-Shed, matching the PATTERN-run-fsm wording."
    origin: lens:code-affected-set
  - id: F6
    severity: LOW
    class: consistency
    location: internal/statuscommit/statuscommit.go:32
    summary: statuscommit's Deps.Push contract says a seam that pushes nothing leaves its transitions unpublished, and batten's startCore passes a no-op Push to build exactly that seam.
    fix: "In internal/statuscommit/statuscommit.go, reword the Deps.Push doc and the package doc's push sentence to state that Push is required, and that a caller deferring a transition's publication passes a Push that does nothing, its own next push carrying the commit; keep the nil-Push panic. Correct _lym/landing/summary.md's statement that internal/statuscommit is untouched."
    origin: lens:code-affected-set
  - id: F7
    severity: LOW
    class: consistency
    location: internal/battencli/commitstatus.go:209
    summary: The seam doc says a start transition commits and pushes nothing, though startCore commits it.
    fix: "In internal/battencli/commitstatus.go line 209, replace A start transition commits and pushes nothing with A start transition commits without a push."
    origin: assist:assist
---

### [BLOCKING] A fresh run still pushes at Seed-Child

**Location:** internal/battencli/arm.go:793

**Issue:** The start branch at `internal/battencli/commitstatus.go:264` needs a marker holding a running row before Run-Shed.
A fresh run never produces one before Seed-Child.
`seedAbsentStatus` (`internal/battencli/arm.go:806-823`) seeds the status as Worktree-Create `running` directly, with no seam call.
`stepLocked` then reads `started` as true (`internal/shedengine/run.go:240`), so the start write at `run.go:319-323` does not fire.
Worktree-Create has no `entry_gate` in `contracts/recipes/batten-recipe.yaml`, so nothing persists it either.
The first `CommitStatus` call is therefore (Seed-Child, running) after Worktree-Create's verdict, on an absent marker, and it takes `core` and pushes.
That leaves one of the five start-row pushes the decision record removes, Seed-Child's, and the board entry's own measurement lists Seed-Child among the rows paying it.
The test row `StartRowsCommitWithoutPush` feeds the seam a Worktree-Create call production never makes, so it passes over the gap.
The change description's "pushes once, at Worktree-Create" is false for the same reason.

### [MEDIUM] A failed start push is not retried on resume

**Location:** internal/battencli/commitstatus.go:264

**Issue:** The marker is written after the commit and before the push (`statuscommit.go:114-122`).
A core push that fails with an error that is not owed, or a process killed between commit and push, leaves the status and the marker at a running start row:
`persistMarked` writes the status before calling `CommitStatus` (`internal/shedengine/run.go:889-939`), so shed records no halt state.
On resume the next start row sees a running pre-Run-Shed marker and takes `startCore`, so no push is tried until Land.
At the review base each later transition retried the push and halted again, per the file header's "any other push failure … halts the run at the transition".
The doc's "a resumed or halted start row … pushes" does not hold for this resume.
With F1 fixed, the same path opens at the Worktree-Create push.

### [MEDIUM] Start-branch behaviours without a covering row

**Location:** internal/battencli/commitstatus_test.go:53

**Issue:** Four behaviours the diff adds pass every row when removed.
Passing `writeMarker` to `startCore` (`commitstatus.go:242`): no row repeats a start-row pair after a pushless commit.
The Worktree-Create exclusion in `isStartRow` (`:246`): every row reaches Worktree-Create on an empty marker, so the missing marker pushes, not the exclusion.
The `slices.Contains(preRunRows, marker.Producer)` conjunct (`:264`): no row enters a start row from Run-Shed or a later row.
The start core's missing `Admit` (`:237-242`): the table's deps leave `Admit` nil.

### [LOW] Resumed start rows push, an unrecorded narrowing

**Location:** internal/battencli/commitstatus.go:264

**Issue:** The decision record says the start rows from Seed-Child to Run-Shed act without a records push and carves out no resumed row.
The diff pushes a start row resumed from a pause or a halt, pins it in `ResumedStartRowPushes`, and writes "uninterrupted" into the PATTERN text.
The change description gives the reason as the writer's own design call (`_lym/landing/summary.md:34-36`), and no `Added after Discussion` entry records it.

### [LOW] PATTERN-pair-remote headline left without the exception

**Location:** PATTERN.md:28

**Issue:** The diff adds the start-row exception to `PATTERN-run-fsm`'s entry and opening and to the body of `pattern/PATTERN-pair-remote.md` (line 20).
`PATTERN-pair-remote`'s entry and its background's opening line still state "both before the run acts" without it.

### [LOW] statuscommit's Push contract contradicted

**Location:** internal/statuscommit/statuscommit.go:32

**Issue:** `Deps.Push` says New panics on a nil Push "since a seam that pushes nothing leaves its transitions unpublished".
`startCore` passes `func() error { return nil }` to pass that guard, building such a seam, and neither that doc nor the package doc's "on origin before the run's next producer starts" was changed.

### [LOW] Seam doc says the start transition commits nothing

**Location:** internal/battencli/commitstatus.go:209

**Issue:** "A start transition commits and pushes nothing" reads as no commit, while `startCore` commits through `deps.Commit` and the following sentences rely on that commit.

## Rejected

- Mutation survivor at `commitstatus.go:225`: the negated condition only changes whether a warning is logged; the marker write still runs, so no test can reasonably kill it.
- Seed-Child's own seed push (`internal/battenshed/seamchild.go:123`): pre-existing, spawned detached (`internal/battencli/wire.go:1666`), so the row never waits on it; the decision targets the transition push.
- The start core skipping the network gate: `pauserecord.Network` gates network calls, `startCore` makes none, and each row's own network calls still pass the gate.
- Takeover while the start commits are unpushed: the diff adds no takeover code and keeps the existing `PushRun` → `HandOver` path, held by `TestPushRun_HandsOverOnATakeover`; the inner run's own pair pushes also meet a takeover before Land. No regression found.
- The owed-push retry missing from `startCore`: `core` retries an owed push until it lands before returning, so no owed push can be pending when `startCore` runs, and `startCore` pushes nothing to owe.
- Existing rows' push assertion: changed from `wantCommits` to `wantPushes` with equal values on every existing row, so it is as strict as at the review base.
