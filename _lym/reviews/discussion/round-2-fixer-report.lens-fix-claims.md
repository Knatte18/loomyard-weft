```yaml
lens: fix-claims
outcome: reported
```

fix-claims: I checked each claim the F12 fix added or rewrote against the design and the code at HEAD.

**Findings**

1. MEDIUM. The claim is wrong when the reroute cap halts the send-back. It concerns the new D4 Bound line "at most `pr_review_small_max` findings are fixed in place between two send-backs, or before the first", together with the new rule "Only the row's clean-tree reroute clears that count".
   - What breaks it: a send-back that reaches the reroute cap still runs the clean-tree clear first, so the count drops to zero. Shed then turns the reroute into the `reroute_capped` halt, so no send-back happens.
   - The resume that the cap's way forward names re-steps PR-Review with a count of zero. That opens a fresh allowance of in-place fixes with no send-back between the two windows, so "between two send-backs" does not hold.
   - Evidence: `internal/shedengine/doc.go:112-116` says a reroute past `rerouteCap` "halts blocked at the row with a budget-exempt Stuck entry marked reroute_capped", and that resuming "grants a fresh allowance". The review's Rejected list (bound-breaker F-D) says "at the reroute cap the row has already cleared the mark before shed halts it". The new D4 text clears the count in that same step.
   - Possible fixes: have the Bound say the count also clears at a cap-halted reroute, or have the row keep the count when shed caps the reroute.

**Claims that hold**
- "A re-step, a crash-resume and the resume after the dirty-tree Stuck keep it." The code has no path that clears pull-request record fields. The only write is `ParentReviewAsked = true` at `internal/landingshed/prreview.go:366`, so neither a resume nor a re-step resets the record.
- The `Call` doc comment at `internal/landingshed/prreview.go:63-69` confirms that the driver re-enters `Call` on a re-step.

Further tool calls:
- One grep in `internal/landingshed` and `internal/loom*` for writes that clear `ParentReviewAsked` or the session id. My context did not show whether a resume or re-step clears pull-request record fields, which the "keep it" claim depends on.
