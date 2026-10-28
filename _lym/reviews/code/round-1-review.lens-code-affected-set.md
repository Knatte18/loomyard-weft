```yaml
lens: code-affected-set
outcome: reported
```

**Affected-set list (lens: code-affected-set)**

1. **Sink check moved out of the scenario step (`hubmutation_integration_test.go`, b1fe0b2d).**
   - The set is the new serial test, the scenario step's remaining push checks, the process-wide sink writers, and the other readers of `TraceDir`.
   - The new test has no `t.Parallel`, so Go runs it to completion before any paused parallel test resumes. The anchor tests (`anchor_integration_test.go:33` cleanup, `anchordeps.go:273` repoint) and the scenario therefore cannot repoint the sink between the push and the read. Covered.
   - The new test restores the sink with `t.Cleanup(SetDurableSinkDir(""))`. This matches the cleanup the other tests use for the sink, and `fabriccli` has no `TestMain` sink setup it would need to restore to instead. Covered.
   - The scenario step keeps `RequireOK`, the weft SHA check and the warp SHA check. It still repoints the sink and still has its own cleanup, which is harmless because it no longer reads the sink. Covered.
   - The scenario step comment was updated to say the bypass branch points the sink at the hub it resolves from cwd. The new test's doc comment states the serial rationale. Covered.
   - The file header comment lists the verbs the file drives. The new test is not a verb scenario, but the header does not claim to list every test. No finding.
   - The new test carries the file's `tier1` tag, so it runs at the existing tier. Covered.

2. **Parent-Align merge a7dfe2ea.**
   - The parent range (burlerengine, commentlint, loomcli, loomshed, shedrecipe, one report) touches no package the settled scanner, recover-batch or reset changes live in. A `git diff --stat` of `internal/fabriccli`, `internal/fabricengine`, `internal/websterengine` and `internal/planparser` between the settled and current parent tips came back empty.
   - Nothing the parent brought in asserts or reads what the settled change altered. Covered.

3. **Rules and docs.**
   - The change is test-only, so no `doc.go`, `PATTERN.md` entry or pattern background file is forced. `PATTERN-test-speed` is met: the test sits in the existing file at the existing tier, and it is serial with a stated reason, as the golang-testing skill requires.
   - The commit message carries no `Pattern-Departure:` trailer, and none is needed.

**Findings:** none.

Tool calls (1):
1. One grep and sed over `internal/fabriccli/testmain_test.go`, `anchor_integration_test.go` and `internal/logger/sink.go`, plus a `git diff --stat` of the four packages across the parent range. My context lacked whether `TestMain` sets a sink and what `TraceDir` reads, and whether the parent range touched the settled packages.
