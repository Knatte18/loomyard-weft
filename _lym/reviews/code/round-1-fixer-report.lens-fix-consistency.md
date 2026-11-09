```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set (PATTERN exception bullet plus reworded battencli keep marker) for agreement with its surroundings.

**MEDIUM — `pattern/PATTERN-test-speed.md:183-184`, the new exception bullet.**
- The bullet makes `//testtiming:keep` the marker of a measured-time exemption.
- The PATTERN defines that marker differently at line 265. It is the redundancy-report marker, for a candidate that is kept because it pins something its covering tests do not.
- The rest of the file reads the marker the same way, at lines 117, 124-125, 142 and 165. It is a stacking marker above a test, not a timing exemption.
- Any test with `//testtiming:keep` would now read as exempt from the rule on measured times. For example, `contracts/stencils/*` tests carry it for coverage reasons only.
- Nothing mechanical ties the marker to a time-bound test, so the exception is unenforceable as written.
- A scoping reference sits at line 265 and a new exemption sits at line 184, so the two uses of one marker can drift apart.

**LOW — `internal/battencli/lifecycle_integration_test.go:313`, the reworded marker.**
- The text now says "under startCostBound". That matches the `startCostBound` comment at 1414-1416, but `startCostBound` is defined 1100 lines below the marker and the marker only names it.
- This is consistent with the code. Line 1417 still reads `2 * time.Second`.
- grep found no remaining "100 ms" or "D2" in `internal/battencli/*_test.go`, so the marker had no stale twin there.

Calls I made:
1. grep for "100 ms", "100ms" and "D2" in `internal/battencli/*_test.go`, and for `testtiming:keep` in `pattern/*.md`. The context did not hold sibling mentions of either.
2. Read `pattern/PATTERN-test-speed.md` lines 258-271 for the redundancy-report definition of the marker. The context held only lines 165-184.
