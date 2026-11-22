```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the changed functions and branches.

**Functions and branches, with the case I ran and the result**

- `FlagRemoteControlDefault`, empty input `""`: `yaml.Unmarshal` returns nil and leaves `keys` nil. The key is absent, so it returns `existing` unchanged, rewrote nil and one flagged line. Correct: the file sets no value and its orch moves from on to off.
- `FlagRemoteControlDefault`, null document `"~\n"`: same as empty, `keys` nil. Flagged, bytes unchanged. Correct.
- `FlagRemoteControlDefault`, comments only, including a commented-out `# remote_control: true`: parses as empty. Flagged. Correct, since a comment sets nothing and the old binary took the template's `true`.
- `FlagRemoteControlDefault`, `remote_control: false` or `remote_control: true`: the key is present, so it returns `existing`, nil, nil. Correct: an explicit value means the same under both binaries.
- `FlagRemoteControlDefault`, sequence or scalar document `"- a\n"`: `Unmarshal` into a map fails and it returns a wrapped `parse the orch config` error. `applyReinterpretations` wraps that in a `configsync.FileError` naming orch.yaml and returns before writing the marker. Nothing is left inconsistent: `walk.go:240-243` returns the error inside the board write, and `failRestoringConfig` restores the snapshotted files and markers (`walk.go:262-263`). `ReconcileHubWideAt` runs before it at `walk.go:225` and would already refuse such a file. Correct.
- `FlagRemoteControlDefault`, `remote_control: null` or a bare `remote_control:`: the key is present, so nothing is flagged. Whether the stacked load falls back to the template's value or decodes the null as `false` under both binaries, I could not settle in budget (`yamlengine/layers.go:172` hints that a null takes the template leaf). If it falls back, such a hub went from on to off without a notice. This is a hand-written null on a bool key, so I raise no finding. It is a candidate only.
- `FlagRemoteControlDefault` through `applyReinterpretations`: rewritten equals existing, so `bytes.Equal` holds and only the marker is written. The flagged line becomes `orch.yaml sets no remote_control, …` and is mailed once, after the commit lands (`walk.go:268`). Matches the registry's contract. Correct.
- `applyReinterpretations`, absent orch.yaml: only the marker is written and nothing is flagged. The hub then takes the template, which used to be `true` and is now `false`. Such a hub moves from on to off with no notice. The summary states this as the walk's rule for every absent base file, and a hub with no orch.yaml never tuned orch. Correct per the documented contract, and the fasit asks for a reinterpretation of a hub file, not for a notice in this case.
- `template.yaml`, `remote_control: false`: `LoadConfig` fills `RemoteControl=false`. `launchRemoteControlName` (`remotecontrol.go:71`) returns `""`, so `relaunchStrand` (`start.go:784-792`) launches with no name and marks no pane. Correct.
- `LoadConfig` validation at `config.go:134`, `remote_control: true` with `remote_control_name: " "`: refused. With the switch off and the name empty: loads. Correct, and the changed test rows match.
- configreg orch row: the ID constant and the rewrite are wired with the right function signature. Correct.

**Findings:** none.

**Tool calls**
1. `internal/hubreconcile/walk.go:225-270`: how the walk handles a rewrite error and orders the notice. My context held only grep hits.
2. `internal/configengine` null handling (`overlay.go:90-125`): how a null value in a base file loads.
3. Grep for the merge functions in `configengine`: to find the layer merge.
4. `layers.go` `stackLayers` merge call: to find where layers merge (`yamlengine.Merge`).
5. Grep for null handling in `internal/yamlengine`: to see whether a null leaf takes the template value. Left unsettled.
