---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D9 Bound, Open risks bullet 1, Acceptance criteria (Repeated refusals)
    summary: D9 says repeated refusals reach the segment's budget escalation to the parent, but a degraded Stuck reaches Shed's generic budget-exhausted block instead; the Bouncer escalates only from a CONTINUE or CIRCLING settle
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D8
    summary: D8 does not say whether a CONTINUE or CIRCLING verdict already on disk at Call entry is key-checked; if not, a lossy ledger settles unchecked and becomes round N+1's baseline, losing the dropped key for good, and D8 states no bound for that path
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D7, last line before Rationale
    summary: D7 describes the unchecked on-disk path as a verdict settled by an earlier Call, but any unattached CONVERGED on disk at Call entry, including one an earlier crashed Call never harvested, is cleared and re-seeded; D7 should state that bound, that the path lets nothing through
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Acceptance criteria vs D2
    summary: No acceptance criterion pins the first step of D2's seed order, a per-segment seed key outranking a set run-wide seed; every seed criterion sets only one seed key
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D9 Rationale
    summary: D9's rationale argues against the rejected override-to-CONTINUE alternative, which the support log's Rejected alternatives already holds; only its requirement that the judge stays the sole writer of its three files belongs in the Decision
---

### [MEDIUM] D9's budget bound names the wrong exit

**Location:** `_lyx/discussion/decision-record.md` D9 `Bound:`, the first Open risks bullet, and the acceptance criterion "Repeated refusals reach the segment's budget escalation and never loop."

**Issue:** D9 says a judge that keeps producing refused judgments "reaches the segment's `review_max_bounces` budget escalation to the parent".
In the code it routes through, that is false.
The refusal returns a degraded Stuck, as `retireLegacyVerdict` does (`internal/shedadapters/bouncer.go:562`).
The Bouncer's budget escalation (`settleUndecided` → `escalate(round, EscalationBudget, ...)`, `bouncer.go:702-710`) fires only from a CONTINUE or CIRCLING settle.
The `Bounces` field doc says the same (`bouncer.go:127-131`): "a CONTINUE or CIRCLING verdict then escalates to the parent instead of reaching Shed's generic budget block".
The `NewBouncer` budget rule says "Every degrade exit ... [is] counted here", and "a judge that never recovers halts within MaxBounces counted Stucks" (`bouncer.go:171-173`).
So repeated refusals end in Shed's budget-exhausted block (`shedengine.ReasonBounceBudgetExhausted`), a blocked run, not an Awaiting escalation to the parent.
The "cannot loop" half of the bound holds; the named outcome does not.
The first Open risks bullet ("escalations to the parent rise") and the acceptance criterion repeat the false premise, so the plan writer would test for an escalation that never happens or build an unrequested route to one.
The same Open risks bullet also pins "A Sonnet judge", though D1 makes the judge model configurable per segment.

**Fix:** Decide which exit the refusal takes and state it in D9's Bound.
To keep today's route, the bound is "spends one counted bounce unit; at budget the run blocks with Shed's budget-exhausted halt".
To escalate to the parent instead, D9 must say the refusal path consults the `Bounces` seam and escalates with cause budget.
Align the Open risks bullet, naming the judge by role rather than by model, and align the acceptance criterion with whichever exit is chosen.

### [MEDIUM] D8 leaves the at-entry settle path unbounded

**Location:** `_lyx/discussion/decision-record.md` D8.

**Issue:** D8 checks keys "on harvesting a judge pass ... for any verdict".
`Bouncer.Call` has a second settle path that is not a harvest: an unattached verdict already on disk at Call entry that is not a settled generation reaches `if b.judged(n) { return b.settle(ctx, n, false) }` (`bouncer.go:395-397`).
A driver killed between the judge's ledger write and the harvest lands there with a CONTINUE or CIRCLING verdict.
D7 states its at-entry behavior; D8 is silent.
If the plan reads "harvest" literally, that ledger settles unchecked.
Round N+1's check then compares against the lossy round-N ledger, so the dropped key is lost for good, which is the loss D8 exists to stop.
The rubric's attack-surface rule requires the Decision to state what the guard lets through and what bounds it.

**Fix:** State in D8 whether the key check also runs on the at-entry settle of a CONTINUE or CIRCLING verdict.
Recommended: yes, with the same D9 refusal, since the archive is safe once the entry-time attach probe has found no live judge.
Otherwise name the gap and its bound explicitly.
Add an acceptance criterion for the chosen behavior.

### [LOW] D7's at-entry sentence misdescribes the path and omits its bound

**Location:** `_lyx/discussion/decision-record.md` D7, "A verdict already on disk at Call entry, settled by an earlier Call, is not re-checked."

**Issue:** At Call entry, an unattached CONVERGED on disk always satisfies `settledGeneration` and is cleared and re-seeded (`bouncer.go:346-365`), whether an earlier Call settled it or crashed before harvesting it.
So the unchecked path lets no CONVERGED through to Done.
The sentence implies the only unchecked CONVERGED is one already settled, and it never states that bound.

**Fix:** Replace the sentence with the bound: "A CONVERGED on disk at Call entry with no live judge is never settled again: Go clears the generation and re-seeds, so the guard has no unchecked path to Done."

### [LOW] D2's seed precedence has no acceptance criterion

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria, against D2.

**Issue:** D2 orders seed resolution as per-segment seed, then `seed`, then the segment's resolved judge model.
The criteria cover `seed` alone, `webster_seed` alone, `plan_judge` alone and `plan_judge` with a seed key.
None sets a per-segment seed key and `seed` together, so D2's first step has no criterion.
The judge order (D1) and the remaining seed steps each have one, and no criterion contradicts D2.

**Fix:** Add a criterion: with `seed` and `plan_seed` both set, Plan-Bouncer's seed runs on `plan_seed` and the other segments' seeds run on `seed`.

### [LOW] D9's rationale carries rejected-alternative deliberation

**Location:** `_lyx/discussion/decision-record.md` D9 Rationale.

**Issue:** The rationale explains why overriding the verdict to CONTINUE in Go was not chosen.
That is a rejected alternative, and the support log's Rejected alternatives section already holds it ("Overriding a refused CONVERGED to CONTINUE in Go").
The writer stencil keeps Decisions to Decision and Rationale and sends rejected alternatives to the support log.
One requirement in the sentence is not yet in the Decision: the judge stays the only writer of its three files, so Go never rewrites the verdict, ledger or focus.

**Fix:** Move "the judge stays the only writer of its verdict, ledger and focus files; Go archives and never rewrites them" into D9's Decision text.
Cut the override comparison from the rationale; the support log already holds it.

## Verdicts on the focus items without a finding

- D5's premise holds: the only embedded recipe with `engine: Bouncer` rows is `contracts/recipes/loom-recipe.yaml`, and none of its three Bouncer rows sets `model`, `effort` or `version`; each carries a comment noting the absence.
  The loom recipe is embedded only (`internal/loomrecipe` parses `recipes.LoomRecipe`), so no on-disk recipe can carry the retired keys.
- D7's review-side check agrees with the judge's decision rule in both round variants (`decisionRuleMarker`, `internal/shedadapters/bouncerprompt.go`): any BLOCKING finding, or a parse-error row for the latest round, rules CONTINUE, and the stencil forbids relabelling BLOCKING downward.
- D8's skip on an unparseable previous ledger is bounded as stated, and it is close to unreachable: `judged(N-1)` required that ledger to parse before round N-1 settled.
  D9's re-judge archives only round N's outputs, so the re-judge compares against the same intact N-1 ledger.
- D10's single-writer premise holds: the only usage writer today is the Burler round (`burler.go:486`), and the Bouncer and Burler rows of one segment run as sequential shed rows under the run lock.
  Shuttle's Attach matches only live runs, so an attached pass cannot be recorded twice.
  The malformed-record, failed-write and re-judged-round cases each state one outcome, and D11's read-as-missing rule covers the records D10's archive path can leave.
- Cross-references: no section cites D3 or A2, which were dropped in the parent review.
  The support log's "Settled D1–D11" and "former D3" are history recorded at the time and are accurate as such.
