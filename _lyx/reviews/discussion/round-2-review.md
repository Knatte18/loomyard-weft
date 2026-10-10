---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:196
    summary: D11 resolves the prime from the hook's cwd, but the orch session's cwd drifts into other worktrees, so the verb refuses there and the draft-case delivery D11 exists for is lost
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:200
    summary: The hook's pointer names the role file before the compaction reload re-renders it, and under a draft the re-render waits on readiness while the watcher then skips its pointer, so a changed role never reaches the session that cycle
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:208
    summary: D11 does not say which lyx binary the hook command runs, so a session spawned by a dev binary can run a PATH lyx that lacks the new verb
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:89
    summary: D4 adds an fswatch wake source on the resize signal with no stated bound on who may write it, what a wake costs, and that a wake finding no file runs nothing
---

### [MEDIUM] The resume-context verb is resolved from a cwd that drifts

**Location:** `_lyx/discussion/decision-record.md:196`, and the Auto-mode assumption at `:250`

**Issue:**
D11 runs `lyx orch resume-context` "from the orch's own cwd", and the assumption says the hook "runs in the session's cwd".
These are not the same place.
The prime orch's own events file (`loomyard/.lyx/shuttle/b1b9d1b7a48d9ef1c8dc6eb39d0623b6/events.jsonl`) records hook payloads whose `cwd` is `/home/knatte/Code/loomyard-LYXHUB/gate-test-scope` (lines 944 and 947) and `.../loomyard/.scratch/haikubench` (line 527).
So the orch session does `cd` into task pairs, and a hook command then runs there.
The verb resolves through `lyxcwd`, finds a task pair with no orch state, and prints its "outside the prime" JSON error.
The typed fallback still delivers the pointer, so nothing is lost outright.
But under a draft, the fallback is the late path that D11 and D7's Bound ("the resume pointer does not wait") exist to avoid, and the record never states this case.
The recording command is not affected, because its events path is quoted in absolutely.

**Fix:**
Make the hook command independent of the session's cwd.
`orchcli` sets the spec field, so it can tell the verb the prime's path in the command itself.
Examples are a flag the verb resolves through `cliwire`, or a `cd` to the prime that `internal/shell` builds.
Then change the Auto-mode assumption to say the hook's cwd does not matter.
Add an acceptance criterion: with the session's cwd in another worktree, the verb still renders the prime's pointer.

### [MEDIUM] The hook points the session at a role file the reload has not yet re-rendered

**Location:** `_lyx/discussion/decision-record.md:200` (D11's pointer choice) and `:210` (the skip)

**Issue:**
A compaction reload renders `role.md` only at the start of the reload.
`reloadAfterCompaction` does it at `internal/orchengine/watcher.go:961`, and `startAutoReload` at `:738`.
Both run only after the gate reads ready on that tick.
D7 makes a draft hold that gate.
In the draft case, then, the order of events is this:
1. The hook renders the pointer in `compacting` or `idle`.
2. The hook writes the mark.
3. The session reads `role.md` during the operator's turn, with `role.md` still as the previous cycle rendered it.
4. The watcher re-renders `role.md` only after the draft clears.
5. The watcher renders the same `PendingResume` text, matches the mark, and skips the pointer.

If the role stencil changed since the last render, as a plugin deploy does (the change this reload exists to pick up), the session runs that cycle on the stale role.
No later step points it at the new one.
D11 says the verb renders only the pointer, and the Constraints line on `PATTERN-stencil-ownership` says the same.
The record states neither the ordering nor its cost.

**Fix:**
In `compacting` and `idle`, the verb renders the role file before it renders and prints the pointer, through the same `RenderRoleFile` the watcher uses.
The file the hook names is then current when the session reads it.
In the alternative, the record states that the hook's pointer may name the previous render and bounds that cost.
Add an acceptance criterion for whichever is chosen.

### [LOW] The hook's `lyx` binary is unstated

**Location:** `_lyx/discussion/decision-record.md:208`

**Issue:**
The spec field carries a command that runs `lyx`, but D11 never says how that `lyx` is resolved.
A bare `lyx` resolves on `PATH`, which reaches the production binary.
Under `./deploy-dev`, the orch runs from `.dev-bin`, and the production binary may predate `resume-context`.
The verb then fails on every compaction.
The fallback hides the failure, so the hook route is never exercised in the very tests meant to prove it.
`reedengine/panebin.go` already exports the spawning binary as `LYX_BIN` to every pane, which the session and its hooks inherit.
`PATTERN-pane-binary-resolution` names this resolution for panes, windows and key bindings, though not for provider hooks.

**Fix:**
State that the hook command runs the spawning binary, either through `orchcli` rendering its own absolute path into the command or through `$LYX_BIN`.

### [LOW] D4's event wake has no stated bound

**Location:** `_lyx/discussion/decision-record.md:89`–`:94`

**Issue:**
D4 replaces the stat poll with an `fswatch` wake on `reed-resize.signal`, which is a new wake source into the daemon.
Any process can touch that file in the worktree's `.lyx`.
D5 bounds its own new wake explicitly.
D4 states none of the following:
- who may write the file;
- that each event costs one stat and one remove and spawns no process;
- that the event from the loop's own remove finds no file and runs nothing;
- that a burst cannot apply faster than the debounce and the retry cap allow.

"Handled as a stat tick is today" implies the third point but does not state it.
There is no acceptance criterion for a wake that finds no file, as there is for D5's discover signal at `:266`.

**Fix:**
Add a `Bound:` paragraph to D4 that covers those four points.
Add a matching clause to the resize-loop acceptance criterion at `:264`.

## Focus verdicts

These verdicts are for directive items that produced no finding.

- **D11's skip and failure path:** not a defect.
  The skip is keyed on a mark written only after a successful render, and only when its text equals `PendingResume`.
  A failed verb writes no mark and exits non-zero, through `clihelp.RunRoot`'s JSON error.
  Its output is not added as context, and the typed pointer follows.
  The mark-then-print order leaves only the dropped-output case, which the Bound and an Open risk already state.
- **Resume versus reload over the whole orch state:** settled, apart from F2.
  `resuming` with `PendingResume`, `compacting`, and every other phase each map to one pointer.
  A tick that enters `resuming` before the verb reads state is covered by the verbatim `PendingResume`.
  In `compacting`, the hook and `reloadAfterCompaction` both call `RenderResumePrompt` with `State.LastHandoff`, so the texts match and the note is not lost.
  A compaction in `handoff-requested` reaches the note through the cycle that follows.
  The prime's events file shows that a typed `/compact` records no `UserPromptSubmit` (none between 04:07:30Z and 04:08:45Z around the 04:08:25Z boundary), so a compaction leaves no unmatched turn start for D7 to release.
- **D5's boot premise:** holds.
  `new-session` is issued at one site, `ensureServerAndSessionLocked` in `internal/reedengine/lifecycle.go`.
  `Up`, `EnsureSession`, `Resume` and revival all reach it.
  Every hub-run `reedengine.Geometry` is built by `hubgeom.ReedGeometry`, so one geometry builder carries the told path for orch start, loom, batten and revival alike.
- **D5's discover-signal bound:** stated (who, rate floor, no-file wake), and the criterion at `:266` mirrors it.
  D4's bound is missing; see F4.
- **D7 against the fold:** complete.
  The ready, held and `unknown` lists cover every name and cause the fold in `internal/shuttleengine/sessionstate.go` yields.
  An unmatched turn start releases through `turnStartHold`.
  `applySignal` has no case for an unhandled kind, so a `session-start` signal moves no state, and `unmatchedTurnStart` ignores it.
  The acceptance criteria at `:268`–`:269` mirror the lists.
