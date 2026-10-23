# Card 37 — batten-restep-skips-landing-halts

**Create:**
- `plan:internal/battenshed#TestInnerRun_BinaryChangeSkipsALandingHalt` -> `func TestInnerRun_BinaryChangeSkipsALandingHalt(t *testing.T)`

**Edit:**
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#NewInnerRun`
- `internal/battenshed#innerRunProducer.binaryRestep`
- `internal/battenshed#restepHarness.producer`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#intentionallyNil`
- `internal/testkit/envkit#FullEnv`
- `internal/testkit/envkit#nilLegal`
- `internal/loomcli#intentionallyNil`

**Uses:**
- `internal/landingshed#HandsBack`

**Extends:**
- `internal/battenshed/doc.go`

**Intent:** Batten's binary re-step of a halted Go child, once per halt and per lyx build, skips the landing halts (D11), so a changed build never re-steps a Finalize reroute-cap halt, a Merge-In escalation or a Merge-In fixer-spent halt.
`InnerRunDeps` gains `HandBack func(status shedengine.Status) bool`, the hand-back check over the child's status; a nil check resolves to marking nothing in `NewInnerRun`.
`binaryRestep` skips a status the check marks, beside its blocked-or-failed test, and writes no marker, so the halt waits for the orch's `lyx loom resume`; every other halt keeps the re-step, so a fix deployed in a new build can still clear it.
battencli's wire fills the check with `landingshed.HandsBack` over the child status's current producer and error, the same predicate the Go driver's hand-back row and the LLM driver's mark read.
The test kit and both `intentionallyNil` maps follow the new seam.

`TestInnerRun_BinaryChangeSkipsALandingHalt` drives the re-step harness over one landing halt, a blocked Finalize reroute-cap status with a changed build, and asserts no spawn and no marker, and over a blocked Webster halt asserts the re-step as today;
one landing-halt row covers every kind, since the re-step reads only the told check's answer and card 34's `TestHandsBack` pins the predicate.

`doc.go`'s binary re-step paragraph names the skip.

**ImpactSummary:** One additive batten seam and one skip condition; battencli's wire is its only filler.
