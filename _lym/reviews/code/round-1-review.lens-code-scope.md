```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. It traces each change in the diff to the board entry's goal points and checks that every goal point has a change that builds it.

**Decisions.** The answer key is the decision record alone. Its `## Goal` has three "What to do" points, and the record has no `Added after Discussion` entries.
- **G1:** find out why the shuttle state check reported `loop=held state=busy cause=background` repeatedly, and whether that pair counts as a real disagreement.
- **G2:** make the loop record its exit, or at least a reason, on every path that ends it without an envelope, so a halt note can name the cause.
- **G3:** have the repeated warning escalate or resolve itself instead of recurring silently.

**Changes and the decision each traces to:**
- **C1:** `loopAgreesWithState` now counts held beside busy on background work as agreement.
  - Code: `internal/shuttleengine/waitshadow.go:100`.
  - Doc: `waitshadow.go:86-91` and `internal/shuttleengine/doc.go:1029-1032`.
  - Test row: `waitshadow_test.go:185-193`.
  - Traces to G1, which settles the pair as no real disagreement (commit `bcb4cc28`), and to G3, since the warning for that pair stops recurring.
- **C2:** `endWithoutEnvelope` marks the loop log on the early exits: directory, lock, refusal and pid write.
  - The same change marks a failed envelope write.
  - Code: `internal/shedverbs/loop.go:212-247, 356-358, 539-546`.
  - Doc: `shedverbs/doc.go:224-226`.
  - Test: `TestLoop_EndWithoutAnEnvelopeIsMarkedInTheLoopLog`.
  - Traces to G2.
- **C3:** the pid record gains `StartedAt`, and `proc.BootTime` is new on Linux and Windows.
  - `LoopSpec.BootTime` is the seam, and `unmarkedExitCause` names the cause.
  - `loopExitEvidence` takes the `LoopSpec` and the start, and `deadLoopStop` and `loopExitedEarlyStop` were updated as its callers.
  - Docs: `shedverbs/doc.go:276-280` and `proc/doc.go`.
  - Tests: the three new rows in `loopwait_test.go` and `TestBootTime`.
  - Traces to G2. This is the "so a halt note can name the cause" half, for the issue's own case, an exit no process marked.
- **C4:** the panic tests gain a clock that counts reads.
  - Locations: `loop_test.go:545-553` and `loop_unix_integration_test.go:137-145`.
  - The integration tests' `loopExitEvidence` call sites were updated to the new signature.
  - Forced by C2/C3: the new `now()` read at the pid write, and the signature change.
- **C5:** commits `34c57816`, `1bfa2d57` and `f49c022b` only re-break comment lines in the hunks above.
  - Forced by C1–C3, to meet the comment line-break lint.

**Decisions and the changes that build them:**
- G1: C1.
- G2: C2 and C3, with C4 and C5 forced by them.
- G3: C1.
  - It resolves the one repeated warning the issue names by removing the false pair.
  - It adds no escalation for other pairs, and the goal asks for "escalate **or** self-resolve" of "the repeated warning", so self-resolving that warning builds it.

**Findings:** none. Every change traces to a goal point or is forced by one that does, and every goal point has a change that builds it.

Tool calls: none.
