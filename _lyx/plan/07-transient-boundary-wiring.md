# Card 7 — wire the classifier at the producer and bootstrap boundaries

**Edit:**
- `internal/shedbuild/newshed.go`
- `internal/shedbuild/newshed_test.go`
- `internal/shedbuild/seam_enforcement_test.go`
- `internal/loomcli/arm.go`
- `internal/loomcli/step_test.go`
- `internal/battencli/arm.go`
- `internal/battencli/step_test.go`

**Prosa:**
- `CONSTRAINTS.md`

**Uses:**
- `internal/shedtransient#Class`
- `internal/shedtransient#Mark`
- `internal/shedengine#TransientOf`

**Intent:**
Turn the mark on for real (decision record D1, D2, D12): every recipe shed classifies its producer errors, and every step bootstrap classifies its own failures, through the one `internal/shedtransient` mapping.

Producer boundary: `shedbuild.NewShed` sets the assembled `Shed`'s `Transient` field to `shedtransient.Class`.
`NewShed` is the single assembler both recipe sheds (`loomrecipe.New`, `battenrecipe`) go through, so no recipe package changes.
`shedbuild`'s import allowlist (`seam_enforcement_test.go`) admits `internal/shedtransient`, with a one-line reason beside the entry.
Which producer failures reach this boundary as errors at all is card 8's concern.

Bootstrap boundary: `loomPreStep` and `battenPreStep` pass each error they return through `shedtransient.Mark`, leaving every kind unchanged.
The busy refusal is never transient by construction, since `Mark` leaves an unclassified error untouched.
No current bootstrap sub-step makes a remote call:
loom's seed-and-commit stage commits through `fabricengine.CommitAnchoredPaths`, which never pushes, and neither hook's lock probe, status seed or status-strand step reaches a remote.
So today the bootstrap mark is D1 and D12's bootstrap boundary held in place for a future remote sub-step, not a live path, and the card's doc comments on both hooks say so.

`CONSTRAINTS.md` gains a **Transient Stop Invariant** section after the Shed Verb-Set Invariant, stating:
- the mark is declared only in `internal/shedengine`, and its class set is closed at the declaration (the set is named there, not restated here);
- a lower-level package exposes its own classification, and `internal/shedtransient` alone translates it into the mark;
- the mark is set only at a producer boundary (the `Shed.Transient` classifier `shedbuild.NewShed` tells) or a step bootstrap boundary (a `PreStep` hook);
- a producer returns a failure `shedtransient.Class` classifies as a hard error rather than a verdict, so it reaches that classifier;
- a gate verdict (`blocked`, `awaiting`, `paused`) never carries it;
- rationale: a mark added at a new site changes what the ly-drive driver re-steps without a human deciding.

Tests:
- `newshed_test.go`: the assembled shed's `Transient` is non-nil and classifies an error wrapping `shuttleengine.ErrNotStarted` as `agent-start` and a plain error as empty.
- `internal/battencli/step_test.go`: `TestBattenPreStep_RunLockHeld_KindBusy`, `TestBattenPreStep_DecodeFailure_KindUnseeded` and `TestBattenPreStep_DoneSlug_KindBootstrap` also assert that `shedengine.TransientOf` of the returned error is empty, so the `Mark` wiring runs under test on every refusal arm.
- `internal/loomcli/step_test.go`: `TestStepCmd_BusyRefusal_BeforeBootstrap`, which drives the real `loomPreStep` through `stepCmd`, also asserts `"transient":""` on the envelope.

**ImpactSummary:** Every recipe shed and both step bootstraps now classify their errors; since no bootstrap sub-step reaches a remote, only producer errors can carry a class today, and only the persisted `transient` field and the envelope key change, never a kind, a state or an error text.

**Commit:** 7: shed: classify producer and bootstrap failures as transient
