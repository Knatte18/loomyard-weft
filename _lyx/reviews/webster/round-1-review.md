---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/loomcli/circling.go:91
    summary: the hard-cap check on continue runs before the not-escalated and already-decided checks, so those cases at or past the cap get a hard-cap refusal that names the wrong way forward
    fix: in circlingVerb, run the hard-cap check after deps.record reports ErrNotEscalated and ErrCirclingDecided are ruled out, or look up whether the latest round carries an escalation record and an undecided state before refusing on the cap; keep the refusal text and add one row to the circling test table for a not-escalated round and one for a decided round at the cap, each expecting the existing not-escalated and already-recorded refusals.
  - id: F2
    severity: NIT
    class: consistency
    location: contracts/stencils/bouncer/bouncer-fragment-decide-capped.md:6
    summary: the capped decide fragment says the segment can only be accepted or handed on, but lists accept as its single option, so handed on names no verb
    fix: reword line 6 of the fragment to say the segment can only be accepted, matching the Awaiting Reason in escalationReason, which names accept alone.
---

### [LOW] Hard-cap check precedes the escalation preconditions

**Location:** `internal/loomcli/circling.go:91`
**Issue:** `circlingVerb` reads the latest round and refuses at or past the hard cap before `deps.record` runs.
A `continue` against a round that carries no escalation record, or one whose decision is already recorded, therefore gets the hard-cap refusal, whose way forward is to raise the cap and re-run `continue`.
That re-run then fails with the not-escalated or already-recorded refusal, so the first message sends the operator down a path that cannot succeed.
The hard-cap bound itself agrees with the Bouncer's: both refuse at `round >= HardCap` over the latest round, so there is no off-by-one.

### [NIT] Capped decide fragment mentions a hand-on with no verb

**Location:** `contracts/stencils/bouncer/bouncer-fragment-decide-capped.md:6`
**Issue:** The fragment reads "the segment can only be accepted or handed on" and then lists the accept verb alone.
The brief's closing paragraph already covers the case where the fork cannot settle the question, so the phrase adds a second, unnamed route.

## Checked and found sound

- The hard cap is compared against the round number (`atHardCap(round)`), never Shed's count; `settleUndecided` opens the soft-cap window on either, and tests pin a goto-reset count at the cap.
- `HardCap` 0 means no cap; `NewBouncer` refuses a negative one.
- Continues past the soft cap are budget-exempt through `cause == EscalationBudget || b.budgetReached(round)`, and the rule's automatic continue is exempt.
- `hard-cap` round-trips through the escalation record and the circling decision file via `EscalationCause.valid`.
- The decide fragment is chosen in Go by `atHardCap`, Go interpolates only the slug, and `escalationReason` names accept alone at the cap.
- Loom and darn refuse a hard cap below 1 and below the resolved soft cap, with the per-layer route from `CapRoute` for loom and the prime-only command for darn, whose keys are barred from the local layer.
- `RowHardMaxBounces` is set for the four loom Bouncer rows and for `Darn-Bouncer`.
- The refusal-spec rows land with the refusals.
- `lyx gate test` over the touched packages, `contracts/...` and `cmd/lyx` (which carries the test budget) is green.
