```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches for the fixer's set-premise findings (F1, F4, F6, F7, F8, F9, F15, F17) and check the fix covers every member.

Findings: none that a gate or reader would trip on. Two items are LOW and are only noted.

- **LOW** — `internal/fabricengine/commit_integration_test.go:265,268`: the failure messages still say `snapshotWarpSHA()`. F6's premise was the name in `doc.go`, which is covered. The oracle `SnapshotWarpSHAForTest` is still called here, so the messages name a symbol that no longer exists. Cosmetic.
- **LOW** — `docs/benchmarks/test-redundancy.md:716-717`: it lists `TestSnapshotWarpSHA_*` keep markers for tests that are now deleted or renamed. These are measurement reports, which the prose skill exempts from staying current. Noted only.

Per-finding results:
- **F1:** no `internal/shedcheck`, `shedcheck.` or `shedbuild.Check` or `Load` reference is left in `.go`, `.md` or `.yaml` outside `docs/benchmarks` (measurement reports) and `_lym`.
- **F4:** the only remaining `PushAnchored` is a row name in `docs/research/speed/fabricengine-per-test-times-2026-11-07.md:144`. That file is a dated timing report.
- **F15:** no `TestRunCLI*` or `RunCLIIn` is left in `internal` or `cmd`.
- **F6:** `doc.go` is clean. The only remaining `snapshotWarpSHA` strings are the two test messages above and the benchmark report.
- **F7:** the only `TestSnapshotWarpSHA_` mentions left are in `docs/benchmarks/test-redundancy.md`.
- **F8, F9, F17:** the targeted strings no longer appear. The `AppendDecision` hits are the test helper in `append_test.go` and its callers, which is correct. The `planglyph.ValidateFormat` hit is a comment in `loomshed/gatefindings_test.go:83`, outside F9's `docs/overview.md` location. It describes a planglyph gate, but the package-level function it names exists only in test code.
- **F10:** not rerun. Its set is the review's named list, and each marker was added.

Tool calls:
1. Combined grep of F1, F4, F15 and F6 over `.go`, `.md` and `.yaml` outside `docs/benchmarks`, `docs/research` and `_lym`, to rerun each finding's own search.
2. Grep of F8, F9, F17 and F7 strings, to confirm no stale name remains; its output was truncated by size, but the visible head was enough.
