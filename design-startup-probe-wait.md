# Startup probe sleeps 10 s between checks; batten window waits for it

`startup-probe-wait` · Note · enhancement, batten

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
