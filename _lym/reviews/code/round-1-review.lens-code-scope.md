```yaml
lens: code-scope
outcome: reported
```

My task: work the code-scope lens, mapping each change in the diff to a goal point of the decision record and each goal point to the changes that build it.

**Decisions** (decision record alone; it has no `Added after Discussion` entries):
- G1: A held decision finding gates until the parent rules.
- G2: A spent budget with BLOCKING findings open asks the parent.
- G3: A hardener round never approves with its focus undriven.

**Per change**
- C1. The Code-Review rows get `escalate_decisions` and `decisions_escalated` in the darn, tapestry and sampler recipes and in `testdata/tapestry-rows-before-motif.yaml` → G1. Sampler is not on G1's Where list, but its Code rows carry the same perch, so the goal's "no acceptance … lets a run past" covers them.
- C2. `blocking_escalates` on the Code-Bouncer rows, with `settleUndecided`, `blockingEscalationDetail` and `openBlockingKeys` in `bouncer.go` → G2.
- C2a. The `BouncerConfig`/`Bouncer` doc comments, `shedadapters/doc.go`, the `code_auto_accept` comment in `loomengine/template.yaml` (named in G2's Where), the escalation stencil's `budget` cause and arbiter line, and `shed-recipe-spec.md` → forced by C1 and C2.
- C3. `escalate_decisions` and `decisions_escalated` on the Harden rows of `hardener-recipe.yaml` → G3.
- C4. `FocusOrigin` and its exemption in `roundcheck.go`, the `burlerengine/doc.go` lines, the `origin: focus` rules in `burler-review-format.md`, and the focus-finding and Converged lines in `harden-rubric-review.md` → G3.
- C5. Tests: `roundcheck_test.go` focus rows, `TestBouncer_AutoAccept` new rows, `TestBouncer_SoftCapRule` path removal, `TestDecisionSwitches_SetOnTheRowsThatAskTheParent`, and the `entries_bouncer_test.go` row → forced by C1 to C4.
- C6. Commit `f2cc07498` re-breaks comments and fixes test ledgers → forced, a fix to C5.
- No change traces to no decision.

**Per decision**
- G1 → C1, C5. The held mark in `prompt.go` and `fixerreport.go` is unchanged, which is fine: it already keys on `DecisionsRouted`, which C1 now sets.
- G2 → C2, C2a, C5. It is only partly built; see F1.
- G3 → C3, C4, C5. `arbiterstop` and `shedarbiter`, named in G3's Where, are unchanged. That holds: `Eligibility` already admits cause `decision` (`internal/arbiterstop/arbiterstop.go`, the `StateAwaiting` case) and refuses everything else to the parent, so "arbiter rules first, then the parent" needs no change there.
- G3's APPROVED settle rule in `bouncer.go`/`burlerengine` gets no mechanical change. The undriven entry is made BLOCKING through stencils only (C4), which removes the Cause the record names. I file no scope finding on this; whether stencil-only enforcement is enough is outside my lens.

**Findings**

F1. BLOCKING, class `scope`.
- Location: `internal/shedadapters/bouncer.go#settleUndecided`. The gap is in the routing order of `settleUnlessRefused`: `routeDecisions` runs before `settleUndecided`.
- Summary: G2's decision ("a run whose code review budget is spent with a BLOCKING finding open stops and asks its parent") is not built for a round whose review also holds a decision-class finding. Such a round escalates with cause `decision`, which the arbiter may settle. An arbiter `accept` passes the round over the open BLOCKING findings at a spent budget without the parent: the accept's refusal check leaves out the decision class, and on a CONTINUE verdict it checks only dropped keys. The arbiter brief forbids accepting only over an open design-wrong finding, and the decision fragment says an accept passes the round over every listed BLOCKING finding.
- Evidence:
  - `internal/shedadapters/bouncer.go:1081` (`routeDecisions` taken before `settle`).
  - `internal/shedadapters/bouncerdecision.go`: `settleDecisionAccept` calls `acceptRound` when the refusal is empty; `decisionRefusal` and `refusalLeavingOut` check BLOCKING only for a CONVERGED verdict.
  - `internal/arbiterstop/arbiterstop.go`, `Eligibility`: cause `decision` is eligible.
  - `contracts/stencils/bouncer/bouncer-template-escalation.md:50`.
  - `contracts/stencils/bouncer/bouncer-fragment-decision.md:22`.

Tool calls:
1. Grepped `bouncer-template-escalation.md` and `bouncer-fragment-decision.md` for BLOCKING, budget and accept. My context held only the diff hunks of the first and not the second, and I needed to confirm what the arbiter is told about accepting open BLOCKING findings.
