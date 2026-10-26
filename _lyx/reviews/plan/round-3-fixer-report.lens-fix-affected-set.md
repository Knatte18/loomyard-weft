```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set. I re-ran each set-shaped finding's search to check that the fix covers every member, not only the ones the review named.

Unchecked findings:
- **F2:** I didn't re-run its search. Its premise is a single paragraph in card 15, not a set of sites.
- **F5:** decision-class. It goes to the parent and the fixer didn't fix it.

Results per finding:
- **F1:** covered. The only sites that pin the shipped `prime` shortcut to `lyx orch start` are `cmd/lyx/shortcuts_test.go:124` and `internal/shortcutcli/shortcut_test.go:218`, and card 10 now lists both tests. `shortcut_test.go:98` uses a fake root, not the shipped file. The `prime` rows in `TestShortcut_Run` read `testShortcuts`, which is not the shipped file, so they stay with card 12.
- **F3:** covered. The callers of `NilSeams` beyond the four the review named are `wiring_beam_test.go:35` and `wiring_darn_test.go:50`. They only check the `BeamDescribe` prefix and `darnSeams`, so a nil `Pawl` seam doesn't fail them. Every registry name needs a row in `shedrecipe`'s `minimalEntries` (`conformance_test.go:106`), and card 16 already edits it.
- **F4:** covered. The other `landingshed.Deps` builders (`validLandingDeps` and `envkit.LandingDeps` callers) build fields by name, so the two new fields break none of them. The only enumerations are the reflective walk and `nilLegal`, and card 14 now lists both.
- **F6:** covered. `startPlainRun`, `killTreeProcs` and `tailString` are called only in `TestSmokeOrch_Adopt`. `turnEnds` and `waitForDiag` have other callers and correctly stay.
- **F7:** covered. Only card 6 still names `TestNewBeam` and `TestBeamRouting`, and that's correct because card 6 is where the rename happens.

no findings

Tool calls:
1. Searched for every site pinning the shipped `prime` shortcut; the context held only the review's two tests and grep output limited to lines 1–6x.
2. Searched for every `BeamDescribe` and `NilSeams` enumeration; the context had only `envkit.go` and the two `intentionallyNil` hits.
3. Read `wiring_beam_test.go`, `wiring_darn_test.go` and card 17's lines on seam coverage; the context didn't show how those `NilSeams` callers filter.
4. Searched `conformance_test.go`, `registry_test.go`, `recipe_test.go` and the `landingshed.Deps` builders; the context didn't show these possible enumerations.
5. Read the `minimalEntries` guard in `conformance_test.go` and card 16's coverage of it; I needed to confirm a new registry name fails without a row there.
