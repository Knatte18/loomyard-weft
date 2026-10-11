---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:62
    summary: D2 Bound and its acceptance line claim a Stuck finalize marks the request expired, but evaluateGate stops at the failing mechanical entry and never runs parent-review's Final
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:158
    summary: D11 claims delivery.json shows a delivered-at stamped by this writer after its SendMessage, but the file records only times, and the delivered verb checks no caller and accepts a stamp without a send
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:189
    summary: The Auto-mode assumption and Open risk still say scope-only judging is unconfirmed by the operator, contradicting the operator decision added after Discussion
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md:71
    summary: The question ledger says the ping interval is a constant in internal/parentreview, contradicting D1 and the support log's own Rejected alternatives
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:31
    summary: D1 cites the shuttle await cap as precedent for its lifetime-less-one-minute margin it does not follow, and its never-pinged claim holds only while the wait bound is at most 60 minutes
---

### [BLOCKING] D2's Stuck branch does not mark the request expired

**Location:** `_lyx/discussion/decision-record.md:62`, `:64`, and acceptance line `:207`.

**Issue:**
D2's Bound states that the probe fires only where the at-boundary gate would itself pass on expiry, and that when the mid-turn tree fails the mechanical entry the run halts "with the request already expired".
Acceptance line 207 requires the request to be marked expired and the timeout Warn logged in that `Stuck` case too.
Neither holds against `internal/shuttleengine/wait.go` as it stands.
`evaluateGate` runs the entries in list order and stops at the first failure.
Every later entry is reported `GateEntryNotReached` and its closure is never called.
`finalize(OutcomeDone)` calls `evaluateGate(true)`, so `Final` is only substituted for an entry the loop reaches.
The loom recipe puts `discussion` before `parent-review`, and D7 makes that order mandatory.
When the mechanical entry fails, `parentreview`'s `Final` never runs, so the request is not marked expired and the "parent review timed out" Warn (`internal/parentreview/gate.go:290`) is not logged.
The first sentence of the Bound fails the same way: on a failing tree the boundary gate would re-prompt on the mechanical entry and never reach `parent-review`, so it would not pass on expiry either.
The probe reaches further than the Bound says: it ends a run the boundary path would have kept alive.
The halted run is then left holding an open, unexpired request.
Neither D2 nor D9 says what a resume or a fresh spawn does with that request.
A plan built from this record either writes a test that cannot pass or quietly changes `evaluateGate`'s stop-at-first-failure rule, which no Decision states or bounds.
Round 1's F2 fix added the "request already expired" clause, so this defect came in with that fix.

**Fix:**
Pick one and state it in D2.
(a) Narrow the Bound and acceptance line 207 to what finalize does: on a failing tree the run ends `Stuck` with the request left open and no timeout Warn.
Also state what a later resume or fresh spawn does with that open request past the bound.
(b) Alternatively, decide that finalize runs a hold entry's `Final` even after an earlier entry failed.
State that change to `evaluateGate` and its bound.
Either way, correct the "fires only where the at-boundary gate would itself pass" sentence.

### [MEDIUM] D11 claims more than `delivery.json` records

**Location:** `_lyx/discussion/decision-record.md:158`, Rationale `:162`, acceptance line `:220`.

**Issue:**
D11 says the round-1 `delivery.json` "must carry a delivered-at stamped by this writer after its SendMessage to `ly:orch`".
The Rationale says "this run's records show the writer's send".
`parentreview.Delivery` holds `delivered_at`, `failed_reason`, `prompts`, `last_prompt_at`, `waiting_notifies` and `cap_warned`, and nothing that names who stamped it or that a send happened.
The `review` group's doc in `internal/loomcli/review.go` states "The verbs check no caller identity" and "A delivered stamped without a send stops re-sends".
`delivered` also accepts a slug, so the prime can stamp it.
This run's own file confirms it: `_lyx/reviews/parent-review/round-1/delivery.json` holds `delivered_at` 19:52:27 and `last_prompt_at` 19:52:21, and `verdict.json` holds an approve `recorded_at` 19:52:43.
That supports "a delivery stamp, then a verdict 16 seconds later" and nothing about who sent what.
The landing caveat on line 159 names only the verdict side's gaps, so the description it prescribes would still overclaim the delivery side.
Round 1's F3 fix narrowed only the verdict side, and the "writer's stamp" claim survived it.

**Fix:**
Narrow D11's Decision, Rationale and acceptance line 220 to what the files record: `delivery.json`'s `delivered_at` and a `verdict.json` `recorded_at` after it.
Extend the landing caveat so it also says neither file records who stamped the delivery or that a SendMessage preceded the stamp.

### [LOW] Scope-only judging is both operator-decided and unconfirmed

**Location:** `_lyx/discussion/decision-record.md:189` (Auto-mode assumptions), `:193` (Open risks); `_lyx/discussion/support-log.md:81` (Question ledger).

**Issue:**
The "Added after Discussion (operator, 2026-10-11)" entry at line 166 records the operator's decision that parent review stays scope-only.
The Auto-mode assumption still says the scope-only brief is kept "without the operator confirmation".
The first Open risk still asks the parent reviewer to reject the record "if a broader parent review is wanted".
The ledger still says "unconfirmed by the operator".
The record now contradicts itself on whether the operator has decided.

**Fix:**
Drop the Auto-mode assumption and the Open risk.
Resolve the ledger line as decided by the operator.

### [LOW] Ledger contradicts D1 on where the interval lives

**Location:** `_lyx/discussion/support-log.md:71`.

**Issue:**
The ledger answers "Ping interval and cap?" with "4 minutes, 3 per round, constants in `internal/parentreview`".
D1 tells the gate the interval as a `GateConfig` duration derived from the engine's cache lifetime, and only the cap is a constant.
The support log's own Rejected alternatives list rejects "A 4-minute interval constant in `internal/parentreview`".

**Fix:**
Restate the ledger answer: the interval is the writer's cache lifetime less one minute, told through `GateConfig`, and the cap of 3 is a constant in `internal/parentreview`.

### [NIT] D1's interval sentence cites a precedent it does not follow and overstates "never pinged"

**Location:** `_lyx/discussion/decision-record.md:31`; acceptance line `:203`.

**Issue:**
"as `lyx shuttle await`'s cap sits below it" points at `shuttleengine.DefaultAwaitCap`, which is 250 seconds, not the lifetime less one minute, so it does not support the margin chosen.
"so a writer whose role is mapped to `1h` is never pinged" holds only while `parent_review_wait_min` is at most 60, its template default.
A larger bound gives that writer a 59-minute interval, which is the rule working as intended but contradicts "never".

**Fix:**
Drop the await clause, since Auto-mode assumptions already name the margin as the writer's pick.
Write "is not pinged at the default 60-minute bound", or "is pinged only when the bound exceeds its lifetime", in D1 and acceptance line 203.

## Focus verdicts

- D1 interval and cost rationale: not a defect beyond F5.
  The Rationale (line 50) and Rejected alternatives weigh mapping the role to `1h` through `claude_prompt_cache_ttl_roles` against pings, and the cost arithmetic checks out (0.3 against 1.25 and 0.75).
  The interval's source respects `PATTERN-shuttle-provider-seam`: a provider-neutral `Engine` method answered by `claudeengine` from `resolvePromptCacheTTL`, restated in Constraints.
- D2 Bound against `finalize`: defect, F1.
- D11 acceptance proof against `delivery.json` and `verdict.json`: overclaimed, F2.
