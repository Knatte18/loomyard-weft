```yaml
lens: fix-affected-set
outcome: reported
```

No findings. The three finding sets are covered; no call site, test or doc is left on the old behaviour.

Each finding's set, as I checked it:
- **F1 (callers of the probe):** `grep -rn "sandboxProbe\|launchSandboxMode" internal` (run earlier in this session) lists exactly two callers, `Prepare` and `OpenLaunch`, both through `launchSandboxMode`. Both inherit the new `errSandboxProbeNotRun` branch.
- **F1 (rows for the new branch):** `TestPrepare_SandboxMode` has the two new did-not-run rows, one under `auto` and one under `workspace-write`. `TestRunSandboxProbe` has the missing-binary row. `TestOpenLaunch` has no did-not-run row. That is not a gap, because both entry points share `launchSandboxMode` and `Prepare` pins it.
- **F2 (probe stubs):** the only stubs are in `fixture_test.go:151`, `prepare_test.go` and `open_test.go`. Both entry-point tests now record the probe arguments.
- **F3 (doc sites):** `sandbox.go` carries the limits sentence. `doc.go:19-26` and `docs/overview.md:700` say "cannot start", which still holds for an exit-error refusal. Neither mentions the new refuse-on-did-not-run behaviour. That is a documented-limit gap, not a contradiction, so I'm not raising it as a finding.

Calls made:
1. `grep` over `docs/overview.md` and the `codexengine` `doc.go`, `config.go` and `codexengine.go` for "cannot start" and "probe", to find docs describing the probe-failure behaviour that the context lacked.
2. Read `doc.go` lines 17-30, to check whether its auto fallback paragraph still agrees with the change.
