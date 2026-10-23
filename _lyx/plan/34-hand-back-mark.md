# Card 34 — hand-back-mark

**Create:**
- `internal/landingshed#HandsBack`
- `internal/landingshed#TestHandsBack`

**Edit:**
- `internal/shedverbs#Spec`
- `internal/shedverbs#loopObject`
- `internal/shedverbs#loopObject.fields`
- `internal/shedverbs#runLoop`
- `internal/shedverbs#statusCmd`
- `internal/shedverbs#TestLoop_RunsUntilStopAndPrintsOneEnvelope`
- `internal/shedverbs#TestStatusCmd_FoundEnvelope_GenericCore`
- `internal/loomcli#loomCLI.specFor`

**Uses:**
- `internal/landingshed#LandingStopPrefix`
- `internal/shedengine#ReasonRerouteCapReached`

**Extends:**
- `internal/shedverbs/doc.go`
- `internal/landingshed/doc.go`
- `contracts/specs/loom-status-spec.md`

**Intent:** Mark the landing halts in what every driver reads, so all hand them back to the orch (D11).
`HandsBack(producer, reason)` is the one producer-and-prefix predicate: a blocked stop at Merge-In or Finalize whose reason starts with `LandingStopPrefix`, the halts the fixer and the caps raise, or a blocked stop at Finalize, the only landing row with a reroute edge, whose reason starts with `shedengine.ReasonRerouteCapReached`.
Shed writes the reroute-cap prefix for every row, so the producer match keeps Webster's plan-conflict reroute-cap halt out, and it is routed as today.
`shedverbs.Spec` gains `HandBack func(producer, reason string) bool`, told by the recipe's owner as `GotoGuard` is, so `shedverbs` stays recipe-generic; nil marks nothing.
The loop's stop envelope gains `loop.hand_back`, true when the stop is blocked and the told check answers true for the status's current producer and error; `lyx shed status` gains the same top-level `hand_back` key over the status file.
loomcli's `specFor` tells `landingshed.HandsBack` on loom and darn runs.
The Go driver's hand-back row, batten's re-step and the LLM driver read this same predicate or its mark in the following cards, so all three agree.

`TestHandsBack` pins a landing-stop reason at Merge-In and at Finalize, the undecodable-attempt-record halt at Merge-In and at Finalize, the lock re-check halt at Finalize, a reroute-cap reason at Finalize, a reroute-cap reason at Webster, a landing-stop prefix at another row, and an unrelated reason at Finalize.
`TestStatusCmd_FoundEnvelope_GenericCore` gains rows pinning the `hand_back` key true for a blocked Finalize reroute-cap status under the told check, false for a Webster one and false with no check told; the loop test asserts the envelope key.

`loom-status-spec.md` documents the key, and both `doc.go` files name the predicate and the mark.

**ImpactSummary:** One additive Spec seam and one envelope and status key; loomcli is the only owner that tells the seam.
