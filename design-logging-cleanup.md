# Logging cleanup: right levels, no per-tick noise, reed resize layout bug

`logging-cleanup` · Task · bug, enhancement, batten, reed, orch, fabric · awaiting · Discussion-Bouncer

Bring the logs to the level policy: ordinary states at Info or Debug, nothing logged per tick or per poll, and no test leaking WARN into verify logs.
The reed resize bug (`reed-resize-pane-count` below) is the largest single noise source and is fixed in this task.
Keeping pair logs in a git-ignored hub archive and a notice for every landing are folded in as the sections below.

## log-level-hygiene

### Seen (2026-10-13)

In the hub prime's trace logs (`.lyx/logs`, 10-10 to 10-13), 516 of 2339 lines are WARN:

| Count | Message | What it is |
|---|---|---|
| 415 | `battenshed: producer stuck` | Mostly normal waiting, such as `reason="inner shed run still running; checking it every 2s"` or `"child running → awaiting at PR-Gate"`, logged at each check |
| 79 | `orch: session state disagrees with the idle probe` | A real diagnostic, repeated per reading |
| 11 | `gitrepo: push refused, retrying` | A real anomaly that recovers by itself |
| 8 | `battenshed: inner shed run halted; waiting for the operator to resume it` | A pause, not a problem |

So about 80 % of WARN lines record an ordinary state as a problem: an action log, not a warning (operator, 2026-10-13).
`internal/logger/logger.go` states a level policy ("Level policy"); these lines break it.

### Idea

- Batten logs a producer's normal waiting at Info or Debug, once per change of state, never per poll; WARN only when a run is stuck in fact.
- The orch's idle-probe disagreement logs once per change, not per reading.
- A halted run waiting for the operator is Info.
- Check the other modules' WARN calls against the level policy while at it.

### Inventory (2026-10-13)

Full report: `.scratch/logging-inventory.md` in the hub prime, with a verdict and reason for each of the top 30 messages.
Data: 30,111 level lines in 647 trace files (8.8 MB) across the hub prime and live pairs; 89 % WARN, 11 % INFO, no DEBUG or ERROR.

- `reed: resize re-apply failed` (`internal/reedengine/watchloop.go:437`): 83.8 % of all lines, WARN on every failing tick with no dedupe; one file under `_board` holds 26,486 lines (7.7 MB), the same tmux layout error thousands of times per session. The repeating failure itself is a live bug: still firing on 2026-10-13 09:36 for session `go-driver-assistant`, with `select-layout: exit status 1: have 4 panes but need 3`, so the layout reed re-applies counts one pane fewer than the window holds (25,343 lines in the hub reed watchdog's trace file since 10-08).
- `claudeengine: could not read the session name` (WARN, 3.7 %): polled before the transcript exists; Debug, or once per session.
- `orch: injection held` (INFO, 1.6 %): the dedupe fails because 409 of 494 reasons embed the changing draft text; key it on a constant reason, and stop logging the operator's draft text.
- `battenshed: producer stuck` (WARN, 1.4 %): normal polled waits; Debug, WARN only for a real block.
- `reed: watchdog lock already held, exiting` (INFO, 1.1 %): 324 short-lived processes each open a full trace file; Debug, ideally no trace file at all.
- `battenshed: session state at notice` (INFO, 0.6 %): shadow logging; Debug.
- `battencli: create worktree` dumps the whole mutation struct, duplicating `fabric: mutation`.
- Everything below rank 12 is lifecycle logging the policy asks for.

Fixing the six main items removes about 91 % of lines (about 45 % outside the one pathological file).

Test noise: the default sink is stderr at Warn, and 38 packages with Warn or Error call sites have no test that redirects the logger, so their WARN lines leak into verify logs. One `logger.SetOutput(io.Discard)` in `tmuxkit.Main` (called from `TestMain`) would silence them; no verify log survived to count.

Related: the `run-log-archive` section below (keeping pair logs makes the noise cost more).

## reed-resize-pane-count

### Seen (2026-10-13)

The hub reed watchdog's trace file (`_board/.lyx/logs/trace-20261008T181526Z-f2e3f7df172f3d30-3854.log`) holds 25,343 lines of `reed: resize re-apply failed` (`internal/reedengine/watchloop.go:437`) since 2026-10-08, still firing on 10-13 09:36:

`session=go-driver-assistant err="select-layout: exit status 1: have 4 panes but need 3: …"`

The layout reed re-applies on resize counts one pane fewer than the window holds, so tmux refuses it on every tick.
The window's layout is then never restored after a resize.
The failure predates `reed-multi-window` (landed 2026-10-13), which moved strands into their own windows; whether that landing changes or removes the mismatch is unchecked.

### To find

- Which pane the layout leaves out: the selvage pane, a status pane, or a strand pane that the recorded state lost.
- Whether reed's recorded pane list and the live window disagree, and how they came apart.

The per-tick WARN with no dedupe is the `log-level-hygiene` section's part; this section is the layout bug itself.

## run-log-archive

### Seen (2026-10-13)

A study of whether `shuttle-hook-first` had an effect could not answer: task-pair trace logs (`<pair>/.lyx/logs`) are deleted at Worktree-Teardown, so almost no shuttle or batten logs from real loom runs survive (report `.scratch/hook-first-effect.md` in the hub prime).
The volume is small: 64 to 336 KB per live pair, 1.6 MB in the hub prime over four days.

### Idea (operator, 2026-10-13)

At teardown, copy the pair's `.lyx/logs` into an archive under the hub's `.lyx/` (git-ignored by definition, never `_lyx/`), for example `.lyx/log-archive/<slug>/`, and prune it after a retention period such as 30 days.
Logs never go into the weft: git keeps them forever and every clone carries them, while they are needed for weeks.

### Open

- The retention period, and whether it is a config key.
- Whether batten's own per-slug logs and verify logs (`.lyx/verify/`) go into the same archive.
- The durable trace sink writes Info and up (Debug only when `LYX_LOG_LEVEL=debug`); `LYX_LOG_LEVEL` sets only the stderr threshold and is an environment variable, not a config key. No setting suppresses WARN in the trace files, and none is needed for the archive's size.

## batten-land-notice

Batten sends a notice at a landing only when the landing makes another task ready (`landed; newly ready: ...`, `internal/battenshed/land.go`, once per batten run).
A plain landing sends nothing, so the orch does not learn of it: on 2026-10-12 `intake-bugs-1012` (19:09) and `cleanup-1012` (19:19) landed through the alignment agent's PR approve with no notice, and the orch reported them as still running until the operator pointed it out.
With the alignment agent landing tasks without an orch turn, the landing notice is the orch's only signal to check the Land row, prod and the board intake.

### What to fix

Every landing sends `<slug>: landed` (with `newly ready: ...` appended when it applies), and a failed Land row still sends its failure notice as today.
