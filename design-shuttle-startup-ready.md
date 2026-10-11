# Shuttle start and seat holds: tolerate a stray % at the prompt, probe every second, cap the seat hold deferral

`shuttle-startup-ready` · Task · bug, enhancement, shuttle, batten · running · Plan-Burler

# Part 1: stray % after the prompt fails the start (#549)

Imported from [issue #549](https://github.com/Knatte18/loomyard/issues/549).

## What happened

`lyx shed step` halted the `Darn-Write` producer as `failed` (anomaly `producer-hard-failure`) with:

```
shedadapters: Darn-Write (shuttle): shuttle run outcome died: shuttle: the provider never became ready
```

The trace shows the start sequence failing before the provider took any input:

- `shuttle: could not type the segment color` with `the session is not idle ... the pane is not idle; retry once the session is idle`.
- `shuttle: start failed before the provider took its first input; strand torn down`, cause `shuttle: load skills [scribe:prose scribe:code-quality]: shuttle: the session is not idle`.

The pane's last lines, quoted by lyx itself, show a fully started Claude Code UI waiting at an empty prompt, with a stray `%` after the prompt character:

```
❯ %
────────────────────────────
  ⏵⏵ bypass permissions on (shift+tab to cycle)
```

The first idle check failed about 70 seconds after the run started, and the skill-load idle check failed about 60 seconds after that, so the pane stayed "not idle" through both waits although the provider was visibly ready.

## Why it is abnormal

The provider was up and at an empty prompt, yet reed/shuttle classified the pane as not idle and gave up, tearing down the strand and failing the whole producer.
The stray `%` is likely a leftover from the shell's end-of-line marker or a partial draft, and idle detection treats any text after `❯` as an operator draft.
A later reflection agent spawned in the same worktree reached `busy` on the same Claude Code version without trouble, so the failure is in the idle/readiness detection, not the provider.

## What to do

- Make the idle check tolerate a lone trailing `%` (or similar non-input terminal marker) after the prompt character, or clear the input line before checking.
- On a not-idle result during start, retry or wait longer, rather than failing immediately with the generic `provider never became ready`.
- Include the pane's last lines in the final `shedadapters` failure message, as the trace already does, so the cause is visible without opening the trace.

Provenance: task `batten-notice-trim`, from `loom-halt.md`.

# Part 2: startup probe sleeps 10 s between checks

## Seen (2026-10-11)

`lyx batten run <slug> --window` opens the Konsole window about 11 s after the child loom run is spawned, every time (per-role-models 11.2 s; gate-test-hygiene, small-fixes-batch, seats-chair-first 11.0–11.1 s).
The Claude driver is up within about a second; the rest is a fixed wait.

- Batten opens the window only after `lyx loom start --no-attach` exits (`internal/battenshed/innerrun.go`, `innerRunProducer.Call`; `internal/battencli/wire.go`, `Spawn` runs it with `cmd.Run()`).
- `loom start` returns only after shuttle's `start` (`internal/shuttleengine/run.go`) has passed `awaitStartup` (`internal/shuttleengine/wait.go`), which captures the pane and between checks sleeps `poll_interval_ms × liveness_every_n_polls`, 1000 ms × 10 = 10 s.
  The 10 s cadence is deliberate there, to avoid stray keypresses during trust-prompt dismissal.
- The same `awaitStartup` gates every shuttle start, so every agent a run spawns (reviewers, fixers, judges, seeds, Webster's Master) likely waits up to 10 s longer than it needs to.
- Seats (bench pair `dbench-ohsh`, 2026-10-11): the chair went live at 10:44:16 and the advisor at 10:44:32, 16 s later, since the advisors start only after the chair; the chair's own `awaitStartup` wait likely accounts for most of it, and each advisor start adds its own. Not yet traced.
- The window runs only `lyx reed attach` in the pair (`internal/termwindow/termwindow.go`), which needs the pair's tmux session, up about 0.1 s after the spawn.

## Idea, ranked by gain

1. Probe every `poll_interval_ms` (1 s) during startup and replay the trust-prompt dismissal at most once per 10 s; saves about 8–9 s on every shuttle start, not only batten.
2. Open the window as soon as the pair's reed session exists rather than after `loom start` exits; saves about 11 s on each batten start, and batten then has to handle a spawn that fails after the window is open.
3. Minor, under 2 s: Seed-Child's `main-weft` push after the spawn, and no segment-color typing before the window opens.

# Part 3: seat chair hold deferral cap

Seen at burler-multi-llm PR-Gate (PR #552): `holdNoticeMaxDeferral` defers a seat chair's held notice up to 20 min while advisor work is outstanding, so an advisor stuck mid-turn hides a real chair stall for up to 20 min.

Idea: shorten the cap once runs show typical advisor batch times, or end the deferral when the advisor itself is held or dead.