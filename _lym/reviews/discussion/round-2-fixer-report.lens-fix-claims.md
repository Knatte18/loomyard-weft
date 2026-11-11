```yaml
lens: fix-claims
outcome: reported
```

I checked every claim the fix added or rewrote against the code at HEAD. Three findings:

1. **MEDIUM.** D4's copy bullet: "The copy joins the outputs the review row archives on a failed exit." The matching D4 criterion is "…copied beside the review file once the round's review is accepted, and a failed review exit archives that copy."
   - Summary: the copy now happens only after acceptance, and no failed exit runs after acceptance. So a test cannot reach the archive half of this claim and its criterion.
   - Evidence: `internal/shedadapters/burlerreview.go:441-448`. `acceptedExit` calls `archiveRound` only when `writeAcceptance` itself fails, which is before acceptance. Every `failureExit` call (`:298-374`) comes before `acceptedExit`. The hand-on path (`:257-259`) archives nothing.

2. **LOW.** D4's new refusal: "run by the run's parent orch, which the refusal names as the verb's guard resolves it". The D4 criterion says "each refusal naming `lym loom decision add` and the run's resolved parent orch".
   - Summary: the guard does not always resolve a parent orch to name, so that criterion cannot hold for every refusal.
   - Evidence: `internal/loomcli/decision.go:685-690` covers a resolution error, and the way forward there points at the operator's unnamed shell. `:697-698` covers `parent.Name == ""`, where the call is "the operator's to record".

3. **LOW.** D7's and D5's new text: "It runs when the chair reaches done with the review written", and "first sets an ask of the round still `pending-post` or `open` to `timeout`".
   - Summary: entries run in list order at each arrival, and the first failure stops the evaluation. The claim holds only if the coverage entry comes after the review-parse entry, and the record says only "beside".
   - Evidence: `internal/shuttleengine/gate.go:99-101` (an ordered list consulted at each arrival), `:111` (a failed entry stops the evaluation), `:118` (later entries are not reached) and `internal/shuttleengine/doc.go:729-730`. Placed first, the entry would run at an arrival before the review exists and close the ask before its deadline.

Calls made beyond my context:
- Read `internal/shuttleengine/gate.go:1-135` for the gate entry semantics (cap, must-pass, order), which my context lacked.
- Grepped `internal/shuttleengine/doc.go` for "arrival" to find when a gate is consulted, which my context lacked.
- Read `internal/shuttleengine/doc.go:700-769` for evaluation order and the finalize rules, which my context lacked.
- Grepped `internal/shedadapters/burlerreview.go` for failed exits and the acceptance write, to place acceptance against the archive; my context held only lines 230-289.
- Read `internal/shedadapters/burlerreview.go:376-455` for `acceptedExit`'s archive behaviour, which my context lacked.
