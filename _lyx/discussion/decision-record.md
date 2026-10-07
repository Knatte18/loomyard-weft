# Decision record — run-recovery

## Goal

The operator and the orch can see what a run waits on, and a run comes back up as it was after a crash or a halt.
Concretely: the Go-side waits Scope names show in `lyx loom status` and on screen with their elapsed time, a verify gate cannot hang without a timeout, reed never revives an ungated shuttle strand whose run is finished and never stacks a bare claude over a dormant orch, `lyx batten run` brings a halted child's driver back without resuming the child, and the orch resumes a child through a verb that only wakes its driver.

## Scope

In:

- A running verify gate in `lyx loom status`: site, elapsed time and command, at every site that runs `verifytree.Verify` (Webster gate, Webster-Burler round gate, Publish and Finalize merge-in verify, `webster verify`).
- Shuttle's other post-turn waits in status too: any gate entry it evaluates, and the background-shell wait (`background_shell_wait_min`) after a turn end.
- The same waits on screen: a pane user option on the waiting strand, rendered by reed's status line with elapsed time.
- A timeout on the verify command, with the verify shell's process group killed on expiry on Unix and the shell alone on Windows (#420).
- reed: `--unless-name` matches a dormant strand (#408); `reed resume` drops, rather than revives, an ungated shuttle run's strand whose outputs are all present (#409).
- batten: a restarted `lyx batten run` brings its child back up as it was, driver included, without resuming a halted child.
- `lyx loom resume`: the orch's resume verb; the orch role stencil names it instead of `lyx loom start`.

Out:

- How gates are tiered or which gate runs which tests (`test-tiers`).
- Why shuttle's wait misses a valid Stop, and how a turn end without outputs is held (`shuttle-turns`); this task only surfaces the background-shell wait, never changes its policy.
- Notice channel, notice wording, runs-overview window, batten detached start, reed status tabs, gate checklist pane (`orch-notice-channel`, `runs-overview`, `orch-owns-the-loop`, `reed-status-tabs`, `gate-checklist-pane`).
- A step-level watchdog goroutine (#420's third bullet): the status note and the on-screen mark answer "which gate is the step in" without one.

## Decisions

### D1 — One wait note per run in `lyx loom status`, with elapsed time

Decision: `lyx loom status`'s `waiting` field reports the current Go-side wait in the form `<producer>: <what> running <elapsed> (<detail>)`, e.g. `Webster-Burler: verify running 6m (go test -tags integration ./...)`.
The verify marker (`verifytree.Marker`) gains the command; its note renders elapsed time from an injectable clock instead of a `since HH:MM` stamp, and keeps the attempt clause.
Shuttle's post-turn waits get their own marker in the run's shuttle run directory: kind (`gate <name>` or `background shells`), start time and pid, written when the wait starts and removed when it ends; a dead pid reads as absent, the same rule `verifytree.ReadMarker` uses.
Loom's `Waiting` hook reads, in order: the verify marker, the shuttle wait marker of the worktree's live runs, the parent-review note; the first marker that is present with a live pid wins, and a marker with a dead pid is skipped, not the end of the search.
The producer name comes from the run's status (current producer).
Rationale: the verify marker already exists and reports correctly; the background-shell wait has no marker. One read path keeps status and the orch's view consistent.

### D2 — On screen: a pane user option on the waiting strand

Decision: reed gains one engine op that sets or clears a wait mark on a strand's pane as two pane user options (`@lyx_wait`, the label; `@lyx_wait_start`, epoch seconds).
Shuttle sets it on its agent strand when a gate entry's evaluation or the background-shell wait starts, and clears it when that wait ends, on every path out (pass, fail, error, cancel).
Shuttle also clears it on the strand when a run starts or attaches, so a mark left by a crashed step is gone at the next touch.
The landing producers' merge-in verify sets the mark on the run's driver strand through a told callback in `landingshed`'s deps, and clears it at the start of every verify and on every path out; loom wires it to the driver strand it already resolves, and a run with no driver strand (Go driver) gets the status note only.
A callback whose strand is gone or replaced is a logged no-op.
Any failure of the mark op, whether at a shuttle site or in the landing callback, is logged and changes no verdict, error return or gate outcome: a verify or a shuttle wait never fails or ends differently because a mark could not be set or cleared.
reed's default status line renders every marked pane of the session as `<pane title> ⏳<label> <elapsed>`, elapsed computed by tmux at each refresh; the segment is a status-line template token, so a custom template can place or drop it.
Rationale: shuttle owns the gates and the shell wait; pane options die with the pane, so a reborn strand never carries a stale mark.
Bound: the mark is display only, never read by any Go decision.
A stale or misdirected mark can show a wait that has ended, and its elapsed time keeps growing so it reads as stale; it never hides a real wait, since `lyx loom status` reads the markers, not the pane.
The next set or clear on the strand, or the pane's death, removes it.

### D3 — The verify command has a timeout and its shell's process group dies on expiry

Decision: on Unix, `verifyrun` starts the shell in its own process group and, on context cancel or expiry, kills the whole group, not only the shell.
On Windows it kills the shell only, as today; Windows has no process-group kill, so descendants may outlive the timeout there.
`verifyrun` cannot tell whether a descendant exists, so it logs on every Windows cancel or expiry kill that only the shell was killed.
`cmd.WaitDelay` stays as the limit on the pipe read on both platforms.
`verifytree.Verify` takes the timeout as a `time.Duration` parameter and derives its own context from the caller's with it, so an expiry stays distinguishable from a caller's cancel.
Production callers pass the one constant `verifytree.Timeout`, 60 minutes, declared in `verifytree`; a test passes a short value.
An expired timeout is a failed verify with a reason naming the timeout, never a returned error: the gate fails with findings naming the timeout, the log path and the log tail, writes no record, removes the marker.
A caller's own context cancel stays a returned error, as today.
Rationale: nothing bounds the gate today (#420). An expired timeout is a defect the writer can act on (a hanging test), so it rides the normal failed-gate path.
Bound: the timeout only shortens a verify that would otherwise run unbounded; it lets no failing tree pass.
The group kill reaches only processes in the verify shell's own group.

### D4 — `--unless-name` matches a dormant strand (#408)

Decision: `AddStrandUnless` skips the add when a strand named `unlessName` exists in this worktree's reed state, live or dormant; the flag's help and the vscode chain's comment say so.
Rationale: the only caller is the folder-open chain, whose intent is "never stack a bare claude over the orch". The dormant orch comes back through `lyx orch start` as today. Bound: the skip only ever suppresses an add; it launches, revives and removes nothing.

### D5 — reed resume revives only strands with work left (#409)

Decision: a reed strand may carry a list of done-when paths, set by its spawner at add time.
`Resume` drops from state, instead of relaunching, a non-live strand whose done-when list is non-empty and whose every path exists.
Shuttle sets the done-when list to its run's output files, the Completion Signal's own test of a finished run, on the strand of an ungated run only.
A gated run's strand carries no list, since its outputs existing does not mean it is finished: the gate runs after them, and a failing gate re-prompts the live agent.
Strands without a list (gated runs, driver, orch, operator strands) are relaunched as today.
Rationale: reed cannot import shuttle (shuttle imports reed), and for an ungated run the output files are what shuttle already treats as "finished" (`PATTERN-completion-signal`). Dropping, not skipping, keeps a finished strand from lingering for the next resume.
Bound: only non-live strands are dropped, only when every listed path exists; a live strand is never touched, and a strand whose run is still working (outputs missing) or is gated, whatever its gate's state, is relaunched as before, so no gate loses its re-prompt.
Reed cannot know that a gate passed, so a finished gated run's strand is relaunched too, when shuttle's own teardown did not run; the relaunch types no prompt, and the strand is retired at the next teardown like any other.

### D6 — `lyx batten run` brings its child back up as it was

Decision: batten's inner-run row, on a child it does not hold live, restores it once per batten process:

- A running child whose spawn was confirmed by an earlier, now-dead batten process is spawned again (the child's bootstrap, idempotent against a live driver, takes the crash-resume path); the spawn confirmation is scoped to the batten process that wrote it.
- A halted (blocked, paused, failed) or awaiting child whose reed state holds a dead driver strand (not live) gets its pair's strands revived through reed resume (D5 applies), which brings back the parked driver with its own session; the run itself stays halted, and the park marker stays, so a later resume wakes that driver.
  A driver strand reed marked retiring is live to reed, and reed resume leaves a live strand alone, so it is not revived; it is left to `lyx loom start`, which removes it and spawns a fresh driver (D7).
  A child with no driver strand in reed state, such as a Go-driven one, has nothing to revive: no revive, no warn, and batten never reads the seed's driver to tell the two apart.
- A revive that fails, or leaves the driver still not live, is warned once; batten does not retry it in that episode.
  A new batten process is a new episode and tries once more.
- A halted child's halt wait reason, the text batten logs and writes to the row's stuck-reason file, replaces its `lyx loom start` text: it names `lyx loom resume` in the task worktree, and also `lyx loom start` in the task worktree once a revive has failed or when the child has no driver strand in reed state.
  `lyx batten status` does not show this reason while the child is halted, and no status surface is added; the orch learns the verb from its role stencil and from D7's refusals.
  The awaiting child's hints keep their verbs: `lyx loom approve`/`lyx loom reject` for a child waiting on a decision or awaiting again, and `lyx loom start` as the fallback in the hint for a decision already acted on whose driver has not re-stepped, for the case that the driver has ended.
  None names `lyx loom resume`, since D7 refuses an awaiting run.
  The notice line and the notice channel are unchanged (`orch-notice-channel`).

Rationale: the child comes back "as it was, driver included", while the halt stays the child's own.
Bound: the revive runs at most once per halt episode per batten process, starts no shed step, removes no park marker and sends no resume line.
Reed resume relaunches every non-live, non-hidden strand of the pair's session, so a halted step's gated agent strand and any operator strand come back too, each at its own prompt with no input typed; D5 drops only an ungated run's finished strand.
The re-spawn runs only for a child with no live driver strand and no held child run lock, so it never stacks a second driver.

### D7 — `lyx loom resume`, the orch's resume verb

Decision: a new loom verb, run in the task worktree, that resumes a halted run only by waking its live, parked driver: it types the driver resume line and removes the park marker, the same branch `lyx loom start` takes for a parked driver, shared rather than copied.
It never spawns a driver, never creates or attaches a strand and never switches a client.
It reads only strand liveness, the park marker, the child's run lock and the run's state, never the seed's driver value (`PATTERN-driver-choice-single-site`).
A driver strand reed marked retiring counts as not live, as `lyx loom start` never adopts one, so it falls under the no-live-driver refusal below, with its own way forward.
It waits for the same bootstrap lock `lyx loom start` takes, reads the run's state, the driver strand's liveness, the park marker and the child's run lock only after holding it, and releases it on every path out, so the two never run at once.
The messages of the shared resume branch and of the reused refusal kinds name the verb that called them, so `lyx loom resume` never tells its caller to run `lyx loom start`.
Outcomes, decided by the run's state first:

- no status file: refused; the way forward is `lyx loom start` in the task worktree;
- done run: a no-op success saying the run is done;
- awaiting run: refused; the way forward is `lyx loom approve` or `lyx loom reject` in the task worktree, after which a batten run watching the child resumes it on the recorded decision, and `lyx batten run <slug>` from the prime when no batten run is watching it; the refusal reads no batten state, so it never names the hint for a decision already acted on;
- running run: a no-op success saying the run is already running when a driver strand is live or the child's run lock is held; otherwise refused with the no-live-driver way forward below;
- halted run (blocked, paused, failed) whose child's run lock is held, whatever its driver strand: refused, a driver is still finishing its post-run work; the way forward is `lyx loom resume` again in a few seconds (`lyx loom status` shows no lock holder, so the refusal names none);
- halted run, live driver not yet parked: refused, retry in a few seconds (start's existing refusal kind); the merge is not probed, as `lyx loom start` does not probe it for a working driver;
- halted run, parked live driver: resumed, the envelope names the stop-report path; over an unfinished merge it is refused instead with start's existing merge-in-progress kind, as `lyx loom start` refuses it;
- halted run, no live driver strand: over an unfinished merge refused with the same merge-in-progress kind; otherwise refused, and the way forward is told apart by reed's strand table, never by the seed's driver:
  - a dead driver strand (present, not live, not retiring): the refusal names both ways, with their conditions, since it reads no batten state: `lyx batten run <slug>` from the prime when no batten run is watching the child, which brings the driver back (D6), then `lyx loom resume` again; `lyx loom start` in the task worktree when a batten run is already watching (`lyx batten run` is refused while batten's own run lock is held) or once D6's revive has failed;
  - a retiring driver strand: `lyx loom start` in the task worktree, which removes the retiring strand and spawns a fresh driver; D6's revive does not reach it;
  - no driver strand in reed state (a Go-driven run, or an LLM-driven one whose strand is gone): `lyx loom start` in the task worktree, since D6's revive has nothing to bring back.

Every refusal gets its spec row and a test (`PATTERN-refusal-way-forward`) and names no denied command (`PATTERN-no-denied-recovery`).
The orch role stencil names `lyx loom resume` wherever it names `lyx loom start` for a halted child, after a circling decision and after a goto, and drops "message the driver by name to resume" as a separate step.
Rationale: `lyx loom start` wakes a parked live driver but spawns a fresh one when none is live, replacing a dead driver's own session; waking the parked driver is the one effect the orch needs, and a dead driver goes back through batten's revive (D6) instead.
Bound: the verb can only deliver the resume line a parked driver already waits for; it cannot start anything that is not there.

### D8 — Docs in the same commit

Decision: each touched package's `doc.go` records its new behaviour, including `battenshed`'s halted-wait sentence per D6; the orch role stencil changes per D7.
`docs/overview.md` gains `lyx loom resume` in its loom verb list, and its batten paragraph on a halted child, which says batten cannot restart the driver and that the reason is on `lyx batten status`, is rewritten per D6.
`PATTERN.md` changes in the same commit:

- `PATTERN-completion-signal` and its background file gain the done-when list as the one place reed learns a strand finished: filled only by `shuttleengine`, from an ungated run's `OutputFiles`, and read only by `reedengine`'s resume.
- One new entry for the wait mark: display only, set only through reed's one op by shuttle and the landing callback, never read by a Go decision.

Rationale: CLAUDE.md's docs-land-in-the-same-commit rule, and its rule that a cross-cutting invariant lands in PATTERN with the code.

## Constraints

- `PATTERN-told-geometry`: `verifytree`, `shuttleengine`, `reedengine` and `landingshed` get every path and the wait-mark callback told; none resolves a cwd.
- `PATTERN-lyxdirs-single-declarer` and `PATTERN-durable-vs-ephemeral-state`: the shuttle wait marker lives in the shuttle run directory under `.lyx`; no new `_lyx`/`.lyx` literal.
- `PATTERN-shuttle-provider-seam`: the wait mark and done-when list are provider-neutral and set in `shuttleengine`, never in `claudeengine`.
- `PATTERN-completion-signal`: D5's done-when list is an ungated run's output-file set; no other "finished" test is invented.
- `PATTERN-driver-choice-single-site`: `lyx loom resume` and batten's revive never read the seed's driver.
- `PATTERN-refusal-way-forward`, `PATTERN-no-denied-recovery`: every new refusal of D7 names a lyx verb, or a retry of `lyx loom resume` while a lock holder finishes, as its way forward, with a spec row and a test; D6's halt reason names a lyx verb and no denied command.
- `PATTERN-cli-cobra`, `PATTERN-cliwire-sole-wiring`: `lyx loom resume` is a cobra verb in loom's subtree, wired through cliwire, JSON errors.
- `PATTERN-sandbox-coverage`: the sandbox suite exercises `lyx loom resume` or excludes it with a reason.
- `PATTERN-spawn-observability`: the verify process-group kill (and Windows' degraded kill) and batten's revive are logged.
- `PATTERN-shell-mechanics-seam`: process-group setup and the group kill stay in `verifyrun`, in per-OS files using only the standard library's `syscall`, not in `internal/shell` and not through an `internal/proc` import.
- `PATTERN-orch-pane-single-writer`: nothing here types into the orch pane; batten's halt notice keeps going through `orchengine`.
- `PATTERN-test-speed`, `PATTERN-test-isolation`, `PATTERN-test-economy`: elapsed-time rendering uses an injectable clock, the verify timeout is a parameter a test shortens, with `verifytree.Timeout` as the production value; tmux-dependent checks sit in the `tmux` tier.
- `PATTERN-comment-line-breaks`: semantic line breaks in every new Go comment.

## Auto-mode assumptions

- Publish/Finalize's on-screen mark goes on the driver strand, since no agent strand waits there; a Go-driven run shows it in status only.
- The verify timeout is a constant, not a config key: no operator has asked to tune it, and one value serves every site.
- `lyx loom resume` lives in loom, run in the task worktree, like `lyx loom approve`; no batten-level resume verb, since batten polls the child and needs no wake-up.
- "Records the decision" in the board's direction is read as the resume's own trace (log line, stop-report path in the envelope, park marker removed); no new decision record type.
- A strand added before this change carries no done-when list and is relaunched as today.

## Open risks

- tmux's elapsed-time arithmetic in a status format (`%s` against a pane option inside `#{e|-:…}`) must be checked live on the tmux version in use; if it does not expand, the fallback is to show the start time (`since HH:MM`) instead of elapsed time.
- A crashed step leaves the wait mark on a surviving pane until the next set or clear on that strand (shuttle's start or attach for an agent strand, the next verify for the driver strand); `lyx loom status` stays the truth, since its markers check pid liveness.
- `shuttle-turns` edits `internal/shuttleengine/wait.go` around the background-shell wait at the same time; the two runs must be compared at PR-Gate.

## Acceptance criteria

- `lyx loom status` during a verify gate at each site (Webster gate, Webster-Burler round gate, Publish merge-in) shows producer, `verify running <elapsed>` and the command.
- `lyx loom status` during shuttle's background-shell wait and during a non-verify gate entry shows that wait; a marker with a dead pid shows nothing.
- The wait mark is set on the right strand's pane when a shuttle gate or shell wait starts and cleared on pass, fail, error and cancel; a run start or attach clears a stale one.
- The landing callback sets the mark on the driver strand and clears it at the start of every verify and on every path out; a callback whose strand is gone is a logged no-op; a failing mark op at that callback or at a shuttle site is logged and leaves the verify's or the wait's outcome unchanged.
- The default reed status line renders a marked pane with its title, label and elapsed time (the start time instead, if the tmux in use does not expand the elapsed arithmetic); an unmarked session renders as before.
- A verify command whose descendant holds the output pipe and outlives the timeout ends as a failed gate naming the timeout; on Unix no process of its group survives, and on Windows the shell is killed and a log line says only the shell was killed.
- A cancelled verify still returns an error and writes no record.
- `reed add --unless-name orch` with a dormant orch strand in state adds nothing; with no orch strand it adds.
- `reed resume` relaunches a non-live strand with no done-when list and one whose done-when paths are partly missing, and drops a non-live strand whose done-when paths all exist; a live strand is untouched.
- A shuttle-spawned strand of an ungated run carries its run's output files as its done-when list; the strand of a gated run carries none, and `reed resume` relaunches it even with all outputs present.
- `batten run` over a halted or awaiting child with a dead driver strand revives the pair's strands once, leaves the child's state and park marker as they were, and does not revive again in that episode; a retiring driver strand is not revived and gets no warn; a failed revive is warned and the halted child's wait reason also names `lyx loom start`; a halted child's wait reason names `lyx loom resume`, and names `lyx loom start` only after a failed revive or for a child with no driver strand in reed state; a halted child with no driver strand in reed state gets no revive and no warn, and its wait reason names `lyx loom resume` and `lyx loom start`; an awaiting child's hints keep `lyx loom approve`/`lyx loom reject`, and the hint for a decision already acted on keeps `lyx loom start` as its driver-ended fallback.
- `batten run` over a running child whose spawn confirmation came from a dead batten process re-spawns the child's bootstrap.
- `lyx loom resume`: halted with a parked live driver → resume line sent, marker removed; running with a live driver strand or a held child run lock → no-op success; halted with the driver not yet parked → refusal; halted, or running with no live driver strand and no held child run lock, with a dead driver strand in reed state → refusal naming both `lyx batten run <slug>` (no batten run watching) and `lyx loom start` (a batten run watching, or a failed revive), with a retiring driver strand → refusal naming `lyx loom start` only, and with no driver strand in reed state → refusal naming `lyx loom start`; halted with the child's run lock held, whatever its driver strand → a retry refusal; a parked live driver or no live driver over an unfinished merge → the merge-in-progress refusal; no status file → refusal naming `lyx loom start`; awaiting → refusal naming `lyx loom approve`, `lyx loom reject` and `lyx batten run <slug>`; done → no-op success; every refusal and failure message names `lyx loom resume`, never `lyx loom start`, as the retry; the verb waits for a held bootstrap lock and reads state only after holding it; no path spawns, adds a strand or switches a client.
- The orch role stencil names `lyx loom resume` for every halted-child resume and no longer names `lyx loom start` for it.
- `go test ./...` passes, including the stencil claims and refusal-spec tests.

## Notes for the plan writer

- Status hook: `internal/loomcli/reviewstatus.go` (`verifyWaiting`, `reviewWaitingFor`); marker: `internal/verifytree/marker.go`.
- Verify sites: `internal/loomshed/verifygate.go`, `internal/websterengine/verifygate.go`, `internal/landingshed/verifygate.go`, `internal/webstercli/verify.go`.
- Shuttle's gate and shell wait: `internal/shuttleengine/wait.go` (`handleGatedBoundary`, `evaluateGate`, `expiredTurnEnd`, `shellWaitBound`).
- reed: `AddStrandUnless` and `liveStrandNamed` in `internal/reedengine/strand.go`; `planResumeLaunches` in `internal/reedengine/lifecycle.go`; status line in `internal/reedengine/statusline.go` and `status-line.md`; the `allow-set-title off` pin in `spawn.go` is why `pane_title` holds the strand name.
- batten: `innerRunProducer.Call` and `callHalted` in `internal/battenshed/innerrun.go`, `SpawnConfirmedFile`.
- loom resume: `resumeParkedDriver` in `internal/loomcli/driverresume.go` and the parked branch of `runDriverSpawnAndWait` in `internal/loomcli/start.go`.
- Orch stencil: `contracts/stencils/orch/orch-template-role.md`, run loop item 2 and Escalations from a child.
