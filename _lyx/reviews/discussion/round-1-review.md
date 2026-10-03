---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md D1
    summary: legacy verdict aliases apply on every read, so a legacy APPROVED harvested in the current Call settles Done, and D1 states no bound for that path
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md D3, D6
    summary: the record never says what the judge decides when the latest round's facts row is a parse error, so CONVERGED stays reachable with no class evidence
  - id: F3
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md D6
    summary: the departure rule names only downgrade examples, never states it runs both ways, and leaves open whether a BLOCKING finding may be departed from to reach CONVERGED
  - id: F4
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md D7, D1
    summary: D7 requires an accept to clear on re-entry but never states the pending-versus-settled distinction the Call-entry check needs, which the support log defers to the plan
  - id: F5
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md Acceptance criteria, facts fixture
    summary: the three-round facts fixture expects a key seen in round 3 and reopened after round 2, but D3 reads only ledgers 1..N-1, so round 3 is not in any ledger yet
  - id: F6
    severity: LOW
    location: _lyx/discussion/decision-record.md D7 Bound
    summary: the bound names the operator's shell as the trust boundary but does not say an agent shell in the task worktree passes the same way, nor what bounds it
  - id: F7
    severity: LOW
    location: _lyx/discussion/decision-record.md Auto-mode assumptions
    summary: the section restates D8 (seed on the judge key) and D5 instead of recording assumptions, and its straddle line claims a coverage F1 and F2 show is incomplete
  - id: F8
    severity: LOW
    location: _lyx/discussion/decision-record.md Open risks
    summary: the requirement that the continue path's budget block names its cause sits only in Open risks, not in D7 where the plan writer acts on it
---

### [MEDIUM] Legacy aliases settle Done at harvest

**Location:** `_lyx/discussion/decision-record.md` D1.
**Issue:** D1 says "On read only, a legacy `APPROVED` parses as `CONVERGED`", and its rationale scopes the purpose to the Call-entry "settled earlier" check.
But the Bouncer reads a verdict through one parser (`parseVerdict` in `internal/shedadapters/bouncerfiles.go`) on every path, including `settle`, which both the judge-spawn harvest and the entry-time attach harvest call (`internal/shedadapters/bouncer.go`, the `attached && b.judged(n)` branch).
A judge spawned by the old binary and still alive when the new binary is re-called is attached, harvested, and its `APPROVED` maps to `CONVERGED` and returns Done.
That old judge read artifacts and the fixer report under the old prompt, which is the exact failure this task removes.
A new-prompt judge that slips and writes `APPROVED` is accepted the same way.
The alias is a routing edge to Done, and D1 states no bound for it.
**Fix:** scope the aliases in D1: they apply only to a verdict already on disk at Call entry (the clear and the `BLOCKING` replay); a legacy word harvested from a judge this Call spawned or attached never settles, and the round is re-judged under the new prompt.
Update the verdict-parser acceptance criterion to match.

### [MEDIUM] Parse-error latest round has no decision rule

**Location:** `_lyx/discussion/decision-record.md` D3 and D6.
**Issue:** D3 gives an unparseable review a parse-error row "instead of counts".
Under the new binary a done burler round always has a parsing review (`burlerengine.Engine.Run` fails loud on a parse error), so the case arises exactly when the latest round was written by the old binary, with no `class`, or was hand-edited.
D6's rule is phrased over gating-class counts; with a parse-error row for the latest round there are none, and D2 still hands the judge the review file, so the judge can read "no `design` finding" off a file that has no classes at all and return `CONVERGED`.
D3's fail-soft row is a softened guard, and neither D3 nor D6 states what it may let through.
**Fix:** state in D6 that a latest round whose facts row is a parse error never converges: the judge returns `CONTINUE`, so a fresh round runs under the new contract; a parse-error row is no evidence for `CIRCLING` either.
Add an acceptance criterion for it.

### [MEDIUM] Departure rule's bound is one-directional and silent on BLOCKING

**Location:** `_lyx/discussion/decision-record.md` D6.
**Issue:** D6's bullet allows a departure "only for a finding whose own summary shows it is mislabeled", and both examples (a nitpicky MEDIUM, a `consistency` problem filed as `design`) move a finding out of gating, toward `CONVERGED`.
The Open risks section claims the opposite direction too ("D6 lets the judge treat a mislabeled finding as gating"), which D6 itself never states.
D6 also never says whether a BLOCKING-severity finding may be departed from; since D6 makes a BLOCKING of any class block convergence because it is the reviewer's strongest signal (support-log turn 6), letting a judge waive it from a one-line summary would remove that guard.
The bound "only from its summary" is consistent with D2: the latest review file is on the judge's reading list and carries each finding's summary and location.
**Fix:** state in D6 that a departure relabels one named finding's effective class or severity, in either direction, from what the review file says about it; that a BLOCKING-severity finding is never relabeled downward; and that the rationale names every departed finding's ID.

### [MEDIUM] Accept re-entry requirement not stated in implementable form

**Location:** `_lyx/discussion/decision-record.md` D7 (`accept` bullet), D1 (Call-entry sentence).
**Issue:** D1 names an on-disk `CONVERGED` verdict as what makes the Call-entry check clear and re-seed.
For `CONVERGED` that works because the verdict and the Done happen in one Call, so an on-disk `CONVERGED` at entry always means "settled earlier".
An accept is different: the verb writes its decision between Calls, so at the next Call entry an accept on disk means "settle now", and at a later re-entry the same file means "settled earlier".
D7 says only that the accept is "recorded so a later re-entry … clears and re-seeds", support-log turn 8 defers the mechanism to the plan, and the Open risks line repeats the danger.
The plan writer never reads the support log and gets a requirement with no state model: nothing says what marks an accept as consumed or when, so the commit-seam-failure-then-resume path (which clears after `CONVERGED`) has no defined counterpart.
**Fix:** state in D7 that an accept has two recorded states: pending, written by the verb, and settled, written by the Bouncer before it runs the Approve seam, mirroring a `CONVERGED` verdict being on disk before Approve; the Call-entry check treats a settled accept exactly like an on-disk `CONVERGED` verdict, and only a pending accept settles.
Extend D1's Call-entry sentence to cover a settled accept.

### [MEDIUM] Facts fixture contradicts D3's ledger range

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria, the `round-<N>-facts.md` criterion.
**Issue:** D3 lists recurring keys "from the ledgers of rounds 1..N-1"; a ledger entry's `rounds` field records where the judge saw a key (`ledgerEntry` in `internal/shedadapters/bouncerfiles.go`).
The criterion asks a fixture "of three rounds" to list "a key seen in rounds 1 and 3 as recurring" and as reopened "when round 2's ledger had it resolved".
The facts for round 3 read ledgers 1 and 2 only; round 3's ledger is written by the judge the facts feed, so no ledger yet records the key in round 3 or as open after round 2.
A plan writer following the criterion literally writes a test D3's design cannot pass.
**Fix:** restate the criterion over the facts for round 4 of a fixture whose rounds 1 to 3 are judged, with the key resolved in round 2's ledger and open with rounds 1 and 3 in round 3's ledger.

### [LOW] D7 bound does not name the agent-shell case

**Location:** `_lyx/discussion/decision-record.md` D7, `Bound:` paragraph.
**Issue:** D7 says the verbs "check no caller identity, like `lyx loom review`; the trust boundary is the operator's shell".
That matches every existing loom verb (no verb in `internal/loomcli` checks a caller), and a task worktree addresses itself, so any shell in the task worktree passes, including an agent's tmux session: a judge left alive after a driver crash is exactly the case the Constraints' circling re-call probe exists for.
The bound that applies to such an agent is the verbs' own precondition plus the fact that no loom stencil names the verbs, and D7 does not say so; read literally, "the operator's shell" claims a boundary the verb does not enforce.
**Fix:** state in D7 that any shell addressing the run, an agent's included, can record the decision; that the precondition (awaiting at a Bouncer row on a `CIRCLING` round, one decision per round) is the whole mechanical bound; and that no agent stencil names the verbs, as with `lyx loom approve`.

### [LOW] Auto-mode assumptions restate decisions

**Location:** `_lyx/discussion/decision-record.md` `## Auto-mode assumptions`.
**Issue:** "The seed pass moves to the `judge` key with the judge" restates D8, and the support-log question ledger already records it as Q9.
"`design` alone gates, per the draft design doc, in all three segments" restates D5; the assumption hiding in it is that a draft the record itself calls "not a spec" (D9) is taken as the operator's intent.
"D1's aliases and D3's fail-soft rows cover that, nothing else does" claims a completeness F1 and F2 show does not hold as written.
**Fix:** drop the D8 restatement; reword the gating line to the assumption it carries; reword the straddle line to the assumption (in-flight runs straddle the update) and point at D1, D3 and D6 for the coverage.

### [LOW] Budget-block Reason requirement sits only in Open risks

**Location:** `_lyx/discussion/decision-record.md` `## Open risks`, the `continue` line.
**Issue:** "the Reason must say so" is a requirement on the implementation, not a risk.
D7 states that a `continue` at the last budgeted round blocks on budget, but not that anything tells the operator why.
**Fix:** move the requirement into D7's `continue` bullet: the `CIRCLING` Awaiting Reason says when `continue` would block on budget and names `lyx loom goto`; drop the Open risks line.

## Verdicts on the remaining focus items

- **goto verb.**
  Both `lyx loom goto` and `lyx shed goto` exist with the same contract (`internal/loomcli/cli.go`, `internal/shedcli/cli.go`): a goto from an awaiting run moves only strictly before the current row and resets the target row's segment bounce budget.
  In every segment the Bouncer row precedes its Burler row (`contracts/recipes/loom-recipe.yaml`), so from Awaiting at a Bouncer the nearest goto target is the writer row, and D7's "full rewrite" claim holds.
  The support log naming `lyx shed goto` is accurate too, so this is not a defect.
- **D8 short form.**
  `sonnet[medium]` parses under `internal/modelspec.Parse` (a bare bracket token means `effort=<token>`), so the short-form key validates today.
