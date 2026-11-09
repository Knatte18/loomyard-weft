```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1 PATTERN exception, F2 marker reword).

**MEDIUM — `pattern/PATTERN-test-speed.md:184`: the new exception ties timing licence to a marker that means something else.**
- `//testtiming:keep <reason>` is the redundancy-report marker (line 265). It says a candidate pins what its covering tests do not, and the report lists it under "Kept". It says nothing about timing.
- Most of the keeps in the repo are on tests that measure no time (the stencils, `envkit`, `fabriccli` and `loomshed` ones from the grep).
- The exception says a cost-budget test "carries a `//testtiming:keep` marker". That reads as if the marker were what permits a measured-time bound. A reviewer cannot tell a licensed time bound from an ordinary keep, and an unrelated keep appears to license one.
- The test that is excepted is `TestBattenIntegration_Rows`. It carries the marker only because it pins an assertion. No scan ties the marker to a time comparison.
- Concrete breakage: any test with a keep marker can cite line 184 to justify comparing a measured time to a bound.
- The review's own fix text proposed the marker link. Dropping the marker clause would remove the conflation: "Exception: a cost budget that no outcome can stand in for may compare a measured time to a bound far above what load adds to the work it measures."

**LOW — `lifecycle_integration_test.go:313` (the reworded marker): the "fails" claim is stronger than the bound.**
- The marker says a row that waits seconds "fails". `startCostBound` is 2 s, and the PATTERN rule says load can reach such bounds. A row that waits 1.5 s passes.
- The `startCostBound` comment says "catches a row that waits seconds". The marker reads as a firmer guarantee than that.
- The review's own fix text also allowed the "purpose" wording, so this is only a nit on the exact phrasing. The marker is also long and now repeats the constant's comment (`:1414-1416`).

**No finding on the 2 s bound itself.** The review accepted it and the fix did not change it.

Tool calls:
1. Grep of `testtiming:keep` in `pattern/PATTERN-test-speed.md`. I needed the marker's defined meaning, which my context lacked.
