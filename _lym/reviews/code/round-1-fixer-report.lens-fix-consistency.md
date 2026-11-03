```yaml
lens: fix-consistency
outcome: reported
```

Task: the fix-consistency check of the F2–F5 change set.

No findings.

- **Docs and comments:** the new `internal/orchcli/cli.go` note and the `internal/loomcli/doc.go` clause both say a live strand of the running build keeps its title. That matches `ensureRunsStrand`, which returns early when `StrandCurrent` holds, and `resolveStatusStrandAction`'s keep branch. The `doc.go` wording also agrees with the `cli.go` field comment, which already said the title is read when the strand is added or replaced.
- **Twin sites:** `ensureStatusStrand` now builds its spec through `statusStrandSpec`. The only other `statusStrandAddSpec` callers are in `smoke_drivingsurface_test.go`, which pass the title explicitly and are unaffected.
- **Test constant:** the orchcli harness constant `harnessRunsWindowTitle` replaces the stubbed `"beam: prime"` in both the stub and `wantDisplay`. The remaining `"beam: prime"` literals are in `TestRunsWindowTitleOf` and `runstitle_integration_test.go`, which call the real title functions and never use the harness stub.
- **Decision record:** the new `statusStrandSpec` test expects `T4: darn`, the `<handle>: <recipe>` form that parent decision #528 recorded.

Further tool calls: none, because my context already held every changed file.
