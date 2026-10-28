```yaml
lens: fix-claims
outcome: reported
```

Lens: fix-claims. I checked every sentence the fix added or rewrote in `_lym/discussion/decision-record.md` against the code at HEAD.

Claims that hold:
- **D2, darn early return:** the darn caller keeps the history check only on a clean record, as `internal/loomshed/gates.go:140-143` already does.
- **D9, two builders:** there are two `PlanContext` builders, `internal/loomcli/validate.go:272` and `internal/shedrecipe/entries_gate.go:126` (`planContextOf`).
- **D9, `RefuseRaw`:** `RefuseRaw` is a value read from `gate.yaml` (`plancontext.go:40-42`, `validate.go:264-267`), so modelling the tier tags on it holds.
- **D7, tier keys:** in `pattern/PATTERN-test-speed.md:7-16` the `tierN` key and the build tag are the same name, so "on a Go hub" holds as a PATTERN rule.
- **D8, declaring file:** quarry symbols carry a declaring `File`, which `internal/planglyph/callercoverage.go:237` and `donecheck.go:193` already read. A member glyph can therefore be classified by the files declaring it.
- **D1, D3, D5, D6, D10, D16 and the acceptance bullets:** the text added for F7, F8, F9, F10, F12, F13, F14, F15 and F16 makes no code claim beyond what the review already established and the fixer re-checked (`envkit_test.go:17`, the chair include on line 8).

Findings:

- **LOW**
  - **Text:** D9, "and the plan gate's index for D8" (`PlanContext` carries the index).
  - **Summary:** this puts the index into `PlanContext`, but the loomshed plan validators already take the index as their own argument. The index would then exist twice, and the two copies could differ.
  - **Evidence:** `internal/loomcli/validate.go:285` calls `loomshed.ValidateReworkPlan(plan, c.env.WorktreeRoot, c.env.PlanIndex, ..., pc)`, which passes `PlanIndex` and `pc` separately. `planContextFindings(plan, cards, worktreeRoot, pc)` (`plancontext.go:55`) is called from inside those validators, where the index is already in scope.
  - **Suggested change:** "`planContextFindings` gains the index its callers already hold" instead of a `PlanContext` field.

Tool calls beyond context:
1. Read `validate.go:260-285` and `entries_gate.go:115-140` to check D9's claims about the two builders and `RefuseRaw`. The fixer's context had only the line numbers the review cited.
2. Grepped for quarry's `ResolveResult` and failed: the type is in an external module.
3. Grepped `internal/planglyph` for symbol `File` fields to check D8's claim that quarry reports the declaring file.
4. Read `pattern/PATTERN-test-speed.md`'s tier lines to check D7's "as PATTERN-test-speed's tierN rule requires".
