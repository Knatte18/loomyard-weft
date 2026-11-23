```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens. I reran the searches for F4 (runners that wait on agent runs) and F8 (`SetReplyProbe` sites).

**Findings**

- **LOW — F4, runner sites not covered.** The fix wires the card fixer, and the summary's Bounds line now says the remaining unwired runners "wait on no agent run's turn end". That claim is not true for two of them.
  - `internal/shuttlecli/run.go:175` calls `c.runner.Run(spec)` on a runner built at `internal/shuttlecli/cli.go:140`. That runner has no notifier and no probe.
  - `internal/orchcli/start.go:198` calls `s.runner.Start(spec)` on a runner built at `internal/orchcli/cli.go:252`. That runner has no probe either.
  - Whether either run's prompt renders the parent directive was not checked. If one does, the directive's "your run waits for the answer" promise is false for it.
  - Fix: confirm both, then either wire the probe or word the Bounds line as "runners of runs whose prompt carries no parent directive".
- **LOW — F4, webster standalone wiring.** `internal/webstercli/wiring.go:~342` is a second `setRunner` path with no notifier and no probe. Only the hub path (line 204) got `SetReplyProbe`. A standalone run has no orch to ask, so this is probably intended, but the summary does not say so.

**Checked, no gap**
- `internal/loomcli/looparm.go:196` and `internal/battencli/wire.go:1431` runners only stop strands; neither waits on a run.
- `weavercli/wiring.go:44` is a reset runner that calls `ProbeGated` only, so skipping it matches the fixer's decision.
- F8: the wired `SetReplyProbe` sites are burler (wiring.go:152), loom (wiring.go:714), batten (drive.go:518), webster (wiring.go:204) and gatecli (cardfixer.go:134). Four of those five have a wiring assertion.
- No finding: the card fixer's probe at `internal/gatecli/cardfixer.go:134` has no assertion.
- Not checked: F3's other `releaseGoneHolder` callers, and the `sweepSubjects` consumers.

**Tool calls**
1. grep of `looparm.go` for runner use, and of the shuttlecli, orchcli and tendercli files for `Wait`/`Start`/`Run`. My context had only the `NewRunner` site list, not what each runner is used for.
2. grep for `.Wait()`, `runner.Start` and `runner.Run(` in shuttlecli, orchcli, tendercli, pairteardown and creelcli, plus a grep for every `SetReplyProbe` and `SetNotifier` site. This confirmed which unwired runners start runs and that the wired set is complete.
