```yaml
lens: code-scope
outcome: reported
```

Lens: code-scope. I traced each change in the diff to the issue #206 goal points under `## Goal`. The record has no `Added after Discussion` entries.

**Decisions** (the goal points of the board entry's "Expected")
- G1: the ask resolves the orch's current pane.
- G2: a parent can answer, or record a satisfied verdict for, the head that a no-parent ask named.

**Per change**
- C1, `internal/burlerengine/parentreview.go`: `AnswerParentAsk` becomes `AnswerParentAskOrUnreached`. It admits a latest, uncleared `no-parent` or `undelivered` ask under the caller's tag and skips the deadline for it. Traces to **G2**.
  - Admitting `undelivered` is within G2. Before the merge base 708cdadc5, a post creel refused was recorded as `no-parent`; since #200 the same case is `undelivered`. So the issue's "no-parent" asks are today's `undelivered` asks.
- C2, `internal/loomcli/parentreview.go`: `parentReviewAnswerVerb` passes `landingshed.PRReviewAskLineTag`, and its doc comment and help text change. Traces to **G2**.
- C3, `internal/burlerengine/doc.go` and `internal/loomcli/doc.go`: the package docs describe the new move out of an end state. Forced by C1 and C2.
- C4: every existing test call site moves to the new name with `""`. That covers driftengine `judgeparent_test.go` and `verify_test.go`, landingshed `prreviewask_test.go`, loomcli `commitrecords_test.go` and `parentreview_test.go` await, shedadapters `bouncer_parent_test.go` and `parentseat_test.go`, and burlerengine's other tests. Forced by C1's rename, after b696b4cca dropped the wrapper.
- C5: new rows in `TestAnswerParentAsk` and `TestParentReviewAnswerVerb`. Traces to **G2** as its tests.
- C6, `contracts/stencils/landing/landing-block-pr-review-autonomous.md`: the session runs `await` on the next mail before asking again. Traces to **G2**. Without it, the session's re-ask would make the parent's answer to the ended ask not-latest, and the answer would be lost.
- C7, `contracts/orchrole/role-autonomy.md`: the orch is told to answer an ask that ended no-parent or undelivered. Traces to **G2**.

**Per decision**
- G2 is built by C1, C2, C5, C6 and C7, with C3 and C4 forced by them.
- G1 is built by no change, and the change description argues no change is needed.
  - The evidence I read supports that. The PR-Review row's own ask (`internal/loomcli/landingdeps.go:514`) and the other askers all resolve the parent through the same `orchcli.ParentStrand`, whose liveness check is `OrchLive` → `orchroute.Live` → `reedengine.Engine.Status`.
  - `Status` builds liveness from `presentPanes(st.Strands, live, windows)` across the session's windows (`internal/reedengine/lifecycle.go:1310-1314`).
  - The issue itself reports that the row's asks were delivered while only the session's asks ended no-parent. A failure to resolve the pane would have hit both, since they share the resolver. That is the #200 signing defect, which 708cdadc5's message describes and which is fixed at the merge base.
  - So G1's premise does not hold at the base, and the missing change is no unlicensed omission. No finding.

**Findings**

None from this lens. Every change traces to G2 or is forced by one that does, and G1 is already met at the merge base on the evidence above. No commit in the range carries a `Pattern-Departure:` trailer, and none is needed.

Tool calls:
1. A grep in `internal/loomcli` for `askPR` and `ParentStrand(`, to confirm whether the session's ask and the row's ask share one parent resolver. My context held the row's resolver line but not the session ask's wiring.
