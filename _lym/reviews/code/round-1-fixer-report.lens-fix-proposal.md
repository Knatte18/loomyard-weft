```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1, F2 and F3 proposals, all fixed as proposed.

**Findings**

1. **LOW** — F2's proposal covers only the two failures the review named, in `promptHook` (`internal/slackcli/hook.go`).
   - The `slackengine.WritePresence(... PresenceTerminal ...)` failure in `promptHook` returns an error with no `logger.Warn`. That failure is the missed switch back to terminal, the same outcome F2 names.
   - `ReadPresence`, `orchClient` and `FormatProgram` errors in both hooks are also unlogged.
   - Concrete case: a read-only presence file means the operator is back at the terminal, presence stays `slack`, replies keep going to Slack, and the log is empty. The turn hook discards the returned error, so nothing records it.
   - Evidence: the F2 summary says "a missed switch back shows in lym's log", but only the creel `Log` failure of `promptHook` is logged.

2. **LOW** — F1's proposal left stale siblings, because it listed only non-test files.
   - `internal/orchcli/start_test.go:1337` says "presence sets the hub's presence to slack".
   - `internal/orchengine/watcher_test.go:1521` says "slack sets the hub's presence to slack".
   - Both still call the presence the hub's, which contradicts the reworded doc comments. The fixer's own grep (the `hub's presence` search) listed them.

3. **LOW** — F3's new test pins the guard only when the binary gets one positional argument.
   - `TestTestMain_ExitsTwoWithoutRunningTheSuiteOnAPositionalArgument` runs one case with the argument `slack-bar-state`. A guard that checks only `flag.Arg(0)` would still pass.
   - I found no wrong-result case, so this is a weak-assertion note only.
   - The tier-1 placement is unverified. `docs/benchmarks/running-tests.md:21` defines tier 1 as real external-binary spawn and tier 2 as tmux, so the placement is plausible.

I found no wrong-result case for F2's two `logger.Warn` calls. Both log before they return, and the "presence stays slack" claim holds because the `Log` failure returns before `WritePresence`.

My one check did not settle whether the logger needs initialising in the `slack hook` process. The `logger.(Init|Setup|Configure|Open)` grep matched nothing, and `internal/logger/logger.go` shows a ready default (`dualHandler`, `durableHandler`). So I found no initialisation problem.

**Tool calls**
- One `grep` of `logger.(Init|Setup|Configure|Open)` across non-test Go files, plus `^func` in `internal/logger/logger.go`: I needed to know whether the logger needs initialising in the hook process, and my context lacked the package.
