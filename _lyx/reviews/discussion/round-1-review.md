---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:D4
    summary: D4 bounds the voucher to one observation but not its lifetime when start writes it and no driver consumes it, and leaves the parked-driver resume path and Tier-2-off writes unstated
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:D5
    summary: D5 replaces a deterministic filer with an agent judgement but states no bound on what the drop rule lets through, only an entry in Open risks
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:D2
    summary: D2, Open risks and Q8 leave open whether the loom loader rejects a retired key, which one read of the loader settles as it does not
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:Scope and D1
    summary: Scope-In and D1 both enumerate the same removals, and D1 adds an identifier list that is call-site enumeration; the crash-resume field list is also stated in both D3 and the acceptance criteria
---

### [MEDIUM] D4 states the voucher's per-observation bound but not its lifetime or its write paths

**Location:** `decision-record.md`, D4, its Bound paragraph, and the Auto-mode assumption "Every resume of a loom run goes through `lyx loom start`".
**Issue:** D4 adds a second voucher writer, an escape hatch that suppresses a crash-resume note, and its Bound says it suppresses "at most one entry observation" that matches history length and state.
Three cases the Bound does not cover:

1. `start` writes the voucher before it spawns, but `start` can still fail after that write, for example when the readiness check removes the driver strand and the next `start` re-spawns.
   The voucher then stays on disk, unconsumed, until some entry observation sees the same history length and state.
   A later bare `lyx loom run` or `lyx loom step` over a dead driver's run, the one case D4 says must note, is silently suppressed.
   The Bound reads as if the voucher cannot outlive its start, which is false.
2. D4 writes "right before it spawns a driver", but `start` has a second deliberate-resume path that spawns nothing: it types a resume line into a live, parked driver (`resumeParkedDriver`).
   D4 claims every resume path goes through `start` so none reads as a crash, but it does not say whether the parked path writes a voucher.
3. The observation that consumes the voucher is gated on Tier 2 (D2/D3).
   While Tier 2 is off, the voucher is written by `start` but never consumed, and it can vouch for a stale observation once Tier 2 is switched on.

**Fix:** In D4's Bound, state that `start` writes the voucher only after its own refusals have passed, immediately before the spawn, and that each write replaces any earlier voucher.
State the stale case and its bound: an unconsumed voucher suppresses one later matching observation at most, and the next `start` or completed `step` overwrites it.
State whether the parked-resume path writes a voucher, and that the voucher is written and consumed under the same Tier 2 gate, or that it is written regardless and the stale case is the accepted bound.
Add the consume-once behaviour for these cases to the acceptance criteria only if a new row is needed; the existing "consumed by the first entry observation" row already covers the one-shot part.

### [MEDIUM] D5 removes a deterministic guard without stating what the drop rule lets through and what bounds it

**Location:** `decision-record.md`, D5 and Open risks.
**Issue:** The Go filer filed every anomaly.
D5 replaces it with an agent that "files an issue only when" the evidence shows a lyx problem and drops the rest.
That is a guard downgraded from deterministic to judgement-based.
Under the attack-surface rule the Decision must state what it can now let through and what bounds it.
D5 states neither.
The third Open risk names the failure (a real lyx fault goes unfiled) but the Decision carries no bound, so the plan writer has nothing that limits the loss.
**Fix:** Add a Bound to D5: a dropped event can leave a real lyx fault unfiled, and the evidence is not lost, because the note names the trace file and the run's status and history remain on disk, and the operator can still file through `lyx selfreport create`.
If a stronger bound is intended, such as the reflection recording each dropped note and why in its own output, state it as a Decision and give it an acceptance criterion.
Keep the Open risk as a pointer, not as the only statement.

### [LOW] D2 leaves open a loader question that the loader settles

**Location:** `decision-record.md`, D2 ("if the loader rejects unknown keys, the plan handles the retired key"), Open risks second bullet; `support-log.md`, Q8.
**Issue:** The loom loader (`configengine.Load`, then `yaml.Unmarshal` into the config struct) fills missing template keys and checks only that template keys are present in the file.
Nothing in it uses `KnownFields`, so an extra `selfreport` key in a live `loom.yaml` loads without error.
The conditional wording in D2, the second Open risk and the open Q8 therefore describe an uncertainty that does not exist, and they invite the plan to build handling for a rejection that cannot happen.
The binding requirement and its acceptance row ("a `loom.yaml` that still carries `selfreport: false` loads without error") are already in place and are correct.
**Fix:** In D2 state the requirement unconditionally: the retired key must stay loadable, the loader needs no change, and the acceptance row pins it.
Delete the second Open risk.
Resolve Q8 in the question ledger with the finding.

### [LOW] The same removals and field lists have two homes

**Location:** `decision-record.md`, Scope-In first two bullets against D1's Decision; D3 crash-resume bullet against the fourth acceptance criterion.
**Issue:** Scope-In and D1 both enumerate the filing pass, the filed-title marker with its accessors, the filing seam, the `selfreport` key and the title and body rendering.
D1 adds a further list of identifiers (functions, types, constants, a field).
That list is call-site and symbol enumeration, which belongs to the compiler and to `Plan-Write`'s own lookups, and the requirement it carries is already stated in D1's own first and last sentences ("No Go code path files an issue", `loomcli` no longer imports `selfreportengine`) and in Scope-In.
Completeness-before-leanness check: no constraint is lost by trimming the list, because Scope-In and the acceptance criteria carry the removal requirement.
Separately, the crash-resume note's field list is written out in D3 and again, in full, in the acceptance criteria.
**Fix:** Keep Scope-In as the one list of what is removed.
Cut D1's Decision to the behaviour (no Go path files an issue, `loomcli` no longer imports `selfreportengine`, the one production caller of `CreateIssue` is `internal/selfreportcli`).
Keep the crash-resume field list in D3 and have the acceptance criterion say the note carries the fields D3 lists.

### Focus verdicts that are not findings

- **D4 and the Goal premise:** the premise that every deliberate resume goes through `lyx loom start` is stated in D4's Decision and Rationale, not only in Auto-mode assumptions.
  The orch stencil routes every resume through `lyx loom start`.
  Not a defect; F1 covers the part of D4 that needs more.
- **D2/D3 premise on `cfg.Selfreport`:** the reads of `cfg.Selfreport` are the entry observation, the filing step and the halt note's `TierOneFiled` line.
  Replacing the filing gate and the observation gate with Tier 2 is faithful, and `TierOneFiled` is removed by D3.
  The one behaviour that changes is the voucher consumption under Tier 2 off, raised in F1.
- **D6 and Scope-Out:** `CreateIssue` has two production callers, `internal/loomcli` (removed by D1) and `internal/selfreportcli`.
  `internal/selfreportengine/inbox.go` holds list, get and comment-and-close calls, not `CreateIssue`, so the test-enforced rule and the Scope exclusion do not contradict each other.
  Not a defect.
