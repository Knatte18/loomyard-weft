---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D2.8, D2.4 and the Acceptance criterion after a verify gate fix
    summary: The Master writes the Verify gate fixes summary before the gate walks the fixer's commits, and a passing evaluation sends the Master nothing, so no gate result it re-arrives with can carry the commits and departures of the fix that passed
    fix: Rewrite D2.8 so the gate closure records the fix:-commit SHAs and subjects and the Pattern-Departure trailers it walks at each evaluation into its VerifyGateNotes, and Run writes them after the wait into summary.md's `## Verify gate fixes` section the way VerifyGateNotes.Apply already writes the flaky identities into summary.md; the Master's step-4 section names only the findings from the findings file. Drop D2.4's clause on the verify gate result the Master re-arrives with naming fix_commits and departures, and replace the Acceptance criterion with one stating that after a fix the gate passes on, summary.md's `## Verify gate fixes` section names each fix commit's SHA and subject and each trailer, written by Go after the wait, and a fix with no trailer names no departure.
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D5.4, the malformed-record refusal sentence
    summary: The refusal's way forward has the run's parent hand-fix the decision record, but the orch, the parent of every loom run, is forbidden to edit or commit the decision record, so the way forward is unreachable for that actor (PATTERN-refusal-way-forward)
    fix: In D5.4 replace "the run's parent or the operator fixes the named section of `_lyx/discussion/decision-record.md`" with "the operator fixes and commits the named section of `_lyx/discussion/decision-record.md` (the orch never edits the record and escalates the refusal to the operator)", keeping the `lyx loom validate-discussion` check and the re-step; update the matching Acceptance criterion and Q13 in the support log's question ledger to name the operator instead of the parent.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D5.2, D5.3 and D5.7
    summary: A PATTERN change appended by decision add after the plan gate never reaches the plan's section, since the verb never touches the plan, and D5.7 does not say whether the landing check reads the record's list or the plan's, so the entry Webster-Review measures against the plan can go unchecked
    fix: In D5.7 state that the landing check reads the declared entries from the record's `## PATTERN changes` as rendered into the Webster-Review directive by D5.4, not from the plan's section. In D5.3 add that a bullet `decision add` appends after the plan gate reaches the plan's section only at the next plan gate, where `pattern-changes-mismatch` asks for it. Add an Acceptance criterion that a bullet appended after the plan gate is rendered into every later directive and checked by D5.7's landing check.
---

### [BLOCKING] Verify-fix commits cannot come from a gate result the Master re-arrives with

**Location:** `_lyx/discussion/decision-record.md` D2.8, D2.4 and the Acceptance criterion that begins "After a verify gate fix".

**Issue:** `webster-template-master.md:171` has the Master rewrite `summary.md`, the `## Verify gate fixes` section included, on the fixer's completion notification, and only then end the turn that re-arrives at the gate.
`internal/websterengine/doc.go:600` and `:616` show that the gate records the pre-fix head at the first failure and walks the commits from it to HEAD at the next arrival, after the Master has already written the summary.
A passing evaluation ends the run without a message to the Master, so in the common case of one fix and a pass, no `Gate findings recorded at …` message ever carries that fixer's commits.
The design rests on a false premise: the source D2.8 names never holds the data when the summary is written.
`VerifyGateNotes.Apply` (`internal/websterengine/verifygate.go:65`) is the existing route by which Go writes what the gate observed into `summary.md` after the wait.
That route keeps the operator ruling that the Master never reads raw fork output and that a departure reaches it through Go.

### [BLOCKING] D5.4's way forward names an edit the orch may not make

**Location:** `_lyx/discussion/decision-record.md` D5.4, the sentence beginning "A record present at a told path that fails to parse refuses the spawn".

**Issue:** The way forward reads "the run's parent or the operator fixes the named section of `_lyx/discussion/decision-record.md`".
In a loom run the parent is the orch, and `contracts/stencils/orch/orch-template-role.md:107` reads "Never edit or commit in either worktree of a task pair, `_lyx/plan` and the decision record included."
The `validate-discussion` check and the re-step are valid way-forward steps under `contracts/specs/refusal-spec.md`'s "What a way forward is", but they only make progress once the record is fixed.
No session that holds the parent role may make that fix, so for a loom run the refusal names no reachable way forward.
The row placement D5.4 states is sound: websterengine's row goes in `## webster`, the Bouncer's and the Burler-round row's in `## shed` beside the other review-segment rows, and loomengine's in `## loom`.

### [MEDIUM] A PATTERN change appended after the plan gate has no stated landing source

**Location:** `_lyx/discussion/decision-record.md` D5.2, D5.3 and D5.7.

**Issue:** D5.2's rationale is a change decided at PR-Rework or by a parent after Discussion, and `internal/websterengine/doc.go:297` describes the Webster-stage route as `lyx loom decision add`, a card edit and `rebaseline`.
`internal/loomcli/decision.go:19` reads "The verb never routes the run, never touches the support log or the plan, and an approved plan is not re-planned by it."
A bullet appended at Webster therefore stays out of the plan's `## PATTERN changes` until a later plan gate, while D5.3's rationale has Webster-Review measure against the plan.
D5.7 says "each declared entry" without naming whether the list comes from the record or the plan, so the D5.4 Bound's claim that D5.7 checks that the declared text lands does not cover such an entry.
The rest of the focus question holds: D5.4's Bound states what the declaration lets through and what bounds it, and D5.2 limits callers to the existing `--by` guards on an append-only bullet.

## Rejected

- The D2.6 focus question is settled as not a defect.
  Its Bound names the plan's `## verify:` command run by the plan-level gate, which matches `internal/websterengine/doc.go:602`.
  The no-verify-section case keeps `FAILED` by rule (D2.6) and by an Acceptance criterion.
- Round 1's F3, F5 and F6 fixes hold with no regression; round 1 recorded no disputed or deferred items.
