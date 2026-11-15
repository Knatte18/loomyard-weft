---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lym/landing/summary.md:34
    summary: The change description still describes the round-1 code, and several of its claims are false for the diff as it stands
    fix: "Rewrite the stale claims in _lym/landing/summary.md to match the code. Under What changes, the watcher bullet: the watcher calls pairteardown.EndSubjectProcesses (not an orchcli endSubjectProcesses) with endDobby unset, so the dobby is not ended and hands the beam over itself; the strand is stopped even when the end fails, and a failed end is not retried once the strand is gone; the subject pass spares the adopt worker, board syncs, detached pushes, coverage-map pass, Slack relay and every teardown child. Under Supporting changes, add that pairteardown's endOwnedProcesses now spares runproc.RoleOrch in cancel, skip and teardown, and that EndSubjectProcesses lives in pairteardown. Under Design decisions, delete or rewrite the bullet saying a pair teardown's runproc.End of a beam's slug now ends that beam's orch, since the teardown now spares the orch. Under Tests, replace 'a failed end stops nothing' with 'a failed end still stops the strand and returns the error', and add pairteardown's TestSubjectRuns."
  - id: F2
    severity: LOW
    class: design
    location: internal/orchengine/owner.go:68
    summary: A failed end with a successful strand stop records the watcher exit as strand gone, so status hides that another machine owns the beam
    fix: "In ownerTick, when Stop succeeded, set st.WatcherExit to 'owned by machine ' + owner and save it through w.save before returning, also when End failed; return the end's error with it so the failure is still logged. Keep returning done=false without touching WatcherExit only when Stop failed. Update the ownerTick doc comment and the matching paragraph of internal/orchengine/doc.go to say the exit reason is recorded whenever the strand was stopped, and change the FailingEndStillStopsTheStrandAndReturnsTheError row of TestWatcher_EndsTheProcessesAndStopsTheOrchOfABeamOwnedElsewhere to expect wantExit 'owned by machine there' in place of wantNoExit."
  - id: F3
    severity: LOW
    class: consistency
    location: internal/pairteardown/doc.go:166
    summary: The pairteardown package doc gives each exported entry point its own section, but says nothing of the new exported EndSubjectProcesses
    fix: "Add a section to internal/pairteardown/doc.go, such as '// # Ending a subject owned elsewhere', stating that EndSubjectProcesses ends every process this machine runs for a subject another machine owns: the registered runs whose adopt subject is the beam, or the root for runproc.HubOwner, through runproc.EndSubject; that the subject's own record is never one of its runs, a run whose subject cannot be resolved is left alone and logged, and the dobby ends last only when endDobby is set."
---

The review range is `git diff 055ef994e63287e8d296ca1b8d50f78cb87bdfe6..HEAD`, eight commits, read with their messages.
No commit carries a `Pattern-Departure:` trailer, and the directive declares no PATTERN change.
The diff's edit of `pattern/PATTERN-child-processes.md` adds `EndSubject` to the rule's list of end functions, which the new function needs to stay within that rule.

The focus questions, each settled:

1. **Cancel, skip and teardown on a beam's slug.**
   `endOwnedProcesses` (`internal/pairteardown/teardown.go:700`) now appends `runproc.RoleOrch` to the spared roles,
   and `cancel.go:166`, `skip.go:94` and `teardown.go:336` all go through it.
   The other `runproc.End` callers on a slug (`tendercli/deps.go:114` for orphan owners, `:342` for a run owned elsewhere) never hit a live beam's orch on its owner machine.
   `fabriccli`'s anchor move spares the orch as well.
   Not a defect.
2. **`endSubject`'s roles.**
   The run pass spares only `RoleTeardown`; a run's slug holds only that run's processes, its records push included, which the dobby's own `PauseOwnedElsewhere` ends anyway.
   The subject pass spares the dobby, teardown, adopt, sync, push, test-map and Slack relay, for the root and a beam alike, and `TestEndSubject_EndsTheRunsThenTheSubjectAndItsDobbyLast` pins both.
   Not a defect.
3. **The dobby's hand-over.**
   The watcher passes `endDobby=false` (`internal/orchcli/cli.go:308`).
   `handOver`'s `pauseRunsForOwner` and `ArchiveOwner` run inside the dobby process itself, whose git children sit below no run or subject record, so the end cannot cut them short.
   Not a defect.
4. **A persistently failing end.**
   `ownerTick` calls `Stop` whatever `End` returned, and the next tick exits on the gone strand, so the strand never stays up on a machine that no longer owns the beam.
   The remaining gap is the lost exit reason, F2.
5. **Run selection.**
   `pairteardown.EndSubjectProcesses` is exported, and `TestSubjectRuns` pins beam versus root, skipping the subject's own owner and leaving an unreadable subject alone.
   `runproc.HubOwner` maps to itself through `AdoptSubject`, so it never joins a beam's runs.
   Not a defect; F3 covers the missing package doc.

The acceptance criteria hold:
`TestStart_ResumeOverALiveHolder` covers ending the own holder and resuming the same id, and refusing an unregistered holder by pid with nothing launched fresh.
The no-transcript rows of `TestStart_ResumesTheRecordedSession` cover the fresh launch, its carry-over count and the warning.
The session id stays in `orchengine.State` under `.lym/orch/`.

### [MEDIUM] The change description describes the round-1 code

**Location:** `_lym/landing/summary.md:34`

**Issue:** Several claims are not borne out by the diff.
The watcher bullet says `orchcli` wires `End` to `endSubjectProcesses` and that `EndSubject` ends the dobby last;
the selection now lives in `pairteardown.EndSubjectProcesses`, and the watcher passes `endDobby=false`, so the dobby is never ended there.
The same bullet says a failed end is retried at the next tick, but after a stopped strand the watcher exits and the end is never retried.
The Design decisions bullet says a pair teardown's `runproc.End` of a beam's slug now ends that beam's orch, but `endOwnedProcesses` now spares `runproc.RoleOrch`.
The Tests section says the owner-check rows assert that a failed end stops nothing, but `FailingEndStillStopsTheStrandAndReturnsTheError` asserts the opposite.
The description also leaves out the pairteardown changes and `TestSubjectRuns`.
The round-1 fixer report deferred this file to the parent because `_lym` links to the `orch-resume-own-session-weft` worktree, outside the fixer's write surface.
The finding stands until the parent, or whoever writes that worktree's `_lym/landing/summary.md`, makes the rewrite.

### [LOW] A failed end hides the ownership exit reason

**Location:** `internal/orchengine/owner.go:68`

**Issue:** When `End` fails and `Stop` succeeds, `ownerTick` returns the error without saving `WatcherExit`.
On the next tick, `tick` (`internal/orchengine/watcher.go:276`) finds the strand gone and records `strand gone`.
`lym orch status` (`orchcli/status.go:67`) and the batten overview (`battencli/overvieworch.go:150`) then report a dead orch rather than one stopped because machine X owns the beam,
so the operator loses the one signal saying where the beam went.

### [LOW] The pairteardown package doc does not describe `EndSubjectProcesses`

**Location:** `internal/pairteardown/doc.go:166`

**Issue:** The package doc gives each exported entry point a section, such as `# Batten follow-up` for `ReadBattenRecord`, `OwnerPauseHeld` and `FollowUpFor`, and `# Origin seams` for `HubSeamOptions` and `OwnerReader`.
The new exported `EndSubjectProcesses` has none, and nothing in `doc.go` names it.
The only package-level description of it is in `internal/orchcli/doc.go`, which documents a caller rather than the package that owns the function.

## Rejected

- `TestRun_CarriedInstructions` tests a one-line getter, but `orchcli`'s tests reach the count through a fake starter, so no other test covers `Run.CarriedInstructions`; no redundancy finding.
- `TestSubjectRuns` drives the unexported `subjectRuns`; `EndSubjectProcesses` reads the live board and process table, and the repository's tests in this package already use the internal test package, so this is no finding.
- `EndRole(owner, RoleOrch, …)` in `endOwnHolder` ends every orch record under the owner, not only the holder's pid; one worktree runs one orch, so no other process can be hit.
