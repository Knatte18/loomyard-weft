# Card 5 — webster-row-auto-rebaseline-stuck

**Edit:**
- `internal/shedadapters#WebsterProducer.Call`
- `internal/shedadapters/webster_test.go`
- `internal/shedadapters/doc.go`

**Uses:**
- `plan:internal/websterengine#ErrAutoRebaseline`

**Intent:** Make the loom's Webster producer the one caller that sets `RunOptions.AutoRebaseline`, and map the auto-rebaseline's errors to Stuck (D1).
`Call` runs the seam with `Fresh: false` and `AutoRebaseline: true`, and an error matching `ErrAutoRebaseline`, the refusal sentinels and a transient read or restamp failure alike, maps to `shedengine.Stuck` with the error text as the reason, never to a returned error, so the run blocks resumable instead of failing with `producer-hard-failure`; card 4 makes every such text name re-stepping the row, a refusal through the rebaseline's step-keyed text and a save, restamp or read failure through the re-step clause, as `ErrPendingAuditFindings` does today.
`Fresh` stays fixed false for the reason the method's comment gives.
`doc.go`'s Webster bullet states the two options the adapter sets and the new Stuck mapping.
`TestWebsterProducer_OtherEngineErrors` gains the row that an `ErrAutoRebaseline` wrap is Stuck with the text as its reason, and `TestWebsterProducer_OutcomeDone` asserts through the fake runner that the options it received carry `AutoRebaseline` true and `Fresh` false.

**ImpactSummary:** One adapter method and its test; loomshed's lazy wrapper passes the outcome through unchanged.
