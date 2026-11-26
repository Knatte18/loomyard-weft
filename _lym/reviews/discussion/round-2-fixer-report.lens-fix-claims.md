```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: I checked every claim about the code that the fix added or rewrote against HEAD.

no findings

These claims hold at HEAD:
- **hubforge's imports:** `internal/hubforge` already imports `internal/fabriccli` (`hub.go:18`), so a call to an installer there adds no import.
- **No cycle:** neither `internal/fabriccli` nor `internal/hubforge` is in the dependency closure of `shedrun`, `shedengine`, `stencilstore`, `vscode` or `ideengine`. An installer in fabriccli that imports their adapters is therefore cycle-free.
- **The one new import:** fabriccli imports `shedengine` and `shedrun` directly today but not `stencilstore`. "The cut extends them only by the seam installer's imports" therefore names a real extension, and that is the only one.
- **The workspace hook in ideengine's test:** `internal/ideengine/spawn_hub_integration_test.go:180` and `:372` install the workspace-content hook today, as the fix says.
- **beamcli and machinecli can import ideengine:** ideengine's closure holds neither package, so those tests can install the IDE module's builder.
- **`internal/state` merges by type, not by bytes:** `keepUnknown` decides from the written Go type (`internal/state/keep.go:34`), and it treats `json.RawMessage` as opaque (`keep.go:58`). So a merge over plain bytes would drop unknown keys, which backs the claim that "the rewind write runs in the adapter".
- **"as today's code does":** the code-tip settle writes a `shedengine.Status` through `state.WriteJSON` today (`statusAtCommit`, `codetipsettle.go:111`; the write is at `:90`).
- **No lost shed write:** the takeover write's lock path is `shedrun.StatusLock` (`takeoverwrite.go:260`). The code-tip settle and shed's own `UpdateJSON` (`shedrun/intents.go:22`) use the same lock, so "a shed write under the same lock is never lost" holds.
- **The takeover write is a one-key patch:** today it writes `takeoverStatusPatch` (`takeoverwrite.go:58-62`). The JSON shape of `shedengine.Takeover` is `machine`, `subject` and `at` (`shedengine/status.go:164-171`), so "fabric's own takeover entry type, whose JSON shape is today's" can be met.

Further calls:
- I read `statusAtCommit`'s signature and the takeover's `statusFile` lock path. My context lacked the status type of the code-tip write and the lock the takeover write takes.
