# Card 7 — orch-start-decisions

**Create:**
- `internal/orchengine/launch.go`
- `internal/orchengine/launch_test.go`

**Uses:**
- `internal/orchengine/state.go`
- `internal/orchengine/prompt.go`

**Intent:**
Put `lyx orch start`'s two judgments in the engine as pure, testable functions, so the verb in card 12 is assembly over them.

`StartAction` enumerates D5's three branches: `StartAttachOnly` (live strand, live watcher), `StartSpawnWatcher` (live strand, no watcher) and `StartRelaunch` (dead or absent strand).
`DecideStart(strandLive, watcherLive bool) StartAction` maps the pair onto them;
a dead or absent strand relaunches whatever the watcher's state, since a relaunched session needs a watcher bound to it anyway.

`ChooseStartPrompt(stencilsDir, handoffFlag string, s State, exists func(string) bool) (prompt, source string, err error)` applies D6's order:

1. A non-empty `handoffFlag`: the resume stencil pointed at it.
   A flag naming a file that does not exist is an error, since the operator named a specific file and a silent fallback would resume from the wrong context.
2. Otherwise `s.LastHandoff`, when it is non-empty and `exists` reports it present: the resume stencil pointed at it.
3. Otherwise the start stencil.

`source` names the branch taken (`flag`, `last-handoff` or `fresh`) for `start`'s envelope and log line.
Only `LastHandoff` is ever consulted, never `PendingHandoff` or a listing of `HandoffsDir`, so a partial handoff an aborted cycle left behind is never chosen (D6, D11).
`exists` is a parameter so tests need no disk.

`launch_test.go` seeds a temporary stencils directory as card 5's test does and covers:
all four `DecideStart` combinations;
the flag winning over a recorded last handoff;
a missing flag file erroring;
the recorded last handoff chosen when present and skipped when `exists` reports it gone;
a state whose only handoff is a `PendingHandoff` from an aborted cycle falling through to the start stencil;
and each branch's `source` value.
