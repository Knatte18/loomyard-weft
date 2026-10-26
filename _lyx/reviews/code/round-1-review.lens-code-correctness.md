```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness.** Each branch below was run with a concrete input.

**Per function or branch, case run, verdict**

- `retryExecAgain`, EAGAIN twice then success: returns the success after two pauses (250 ms, 500 ms). Correct.
- `retryExecAgain`, EAGAIN on every try: pauses 250, 500 and 1000 ms, makes 4 calls, and returns the last error unchanged. Correct.
- `retryExecAgain`, tmux ran and exited nonzero: `*exec.ExitError` does not match EAGAIN, so there is one call. Correct.
- `isExecAgain` through `wrapTmuxError`: that function wraps with `%w` (`overlay.go:176`), so an EAGAIN `PathError` still matches `errors.Is`. Correct.
- `runStdin`, stdin replay: `exec.Command` and `bytes.NewReader(stdin)` are rebuilt inside each attempt, so a retry replays the full stdin. Correct.
- `runStdin` with nil stdin, as `run` now passes: `cmd.Stdin` stays unset, which matches the old `run`. An empty non-nil slice gets an empty reader, so no behavior change. Correct.
- `output`, with and without `execHook`: the hook path retries on a hook-returned EAGAIN, and the exec path rebuilds the command per attempt. `checkFloor` runs once, before the retries. Correct.
- `start` loop with `StartRetries=0`: `attempts=1`, so any failure returns `(nil, result, err)`. Correct.
- `start` loop with `StartRetries=1`, first attempt `died` with the strand removed: it sleeps 5 s on the runner clock, then launches again into a fresh run directory, and returns the last attempt's error if that fails too. Correct.
- `start` loop, `timeout` outcome or a failed removal: `startRetryable` is false (`startupStrandRemoved` is set only on a successful removal), so there is no retry. Correct.
- `launch` failure before the strand registered: returns `(nil, Result{}, err)`, so `run == nil` and there is no retry. Correct.
- `launch` failure in `loadSkillsThenPrompt` that is not an `abandonStartup` result: `result.Outcome` is empty, so `startRetryable` is false. Correct.
- `removeStartupStrand`, fail, fail, succeed: sleeps 1 s then 2 s on the run clock and returns nil, and `startupStrandRemoved` is set. Correct.
- `removeStartupStrand`, three failures: no sleep after the last attempt, returns the last error, and the error text keeps the by-hand remedy. Correct.
- `startupCaptureExcerpt`, capture with at most `paneTailLines` non-blank lines: shown once under the last-lines heading. Correct.
- `startupCaptureExcerpt`, longer capture: `head = lines[:min(N, len-N)]` never overlaps the tail, so for 2N+ lines it shows N head lines and N tail lines. Correct.
- `startupCaptureExcerpt`, whitespace-only capture: gives an empty last-lines block, the same as the old `captureTail` behavior. Correct.
- `LoadConfig` with `start_retries: -1`: refused. 0 is accepted and means no retry. Correct.
- Held turn end from the gated plain-turn-end judgment: `unawaitedShellsOnly` requires non-empty tasks, so `background` is true. Correct.
- Held turn end from the ungated shell expiry: it reuses `run.waitingTurnEnd`, which carries `background` from its `EventWaiting`. Correct.
- Plain held turn end (`wait.go:912`): `background` is false, so it is notified. Correct.
- New switch case in `Wait`: placed after the usage-limit case and before `HoldDeferral == nil`, so no notice is sent, no deferral is recorded, and `NotifiedOffset` is untouched. Correct.

**Findings**

1. **LOW, design, `internal/shuttleengine/wait.go:772`.** `removeStartupStrand` retries on every `RemoveStrand` error with no check on the kind.
   - If a previous attempt removed the strand from state and killed its panes, then failed in a later step, the retry would likely fail with strand-not-found.
   - The strand would then be reported as not removed, and the start would not be retried.
   - I did not read `removeStrandLocked`'s unknown-guid result, so this is unconfirmed. It is not worse than before the change, apart from the added 3 s delay.
2. **LOW, design, `internal/shuttleengine/run.go:543-553`.** The start retry loop never checks `allOutputFilesExist` before launching again.
   - The decision record says a run whose output files exist is never retried.
   - The per-attempt `abandonStartup` sites I saw (`wait.go:687-725`, `run.go:747-784`) are not shown here to consult `allOutputFilesExist`, and I did not read the `awaitStartup` body.
   - Unconfirmed: output files existing is unlikely at start time, so this is the only gap against the stated bound.

**Tool calls**
1. Bash grep of `wrapTmuxError` and the `abandonStartup` call sites: to check whether `%w` preserves EAGAIN and which paths end `died`.
2. Bash grep of `Engine.RemoveStrand` in `reedengine/strand.go`: to check what a repeated removal returns.
