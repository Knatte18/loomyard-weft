<!-- This is the decision detail of the Bouncer's escalation brief for cause `decision`: the questions only the run's parent can settle, the BLOCKING findings the round's review holds, the refusal the recorded answer would meet, and the verbs that record it.
     renderDecisionDetail fills it and passes the result as the brief's cause_detail marker, after the cause list.
     Every marker below is a top-level {{.X}} substitution: slug, decision_findings, blocking_findings, refusal and decide_verbs, each Go-filled and non-empty.
lyx-stencil: sha256=805611b975e618d5f4bd3b8536c5e0461c9d63b777637e748947897a9fd9477f build=5e449651fc9099cf3c0ec513ef579c464138a703 time=2026-10-25T09:35:58Z -->

### Decision findings

Each finding below is a question only the run's parent can settle.
Put each to the operator, unless a recorded operator decision answers it.

{{.decision_findings}}

### BLOCKING findings of the round's review

{{.blocking_findings}}

### Refusal

{{.refusal}}

When a refusal is named, the round is re-judged before the recorded answer acts while budget is left.
An `accept` passes the round over every BLOCKING finding listed once a judgment is admitted, and at or past the hard cap halts the run if the refusal still holds at the budget.
A `continue` runs another round whose fresh reviewers re-review the artifact, so a listed BLOCKING finding reaches that round's fixer only if they raise it again.

### Way forward

Record the answer with `lyx loom decision add {{.slug}}`, then record the segment's outcome with exactly one of:

{{.decide_verbs}}

Then resume the run with `lyx loom resume`.
