```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency lens: checks whether every sibling the change set touches still agrees with it.

Findings:

1. MEDIUM. Changed text: decision-record.md D3. The union's second member now admits only test files the gate's host compiles. Its fourth member was left as it was: "every `//lym:guard` test of the tier the package declares".
   Summary: the platform filter covers only one of two union members that are read from source, so a guard test in a test file constrained to another GOOS keeps F2's zero-test route open.
   Evidence: the guard scan in `internal/impactset/guards.go:210,299` reads `//go:build` only for the tier and does not filter by GOOS. If such a guard test is named in the selected `-run`, it matches nothing on the host. `emptyTierFailure` then fails the tier, because `holdsTierTest` finds the package's other admitted tier tests. The D6 sentence the fix added puts that `-run` in the run's args, which keeps the check active. No guard test in this repository sits in a GOOS-constrained file today: the three `cmd/lym` hits are unconstrained files. So the gap is in the record's rule, not a present failure. The fix is to apply the same host-admission clause to the guard member, or state it once for the whole union.

2. LOW. Changed text: support-log.md Round 3 and Q10. The history entries were left as they were: Turn 3 says "interactions 2 and existing-paths 2 (merged): … Fixed: D2 adds covered-package drift to testmap's staleness; the cost is a question to the parent in Open risks", and "board-scope 1 (decision): D2 ahead of `testmap-upkeep`. Kept as a question for the parent in Open risks."
   Summary: these turn-history lines now describe a D2 rule and an Open risks question that the record no longer holds, and nothing points forward to the round-3 reversal.
   Evidence: D2's staleness sentence and the first Open risks bullet were deleted. Round 3's F1 line records the reversal, but Turn 3 still reads "Fixed". A reader would notice this, though it is a history log.

Checked and consistent:
- D6's new sentence agrees with the green record. `greenEntry.Args` is `run.args` (`internal/gatecli/green.go:447`), so "the green entry carries its `-run` arguments" still holds once the arguments sit in the run's own args.
- An entry with arguments matches only exactly (`green.go:228`) and never serves as a base (`green.go:206`).
- D2's "the Bound below" points to the D2 Bound placed after Rationale.
- The new D2 criterion, the replaced stale criterion, the Auto-mode line and both Open risks bullets agree with D2.
- D5's predeclared miss bullet agrees with the Rationale qualifier and Q6. The removed dot-import clause is covered by the new bullet.
- The D5 embed sentence agrees with the new D5 criterion.

Tool calls:
- Grepped `internal/gatecli/green.go` for how entries record args, and `internal/impactset` for build-constraint reading. My context had neither.
- Grepped for `//lym:guard` in test files with a platform `//go:build`, to see whether the guard twin is hit in practice. Not in context.
- Read the head and guard lines of the three `cmd/lym` hits, to tell fixture text from real constraints. Not in context.
