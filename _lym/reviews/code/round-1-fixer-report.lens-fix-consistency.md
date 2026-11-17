```yaml
lens: fix-consistency
outcome: reported
```

Task: check the three fix commits through the fix-consistency lens.

no findings

I grepped the repo for stale twins and found none. The patterns were the old "Every successful start", "see each change", "see them with", "a later start", `wroteStencilPins` and the bool-returning `writeStencilPins`. The `lym stencil diff` wording now agrees across the notice, the start and resume long help, and `loomcli/doc.go`. The `shedrun/doc.go` pins paragraph still holds. `bootstrapCommitPaths` has one non-test caller, and `runDriverSpawnAndWait` has none left outside `start.go` and the tests that take two return values.

One observation, not a finding: the `seedAndCommitBootstrap` doc and `bootstrapStageCommit` doc still list only the seed, status file, origin record and friction directory as committed. They never named the usage directory or the pins record, so this omission was there before the fix.

Tool calls: one `Bash` grep over the Go and markdown files, which my context lacked. A second `Bash` grep covered `docs`, `stencilcli`, `burlerengine`, `stencilstore` and `shedrun` docs for the same references.
