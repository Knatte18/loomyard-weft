---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:D7
    summary: D7 does not say what happens when the remote update fails after the local reset, and its pre-mutation refusal makes the already-diverged #404 state unrepairable
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:D7
    summary: D7's lease is "the remote tip read just before", which can differ from the tip the ancestry check was run against, so a remote move in between is not bounded
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D6
    summary: D6 does not cover a batch with terminal status stuck, which the state model has beside done, dead and failed
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D3
    summary: D3's bound names where the relaxation applies but not what it lets through, and a Delete target missing for any reason passes with only an advisory
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:D1,D2,D3,D4
    summary: D1 and D2 scope dispatch checks to not-yet-completed cards, D3 says not yet begun, and D4 reuses D2 at record-batch and recover-batch time without stating its card scope
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:D6,D4
    summary: D6's premise that recover-batch re-drives every failed batch ignores that recover-batch refuses a failed batch carrying uncheckable paths
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:D9
    summary: D9 depends on open question Q11 only through an Auto-mode assumption, with no fallback text if the plan finds no answer
---

### [BLOCKING] D7 leaves a half-done reset and the diverged #404 state unhandled

**Location:** `_lyx/discussion/decision-record.md`, D7 Decision and Bound.
**Issue:** D7 refuses before any mutation when the remote tip holds a commit not reachable from the pre-reset local HEAD.
Two states fall outside it, and the Decision is silent on both.
First, the remote update runs after fabric has already reset the local warp checkout, so a failed update (lease rejected, network down) leaves the local branch rewound and the remote not.
A re-run of the reset then has a pre-reset HEAD that lacks the remote's abandoned commits, the refusal fires on the run's own commits, and its way forward ("integrate them") is wrong for them.
Second, #404 itself is that state: the remote holds the run's own abandoned commits and the local HEAD was already rewound by an earlier reset, so the rubric's attack-surface question for the force update gets a bound that blocks the very case the bug reported.
The Bound says what the update may discard but not what a refused or failed reset leaves behind or how the operator gets out.
**Fix:** state in D7 the order and failure semantics (what a failed remote update leaves, and that a re-run converges instead of refusing), and how a remote tip that is already ahead of a rewound local HEAD is handled.
One workable shape: record the pre-reset HEAD durably before the local reset so the re-run uses it, or run the remote update first.

### [MEDIUM] D7's lease and the pre-mutation fetch can disagree

**Location:** `_lyx/discussion/decision-record.md`, D7 Decision and Bound.
**Issue:** the ancestry check runs against the fetched remote tip, but the Bound puts the lease on "the remote tip read just before" the update.
If those are two reads, a foreign commit pushed between them is covered by neither the check nor the lease, which is the case the focus directive asks about.
**Fix:** state that the lease value is the tip the ancestry check was run against, so any later move makes the update refuse instead of overwrite, and name the refusal's way forward.

### [MEDIUM] D6 omits the stuck terminal status

**Location:** `_lyx/discussion/decision-record.md`, D6.
**Issue:** the batch state model has four terminal statuses: done, stuck, dead and failed (`internal/websterengine/state.go`, `BatchState.Status`).
D6 accepts `failed` and `dead`, refuses `done` and non-terminal, and says nothing of `stuck`.
`recover-batch`'s own header says it escalates a batch a fork reported stuck, so `stuck` is a re-driven state whose edited card would be rendered from the current plan, exactly D6's rationale.
The Acceptance criteria repeat the gap.
**Fix:** add `stuck` to D6's accepted set, or state why it is refused, and cover it in the criterion.

### [MEDIUM] D3 states where the relaxation applies, not what it lets through

**Location:** `_lyx/discussion/decision-record.md`, D3 Decision and Bound.
**Issue:** the rubric requires a downgraded guard to state what it can now skip and what bounds it.
D3 bounds the scope (Delete targets, dispatch only) but not the content: a Delete target missing because an earlier card edited or removed the wrong file, or one that never existed in the tree, passes with an advisory the run may never read.
Plan-Gate time catches a typo only for a plan not yet run, and a rework plan checked mid-run may still be refused for a target an earlier batch deleted, since D1 puts the format set on the Rework-Plan-Gate and D3 relaxes only dispatch.
**Fix:** state the residual let-through and its bound (for example, the advisory names the target and the card's own `delete-not-done` check at `record-batch` still holds), and say how the Rework-Plan-Gate treats a Delete target an earlier batch removed.

### [LOW] Card scope differs between D1, D2, D3 and D4

**Location:** `_lyx/discussion/decision-record.md`, D1, D2, D3, D4.
**Issue:** D1 and D2 scope dispatch checks to "cards not yet completed", D3 to "a card not yet begun", and D4 reuses D2's check at `record-batch` and `recover-batch` time, where the deleting card is begun and not completed.
D2's scope as written covers that case, but D4 never says it needs it, and D3's "not yet begun" can be read as narrower than D2's scope for the same dispatch validation.
**Fix:** use one scope phrase across the dispatch checks, and state in D4 that the check runs over the batch's own begun cards against later not-completed cards.

### [LOW] D6 assumes recover-batch re-drives every failed batch

**Location:** `_lyx/discussion/decision-record.md`, D6 Rationale and D4.
**Issue:** `recover-batch` refuses a failed batch that carries uncheckable suspect paths, toward `run --fresh` (`internal/websterengine/recoverbatch.go`).
D6's rationale and D4's "every other batch failure keeps `recover-batch` as its way forward" do not account for it, so a restamped card there still cannot be re-driven.
**Fix:** state that D6's way forward holds only where `recover-batch` accepts the batch, or name the refusal's own way forward for the rest.

### [LOW] D9's dependence on Q11 lives only in the support log and an assumption

**Location:** `_lyx/discussion/decision-record.md`, D9 and Auto-mode assumptions; `_lyx/discussion/support-log.md`, Q11.
**Issue:** Q11 is open "for the plan", but the record carries it only as an assumption that the plan will establish the answer.
The plan writer never reads the support log and the record gives no wording if the plan finds no answer.
**Fix:** state in D9 that the note names what ends the session only when the plan establishes it, and otherwise omits that clause without a guess.

## Checked and not flagged

- D1's reasoning for the cycle case and the card-order guarantee follows from the Decision, and including **Delete:** targets matches `deriveEdges`, which reads the flat `Targets` list; the record states the cost as an accepted assumption.
- D5's premise (no begin-batch record means no attributable fork) is consistent with the existing refusal text in `beginbatch.go` and with D4.
- D8 and D10 introduce no verb, flag or config key; D8's way forward uses the existing resume verb and a plain git merge.
- The record carries no rejected alternatives and no deliberation narrative that needs relocating, so there is no relocation finding.
