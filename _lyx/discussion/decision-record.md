# Decision record — run-recovery

## Goal

The operator and the orch can see what a run waits on, and a run comes back up as it was after a crash or a halt.
Concretely: every Go-side wait after an agent's turn shows in `lyx loom status` and on screen with its elapsed time, a verify gate cannot hang without a bound, reed revives only strands with work left and never stacks a bare claude over a dormant orch, `lyx batten run` brings a halted child's driver back without resuming the child, and the orch resumes a child through a verb that only wakes its driver.

## Scope

In:

- A running verify gate in `lyx loom status`: site, elapsed time and command, at every site that runs `verifytree.Verify` (Webster gate, Webster-Burler round gate, Publish and Finalize merge-in verify, `webster verify`).
- Shuttle's other post-turn waits in status too: any gate entry it evaluates, and the background-shell wait (`background_shell_wait_min`) after a turn end, the wait #420's correction found behind the "nothing shown" case in `bugfix-webster`.
- The same waits on screen: a pane user option on the waiting strand, rendered by reed's status line with elapsed time.
- A deadline on the verify command, with the whole process tree killed on expiry (#420).
- reed: `--unless-name` matches a dormant strand (#408); `reed resume` drops, rather than revives, a strand whose work is done (#409).
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
Loom's `Waiting` hook reads, in order: the verify marker, the shuttle wait marker of the worktree's live runs, the parent-review note.
The producer name comes from the run's status (current producer).
Rationale: the verify marker already exists and reports correctly; the `bugfix-webster` blank was the background-shell wait, which has no marker. One read path keeps status and the orch's view consistent.

### D2 — On screen: a pane user option on the waiting strand

Decision: reed gains one engine op that sets or clears a wait mark on a strand's pane as two pane user options (`@lyx_wait`, the label; `@lyx_wait_start`, epoch seconds).
Shuttle sets it on its agent strand when a gate entry's evaluation or the background-shell wait starts, and clears it when that wait ends, on every path out (pass, fail, error, cancel).
Shuttle also clears it on the strand when a run starts or attaches, so a mark left by a crashed step is gone at the next touch.
The landing producers' merge-in verify sets the mark on the run's driver strand through a told callback in `landingshed`'s deps; loom wires it to the driver strand it already resolves, and a run with no driver strand (Go driver) gets the status note only.
reed's default status line renders every marked pane of the session as `<pane title> ⏳<label> <elapsed>`, elapsed computed by tmux at each refresh; the segment is a status-line template token, so a custom template can place or drop it.
Rationale: shuttle owns the gates and the shell wait (operator, 2026-10-07); pane options die with the pane, so a reborn strand never carries a stale mark. Bound: the mark is display only, never read by any Go decision.

### D3 — The verify command has a deadline and its whole process tree dies on expiry

Decision: `verifyrun` starts the shell in its own process group (Unix) or a new process group (Windows) and, on context cancel or deadline, kills the whole group, not only the shell; `cmd.WaitDelay` stays as the bound on the pipe read.
`verifytree.Verify` runs the command under a deadline (a named constant, 60 minutes, well above the 8–15 minutes observed under contention).
An expired deadline is a failed verify with a reason naming the deadline, never a returned error: the gate fails with findings naming the timeout, the log path and the log tail, writes no record, removes the marker.
A caller's own context cancel stays a returned error, as today.
Rationale: #420's correction established that nothing bounds the gate. A timeout is a defect the writer can act on (a hanging test), so it rides the normal failed-gate path. Bound: the deadline only shortens a verify that would otherwise run unbounded; it lets no failing tree pass.

### D4 — `--unless-name` matches a dormant strand (#408)

Decision: `AddStrandUnless` skips the add when a strand named `unlessName` exists in this worktree's reed state, live or dormant; the flag's help and the vscode chain's comment say so.
Rationale: the only caller is the folder-open chain, whose intent is "never stack a bare claude over the orch". The dormant orch comes back through `lyx orch start` as today. Bound: the skip only ever suppresses an add; it launches, revives and removes nothing.

### D5 — reed resume revives only strands with work left (#409)

Decision: a reed strand may carry a list of done-when paths, set by its spawner at add time.
`Resume` drops from state, instead of relaunching, a non-live strand whose done-when list is non-empty and whose every path exists.
Shuttle sets a strand's done-when list to its run's output files, the Completion Signal's own test of a finished run.
Strands without a list (driver, orch, operator strands) are relaunched as today.
Rationale: reed cannot import shuttle (shuttle imports reed), and the output files are what shuttle already treats as "finished" (`PATTERN-completion-signal`). Dropping, not skipping, keeps a finished strand from lingering for the next resume. Bound: only non-live strands are dropped, only when every listed path exists; a live strand is never touched, and a strand whose run is still working (outputs missing) is relaunched as before.

### D6 — `lyx batten run` brings its child back up as it was

Decision: batten's inner-run row, on a child it does not hold live, restores it once per batten process:

- A running child whose spawn was confirmed by an earlier, now-dead batten process is spawned again (the child's bootstrap, idempotent against a live driver, takes the crash-resume path); the spawn confirmation is scoped to the batten process that wrote it.
- A halted child (blocked, paused, failed) or an awaiting one whose driver strand is not live gets its pair's strands revived through reed resume (D5 applies), which brings back the parked driver with its own session; the run itself stays halted, and the park marker stays, so a later resume wakes that driver.
- A revive that fails, or leaves the driver still not live, is warned once and named in the halt notice with the way forward (`lyx loom start --no-attach` in the task worktree); batten does not retry it in that episode.

Rationale: the operator wants the child back "as it was, driver included", while the halt stays the child's own (operator, 2026-10-07). Bound: the revive runs at most once per halt episode per batten process, starts no shed step, removes no park marker and sends no resume line.

### D7 — `lyx loom resume`, the orch's resume verb

Decision: a new loom verb, run in the task worktree, that resumes a halted run only by waking its live, parked driver: it types the driver resume line and removes the park marker, the same branch `lyx loom start` takes for a parked driver, shared rather than copied.
It never spawns a driver, never creates or attaches a strand and never switches a client.
It reads only strand liveness, the park marker and the run's state, never the seed's driver value (`PATTERN-driver-choice-single-site`).
Outcomes:

- parked live driver: resumed; the envelope names the stop-report path;
- running run with a live driver: a no-op success saying the run is already running;
- halted at a hand-back, driver not yet parked: refused, retry in a few seconds (start's existing refusal kind);
- no live driver strand: refused; the way forward names `lyx batten run <slug>` from the prime, which brings the driver back (D6), then `lyx loom resume` again, or `lyx loom start --no-attach` in the task worktree for a run batten does not hold;
- done or awaiting run: refused; awaiting is resumed by batten on a recorded decision.

Every refusal gets its spec row and a test (`PATTERN-refusal-way-forward`) and names no denied command (`PATTERN-no-denied-recovery`).
The orch role stencil names `lyx loom resume` wherever it names `lyx loom start` for a halted child, after a circling decision and after a goto, and drops "message the driver by name to resume" as a separate step.
Rationale: the orch's `lyx loom start` calls spawned and, on 2026-10-07, did not move the runs; a live driver resumed only when messaged. Waking the parked driver is the one effect the orch needs. Bound: the verb can only deliver the resume line a parked driver already waits for; it cannot start anything that is not there.

### D8 — Docs in the same commit

Decision: the package `doc.go` of each touched package (`verifyrun`, `verifytree`, `shuttleengine`, `reedengine`, `battenshed`, `loomcli`, `landingshed`) records its new behaviour; the orch role stencil changes per D7; no PATTERN entry is added, since no new cross-cutting rule arises.
Rationale: CLAUDE.md's docs-land-in-the-same-commit rule.

## Constraints

- `PATTERN-told-geometry`: `verifytree`, `shuttleengine`, `reedengine` and `landingshed` get every path and the wait-mark callback told; none resolves a cwd.
- `PATTERN-lyxdirs-single-declarer` and `PATTERN-durable-vs-ephemeral-state`: the shuttle wait marker lives in the shuttle run directory under `.lyx`; no new `_lyx`/`.lyx` literal.
- `PATTERN-shuttle-provider-seam`: the wait mark and done-when list are provider-neutral and set in `shuttleengine`, never in `claudeengine`.
- `PATTERN-completion-signal`: D5's done-when list is the output-file set; no other "finished" test is invented.
- `PATTERN-driver-choice-single-site`: `lyx loom resume` never reads the seed's driver.
- `PATTERN-refusal-way-forward`, `PATTERN-no-denied-recovery`: every new refusal (D7, D6's notice) names a lyx verb as its way forward, with a spec row and a test.
- `PATTERN-cli-cobra`, `PATTERN-cliwire-sole-wiring`: `lyx loom resume` is a cobra verb in loom's subtree, wired through cliwire, JSON errors.
- `PATTERN-sandbox-coverage`: the sandbox suite exercises `lyx loom resume` or excludes it with a reason.
- `PATTERN-spawn-observability`: the verify process-group kill and batten's revive are logged.
- `PATTERN-shell-mechanics-seam`: process-group setup stays in `verifyrun`/`internal/proc`, not `internal/shell`.
- `PATTERN-orch-pane-single-writer`: nothing here types into the orch pane; batten's halt notice keeps going through `orchengine`.
- `PATTERN-test-speed`, `PATTERN-test-isolation`, `PATTERN-test-economy`: elapsed-time rendering uses an injectable clock; tmux-dependent checks sit in the `tmux` tier.
- `PATTERN-comment-line-breaks`: semantic line breaks in every new Go comment.
- Windows: process-group kill and pane options must hold on Windows or be stated as Unix-only with the Windows fallback (kill the shell only, as today) logged.

## Auto-mode assumptions

- The `bugfix-webster` blank in status was shuttle's background-shell wait, per #420's correction, not a verify path that skips the marker; D1 covers both.
- Publish/Finalize's on-screen mark goes on the driver strand, since no agent strand waits there; a Go-driven run shows it in status only.
- The verify deadline is a constant, not a config key: no operator has asked to tune it, and one value serves every site.
- `lyx loom resume` lives in loom, run in the task worktree, like `lyx loom approve`; no batten-level resume verb, since batten polls the child and needs no wake-up.
- "Records the decision" in the board's direction is read as the resume's own trace (log line, stop-report path in the envelope, park marker removed); no new decision record type.
- A strand added before this change carries no done-when list and is relaunched as today.

## Open risks

- tmux's elapsed-time arithmetic in a status format (`%s` against a pane option inside `#{e|-:…}`) must be checked live on the tmux version in use; if it does not expand, the fallback is to show the start time (`since HH:MM`) instead of elapsed time.
- A crashed step leaves the wait mark on a surviving agent pane until shuttle next touches that strand; `lyx loom status` stays the truth, since its markers check pid liveness.
- `shuttle-turns` edits `internal/shuttleengine/wait.go` around the background-shell wait at the same time; the two runs must be compared at PR-Gate.
- A gated run that crashed with outputs present but its gate unpassed is dropped by D5; the next step's Attach harvests it and evaluates the gate, and a failing gate then has no live agent to re-prompt. This is today's behaviour for a dead strand, unchanged by this task.

## Acceptance criteria

- `lyx loom status` during a verify gate at each site (Webster gate, Webster-Burler round gate, Publish merge-in) shows producer, `verify running <elapsed>` and the command.
- `lyx loom status` during shuttle's background-shell wait and during a non-verify gate entry shows that wait; a marker with a dead pid shows nothing.
- The wait mark is set on the right strand's pane when a shuttle gate or shell wait starts and cleared on pass, fail, error and cancel; a run start or attach clears a stale one.
- The default reed status line renders a marked pane with its title, label and elapsed time; an unmarked session renders as before.
- A verify command whose descendant holds the output pipe and outlives the deadline ends as a failed gate naming the timeout; no process of its group survives.
- A cancelled verify still returns an error and writes no record.
- `reed add --unless-name orch` with a dormant orch strand in state adds nothing; with no orch strand it adds.
- `reed resume` relaunches a non-live strand with no done-when list and one whose done-when paths are partly missing, and drops a non-live strand whose done-when paths all exist; a live strand is untouched.
- A shuttle-spawned strand carries its run's output files as its done-when list.
- `batten run` over a halted child with a dead driver strand revives the pair's strands once, leaves the child's state and park marker as they were, and does not revive again in that episode; a failed revive is warned and named in the halt notice.
- `batten run` over a running child whose spawn confirmation came from a dead batten process re-spawns the child's bootstrap.
- `lyx loom resume`: parked live driver → resume line sent, marker removed; running → no-op success; not yet parked → refusal; no live driver → refusal naming `lyx batten run <slug>`; done/awaiting → refusal; no path spawns, adds a strand or switches a client.
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
