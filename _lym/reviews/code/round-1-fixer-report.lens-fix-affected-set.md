```yaml
lens: fix-affected-set
outcome: reported
```

Task: the fix-affected-set lens, covering F1, F3, F6 and F7. I found no missed member of any finding's set. Two LOW doc comments are left imprecise.

**F1 (Accepted flag):** no live `rec.Accepted`, `Accepted bool`, `Accepted:` or `accepted,omitempty` remains in the `FixerAttempts` code or tests. Every consumer (`finalize.go`, `fixaccept.go`, `fixaccept_test.go`, `finalize_gate_test.go`, `fixerattempts_test.go`) now uses `AcceptedHead`. The only caller of `AcceptFixReview`, `internal/loomcli/circling.go`, passes the new `taskHead` argument.

**F3 (spec halt rule):** `contracts/specs/loom-plan-spec.md:330` is reworded. No other doc or spec still says the landing rows halt only on a spent cap or an over-cap change. The remaining "over the caps" hits are accurate: `docs/overview.md:597`, `contracts/specs/landing-fix-review-spec.md:7` and `contracts/stencils/landing/landing-template-fix-review.md:17`. The `docs/research/**` hits are archived benchmark data, so I left them.

**F6 (Finalize reroute-cap examples):** two Finalize reroute-cap sites outside the review's list remain, and both are correct.
- `internal/landingshed/handback_test.go:30` ("RerouteCapAtFinalize") asserts `HandsBack` returns false for Finalize. That is the right post-change behaviour.
- `internal/shedverbs/status_test.go:495` is now the negative row.

**F7 (accept descriptions):** `docs/overview.md` lines 581 and 690 are updated. Three other sites describe `accept` only for review escalations, and I judge them correct:
- `internal/battenshed/innerrun.go:131` (`awaitingHandOff`) covers the `awaiting` state, and a Finalize halt is `blocked`.
- `internal/loomcli/arbiterprompt.go:13` covers the arbiter, which sees review escalations only.
- `contracts/stencils/orch/orch-template-role.md:67` covers the cap/hard-cap path.

**LOW: stale wording in two doc comments.** Neither is false, but the acceptance is now bound to a HEAD.
- `internal/landingshed/doc.go:96` says `AcceptFixReview` "marks the record accepted".
- `internal/landingshed/finalize.go:620` (the `gateAndFix` doc) says the resume "lands the change when the operator accepted it".

Both could say "accepted at the task HEAD". `judgeFixerChange`'s own doc and `fixaccept.go` already state the binding.

**Tool calls**
1. Grep for `Accepted` wiring plus F6 and F3 phrases across `internal`, `docs` and `contracts`: the three set searches for F1, F6 and F3, which my context lacked.
2. Grep for `accept` descriptions and `OnReroute` assertions: the F7 and F5 set searches, which my context lacked.
