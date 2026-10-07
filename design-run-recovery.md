# A run shows what it waits on and comes back up as it was

`run-recovery` · Task · bug, enhancement, loom, reed, batten, orch · paused · Discussion-Bouncer

## Goal

The operator and the orch can see what a run waits on, and a run comes back up as it was after a crash or a halt.
Split from the run-errors bundle (operator, 2026-10-07).

## A running verify gate is visible in status and on screen

After an agent ends its turn, a Go gate (`verifyrun`: build, unit, integration, smoke vet) can run for many minutes while the agent's pane sits idle; the operator and the orch read it as a hang (`orch-cycle-hardening` and `board-config-per-worktree` 2026-10-05, `bugfix-fabric` and `bugfix-webster` 2026-10-07, where the orch SIGQUIT-ed a gate it took for part 1).

- `lyx loom status` names the gate and how long it has run, such as `Webster-Burler: verify running 6m (go test -tags integration)`, for every producer.
  On 2026-10-07 `bugfix-sessions` showed `verify Webster-Burler gate (attempt 1, since 10:15)` in `waiting` while `bugfix-webster`, in the same producer, showed nothing: find which gate path skips it.
- On screen: shuttle owns the gate, so it sets a pane user option on the agent's strand when a gate starts (e.g. `@gate=verify`, `@gate_start=<time>`) and clears it when the gate ends; reed's status format shows it with the elapsed time computed at each refresh, such as `webster ⏳verify 2m14s` (operator, 2026-10-07).
- The same holds for every Go-side wait after an agent's turn: the Webster verify gate, the Burler round's verify, Publish's merge-in verify.

## reed: stray strands after a crash (#408, #409)

- **Bare claude above a dormant orch (#408).** The folder-open chain in `internal/vscode/config.go` runs `lyx reed add --if-absent --unless-name orch --name claude`; `--unless-name` skips only while the orch strand is live (`AddStrandUnless`, `internal/reedengine/strand.go`), so after a reboot a bare `claude` strand lands above the dormant orch.
  Expected: an orch strand, live or dormant, blocks the bare add; let `--unless-name` match a persisted dormant strand, or run `lyx reed resume` before the add.
- **Resume revives finished strands (#409).** After a crash, `lyx reed resume` in `test-suite-prune-core` recreated a burler whose round was done, beside the next producer's own strand.
  A strand stays in reed's state after its work is done, since only the shuttle wait that would remove it died with the crash.
  Expected: resume brings back only strands with work left, or a strand leaves reed's state as soon as its shuttle run reaches a final outcome.

## batten run brings its child back up

`lyx batten run <slug>` on a run whose batten process had died, with its inner loom run halted (paused, blocked or failed), started batten, which logged "inner shed run halted; waiting for the operator to resume it" and waited without bringing the loom run back up; the orch had to run `lyx loom start --no-attach` in the task worktree as a second step.
Seen 2026-10-07 restarting the three bugfix runs after their batten processes died; the runs went on only once their drivers were also messaged.

Wanted (operator, 2026-10-07): `batten run` brings its current producer, the loom run, back up as it was, driver included.
The loom run itself stays where it halted, since that halt is its own; resuming it remains a separate step (a resume, or a message to its driver).

## An orch resume verb

Seen 2026-10-06: the prime's reed window jumped to other sessions "by itself" while the operator watched the orch.
The cause was the orch's own `lyx loom start` calls in task worktrees: the orch runs inside reed's tmux, so `$TMUX` is set, and `lyx loom start` without `--no-attach` runs `tmux switch-client` onto the task's session (`internal/loomcli/start.go`), taking the operator's terminal with it.
`lyx orch start` does the same without `--no-attach` (`internal/orchcli/handover.go`).

The orch role (`contracts/stencils/orch/orch-template-role.md`) tells the orch to resume a halted child with `lyx loom start`, an operator verb whose job includes handing over the terminal.
In practice the call also did not move the runs: a live driver resumed only once the orch messaged it.

Direction:

- The orch resumes a child through a verb with no interactive side effect: a resume verb that only records the decision and wakes the run's driver (or batten), never spawning or attaching anything.
  The role stencil names that verb and never `lyx loom start`.

Start verbs no longer attach for any caller; attaching is only `lyx reed attach` (operator decision 2026-10-06, landed in `bugfix-sessions`, 26f329be7).
This note keeps the orch resume verb.


## From issue #420

Imported from [issue #420](https://github.com/Knatte18/loomyard/issues/420).

## What happened

A `lyx shed step` on a Burler round (Webster-Burler, round 3) never returned an envelope.
The round's agent finished, but the step stayed in the gated boundary for about 19 minutes until the operator's delegate sent SIGQUIT; the process exited with code 2.
The trace held `shed: step` and no `shed: step done`, and no `fabric: mutation` records.
The status afterwards was unchanged: `state: running`, `last_step.finished: false`, `interrupt_policy: reinvoke`.
The next `lyx loom step` then wrote a `loom-crash-resume` note for a suspected driver crash.

The goroutine dump shows the main goroutine blocked in a syscall wait on the child process, with the stack:

`verifyrun.run` -> `os/exec.(*Cmd).Wait` <- `verifytree.Verify` <- `loomshed.NewVerifyGate` <- `burlerengine.repairReportBeforeGate` <- `shuttleengine.(*Run).evaluateGate` <- `handleGatedBoundary` <- `Wait` <- `RunGated` <- `burlerengine.(*Engine).Run` <- `shedadapters.(*BurlerProducer).Call` <- `shedengine.(*Shed).Step`.

All other goroutines were idle, so nothing else was working: the step was waiting only on the verify command's `Cmd.Wait`.

## Why this is abnormal

The verify gate is meant to end in a pass or fail verdict.
`verifyrun.run` is passed a duration argument (a timeout), yet the wait outlived it by many minutes.
A likely cause is that the verify command's descendants keep the output pipes open, or the child is not killed on timeout, so `Cmd.Wait` does not return.
Which of those it was is not established by the dump.

## What to do

- Make the verify command's wait bounded: kill the whole process group on timeout and set `Cmd.WaitDelay` so `Wait` returns even when a grandchild holds the pipes.
- Treat a timed-out verify as a gate failure with a clear reason, not a silent hang.
- Consider a step-level watchdog that logs which gate the step is waiting in, so an operator can see the hang without a SIGQUIT dump.

Provenance: task `bugfix-webster`; notes `repair-20261007-webster-burler-r3-sigquit.md`, `loom-crash-resume.md`.

Correction (orch, 2026-10-07): the premise above does not hold.
`verifyrun.run`'s duration argument is `cmd.WaitDelay` (10 s), which only bounds the pipe read after the process exits (`internal/verifyrun/run.go`); nothing puts a deadline on the gate.
The dump caught a gate about 20 s old: `.lyx/verify/running.yaml` appeared at 10:33:37 and the SIGQUIT came at 10:34, while the burler's turn ended at 10:23.
The ten minutes before the gate were most likely shuttle's `background_shell_wait_min` (10) wait after the burler's turn end, with background agents showing in its pane.
So two things belong here: that wait is silent in status and on screen, looking the same as a hang or a gate, and a verify gate has no deadline.
