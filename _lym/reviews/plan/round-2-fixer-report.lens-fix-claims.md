```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD.

no findings

All the claims I checked hold:
- **A run command marks itself and its children.** Card 12 and the overview say a run command sets `pauserecord.RunProcessEnv` for itself and every process it starts. `internal/tendercli/gate.go:57` sets it, and `internal/tendercli/doc.go:54-55` says every process the command starts inherits it. The commands carrying `clihelp.RunProcessAnnotation` are `shedverbs` `run` (`internal/shedverbs/run.go:23`), `step` (`internal/shedverbs/step.go:329`) and one `beamcli` verb (`internal/beamcli/cli.go:694`).
- **A hand-run `lym reed window` in a pane is outside a run.** Card 12 says such a run is not registered. `stripIdentityEnv` in `internal/reedengine/lifecycle.go:691-700` removes `RunProcessEnv` before the tmux server inherits its environment, so pane processes are not marked.
- **`lym machine start` is outside a run.** None of the commands carrying `RunProcessAnnotation` is `machine start`.
- **Card 8's command line uses a valid flag.** `lym gate test --help` documents `--tier N` and says arguments after `--` fill each tier's `{args}`, so `--tier 3 … -- -run …` is accepted.
- **Cards 11 and 9 agree on the mode `off` result.** Card 11's `off` row says an entry with no home takes the snapshot desktop as both its `LastDesktop` and its `Home`, with its `Output` from the snapshot. That follows from card 9's rewritten `updatePlaces` rule, where a non-empty landed desktop becomes the home when there was none, and from `ReportUpdate` carrying the output.

Calls made:
- Read `internal/reedengine/lifecycle.go:685-705` and grepped for `RunProcessAnnotation` across `internal`: my context had only the grep line numbers, not the stripping code or the list of annotated commands.
- Ran `lym gate test --help`: my context did not have the `--tier` flag syntax.
