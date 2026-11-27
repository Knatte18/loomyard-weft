# Machine stops, dead judges and idle seats are not producer failures

`infra-stops-not-failures` · Task · bug, shuttle, claude, tender, gate, burler, shed, reed · running · PR-Review

Bundled from the notes below; each part keeps its note's text.

# Gate fixer interrupted by a machine stop is reported as dead (timeout)

Imported from [issue #248](https://github.com/Knatte18/loomyard/issues/248).

## What happened

A row gate spawned a cold fixer, and the machine rebooted under a minute later.
The run was re-stepped more than two days after that.
The loop's reconcile had stopped the fixer's shuttle run, but the gate fixer's record still read live.
The restarted driver waited on that record, classified it `dead` with reason `timeout`, and told the writer:

> The cold fixer ended without a reply (timeout); any change it left uncommitted is in the worktree.

The interrupted-stop detail, from the first entry after the reboot:

```
loop exit: not recorded, since no process marked the loop's exit, and the machine has booted since the loop started, so a stop of the machine likely ended it; (...) stopped shuttle runs: ly:<slug>:gate-fixer; left runs whose output files all exist: ly:<slug>:darn
```

The fixer's shuttle `run.json` (excerpt):

```
"createdAt": "2026-11-23T19:19:20Z",
"outcome": "stopped",
"roleKey": "gate-fixer",
"interrupted": true
```

The gate fixer's `records.json` after the resume (excerpt):

```
"spawnedAt": "2026-11-23T19:19:27Z",
"status": "dead",
"deadReason": "timeout"
```

## Why it is abnormal

The fixer did not run out of time: the machine stop ended it, and lym had already recorded its run as interrupted.
`Await` in `internal/gatefixer/await.go` checks the elapsed time since spawn before it checks whether the strand is live,
so any fixer interrupted by a stop that outlasts its timeout is reported as `timeout` instead of `died`.
The writer and anyone reading the record are sent looking for a slow fixer or a timeout that is too short.

The dead record also stays in the visit's set, which `internal/gatefixer/rowgate.go` counts against the fixer budget.
The interrupted fixer's session is not resumed, so the failure it was given falls back to the writer.

## What to do

- Report a fixer whose run is marked interrupted, or whose strand is gone across a reboot, as interrupted or `died`, not `timeout`.
- Consider resuming its session, or spawning a replacement that does not use up the budget, as is done for interrupted run agents.

Provenance: task `sandbox-without-new-kernel`, note `loom-halt.md`.


# A died bouncer judge session spends Code-Bouncer's bounce budget

Imported from [issue #239](https://github.com/Knatte18/loomyard/issues/239).

On a flaky network, Code-Bouncer's judge session twice failed to start: shuttle logged 'provider never became ready; strand torn down', relaunched, and the second run finished at once with no transcript. shedadapters then logged 'bouncer judge run did not complete ... outcome=died', and the row stopped with 'bounce budget exhausted for Code-Bouncer'. The judge never gave a verdict, so no bounce happened, yet the run stalled until a manual `lym loom resume`.

Expected: a judge that dies before giving a verdict is relaunched, or the row halts as an infrastructure failure, without spending the bounce budget meant for real verdicts.

# tender revive reads Claude Code placeholder after an API error as a typed draft and skips the session

## Seen

On 2026-11-26 two Claude sessions stalled on `API Error: Connection refused (ECONNREFUSED)`: `ly:rc-click-wrong-client-session:darn` and `ly:async-run-pusher:code-burler-fix`.
`lym tender revive` skipped both with reason `the input box holds a draft: "Resend your message to try again"`.
That text is Claude Code's own prompt after an API error, not text anyone typed, so revive never reaches the case it exists for.
No code on `origin/main` names the string.

## Workaround

The orch paused each run with `lym batten pause <slug>` and resumed it with `lym batten run <slug>`, which relaunches the row's agent on its own session.

## Expected

The input-box reader in `internal/shuttleengine/claudeengine` treats Claude Code's post-error prompt as an empty box, and `tender revive` gives the session its carry-on line.


# Shuttle idle check reports no input box on an idle seat, killing Plan-Burler-Review

Imported from [issue #219](https://github.com/Knatte18/loomyard/issues/219).

## What happened

The `Plan-Burler-Review` row (`burler-review`) halted `failed` (anomaly `producer-hard-failure`) after round 3: two consecutive `died` outcomes, both "review never started".
Each attempt's seat start ended with:

```
shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: strand "<id>": no input box is on screen; a menu or picker may be open; retry once the session is idle.
```

The pane lines quoted in the same error show the opposite: the skill-load reply `ok scribe:prose, scribe:code-quality, scribe:testing` had finished, a `done` status line was printed, and the `❯` input prompt plus the `bypass permissions on` footer were on screen.
The session was idle with an input box; the idle check reported none.
It happened on attempts 3 and 3b, in two separate halts, with the same pane text each time (`Sautéed for 3s`, `Cogitated for 3s`, `Cooked for 2s` status lines).

## Why abnormal

The seat was healthy.
A false "no input box" reading kills a seat that never got its prompt, burns the review rounds, and halts the whole run as `failed`.
The error text itself says "retry once the session is idle", yet the shuttle did not retry after the capture showed an idle pane.

## What to do

Check the input-box detection in the shuttle's deliver-after-skills step against a pane whose last lines are a completed skill-load reply, a `✻ <verb> for Ns · done` line, the `❯` prompt and the permissions footer.
Retry the idle check for a short window before declaring `died`, so a repaint race right after the skill-load reply does not kill the seat.

Provenance: task `models-torque-drag-adapter`; notes `loom-halt-2.md`, `loom-halt-3.md`.


# Review findings carried over from two landings

## Strand pane memory cap (`10ae7e112`)

- The degraded-launch Warn, on a host without the memory controller delegated to the user manager, has no test of its own.
- The OOM notice can name an earlier kill, or the wrong process, when several panes share a cgroup or the journal line lands after the kill is counted; a shared cgroup is only partly covered.
- The notice's delivery to the parent orch is untested end to end.
- The OOM step reads the strands, which waits on the operation lock; check that it does not stall the watchdog loop.

## Resume after a crash (`dfade5f9a`)

- `removeSupersededStrands` (`internal/shuttleengine/attach.go`) records a superseded running strand's stop as interrupted; check a test covers that path.
- The start retry loop (`internal/shuttleengine/run.go`) reuses the resume on a retried start; a resumed start that fails to come up should retry fresh.
