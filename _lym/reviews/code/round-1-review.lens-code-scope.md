```yaml
lens: code-scope
outcome: reported
```

Lens code-scope: tracing each change in the diff to the decision record's goal points and its Added after Discussion entry.

**Decisions (decision record alone)**
- G1, Idea 1: find the hardenshed test that writes a record outside `t.TempDir()` and point it there. The Added after Discussion entry drops it.
- G2, Idea 2: a gate check that fails on any new untracked file in the source tree after a test run.
- A1, "No hardenshed test fix; stray-file gate check instead": drop G1 and build only the gate check that fails a tier run leaving new untracked, non-ignored files.

**Changes → decision**
- C1, `internal/gatecli/strayfiles.go` (new): `untrackedWorktreeFiles`, `checkStrayFiles`, `strayFilesLine`, `strayFilesListed`. Traces to G2/A1.
- C2, `internal/gatecli/tierrun.go`: `runTier` becomes a wrapper that lists before and after the run and calls the check. The old body is renamed `runTierCommands` and its copies sentence moves into `runTier`'s doc. Traces to G2/A1. The split is needed so a `--copies` run is listed once around all copies, not once per copy.
- C3, `internal/gatecli/copies.go`: the copy loop calls `runTierCommands` instead of `runTier`. Forced by C2: without it each copy would recurse into the wrapper and take its own listings.
- C4, `internal/gatecli/doc.go`: new `# Stray files` section. Forced by C1/C2, as package docs describing the new behaviour.
- C5, `internal/gatecli/strayfiles_integration_test.go` (new): `TestRunTierStrayFiles`. Forced by C1/C2, as the test of the new behaviour.

**Decision → changes**
- G1: no change, and correctly so. A1 drops it, and the focus says a missing hardenshed change is no finding.
- G2/A1: C1 and C2, with C3–C5 forced by them.

**Findings**

None. Every hunk traces to A1 or is forced by a change that does, and A1 has a change that builds it. The diff touches no file outside `internal/gatecli`. The commits in the range carry no `Pattern-Departure:` trailer, and none is needed. A1 covers dropping G1 (decision-record.md, "Added after Discussion (parent, 2026-11-19)"), and the summary and commit 54938b72 say the same.

Tool calls: none.
