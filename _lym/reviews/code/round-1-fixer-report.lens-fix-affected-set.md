```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches behind F1, F2, F3, F6 and F9 for the fix and check every member is covered.

Findings: none.

- **F1 (seam callers):**
  - Every user of `ReplacedDecisionRecord` takes the new `func(sinceHead string)` signature: `deps.go`, `judgesnapshot.go`, `driftwiring.go`, `envkit.go` and the test fixture.
  - `battencli/wire_seams_test.go` lists the seam by name only.
  - `doc.go` and the `Changes` and `appendedDecisionRecord` doc comments were updated.
- **F2 (logs literal):**
  - The only remaining `"logs"` literals in `driftengine` are the two constants in `paths.go`.
  - The other `"logs"` hits (`junctionnames.go`, `standalonegeom`, `logger`, `cleanupcli`) are separate directories outside the drift module and out of scope.
- **F3 and F6 (callers):** `groupRuledCells` and `cappedFindings` have one production caller each (`verify.go:364` and `verify.go:354`). Both go through the changed functions, so no site is missed.
- **F9 (`GuardHalt.At` consumers):** The consumers of `GuardHalt.At` are `changeTime` and `recordGuardHalt`, and both handle an empty `At`. `changedSince` only reads `.Changes`, which is unaffected.

Unchecked, by choice: the F5, F7, F8, F10 and F11 tests, because each is a coverage addition, not a set premise.

Tool calls:
1. One grep for `ReplacedDecisionRecord` and `appendedDecisionRecord(`, plus a grep for the `"logs"` and `verify-findings` literals, to rerun the F1 and F2 set searches. My context had only the edited files, not the repo-wide hits.
2. One grep for `groupRuledCells`, `cappedFindings` and `haltFindingsLimit` callers, plus a grep for `GuardHalt` consumers, to rerun the F3, F6 and F9 set searches. My context did not show the other files' usages.
