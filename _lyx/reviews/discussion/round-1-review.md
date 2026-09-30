---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/discussion/decision-record.md (D2 gate verdict table)
    summary: D2 gate table states no row precedence, lets a matching approval outrank a closed PR if read top-down, and has no row for the no-PR-required mode Publish already short-circuits
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (D4)
    summary: approve today refuses unless the run is at Publish; D4 never moves that check to the gate row, so after the split approve refuses exactly where the run waits
  - id: F3
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (D3, D4)
    summary: the exhausted-budget boundary is unstated and D3 claims exhaustion leaves the run blocked at the gate, but a reject that refuses at count >= budget leaves it awaiting there
  - id: F4
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (D5)
    summary: the append-only check baselines on a snapshot taken before the session, so a retry after a Stuck attempt accepts that attempt's leftover edits; recovery from a run blocked at the rework row is never stated
  - id: F5
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (D5, Open risks)
    summary: crash between the rework commit and the record deletion is only a risk and an acceptance criterion; no decided detection mechanism reaches the plan writer
  - id: F6
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (D8)
    summary: D8 never defines the decision identity; batten's existing identity is approved_at plus head_sha, which collides between an approval and a rejection at the same head within one second
  - id: F7
    severity: LOW
    location: _lyx/discussion/decision-record.md (Auto-mode assumptions, last bullet)
    summary: a plain loom start no longer pushing a hand-made fix is an observable behavior change with docs consequences, recorded only as an assumption
  - id: F8
    severity: LOW
    location: _lyx/discussion/decision-record.md (Acceptance criteria)
    summary: the reject refusal criterion omits D4's origin URL unusable refusal
  - id: F9
    severity: NIT
    location: _lyx/discussion/decision-record.md (D6 Rationale)
    summary: D6's rationale clause about --fresh discarding completed batches argues a rejected alternative that is missing from support-log.md's Rejected alternatives
---

### [BLOCKING] D2 gate table: precedence, closed-PR ordering, no-PR mode

**Location:** `_lyx/discussion/decision-record.md`, D2 table.

**Issue:**
The table never says whether rows are evaluated in order.
Read top-down, the second row (approval matches PR number, PR head and local HEAD) fires before the closed-unmerged row, so a PR the operator closed with a still-matching approval record returns `Done` and Finalize lands it locally.
Today's `checkApproval` in `internal/landingshed/publish.go` checks merged and closed before matching the record, for exactly this reason.
Two matching records at once (a crash inside `approve` or `reject` between writing one record and removing the other) have no stated verdict, and D4 does not fix the write/remove order that would rule the case out.
The malformed-record row's position relative to a valid record of the other kind is also open.
Separately, `Publish` returns `Done` immediately when the parent branch is not in `require_pr_to_base` (Publish step 2), with no PR and no GitHub call.
The gate has no row for that mode, so every no-PR-required run reaches a gate that finds no PR and has no verdict for it; a sane implementation of the table as written would await forever.
The same gap covers a required PR that the gate cannot find, and the origin-URL and client failures that are not "GitHub failures".
Publish's own verdict for a closed PR after the split is also unstated, while the gate carries a closed-PR row of its own.

**Fix:**
State that the gate evaluates rows in order and the first match wins, and reorder: no PR required → `Done` without GitHub; merged → `Done`; no PR found, or closed unmerged → `Awaiting`; malformed record → `Awaiting`; matching approval → `Done`; matching rejection → `Stuck`; stale record → `Awaiting`; no record → `Awaiting`.
Put every other non-transient failure (origin URL, client, GitHub) under one `Awaiting` row.
Fix the verb write order (remove the other kind's record first, then write), so a crash leaves at most one record, and say the approval row still outranks a rejection if both ever match.
State that Publish keeps its closed-PR `Stuck`, since Publish's `Stuck` halts for a human while the gate's never may.

### [MEDIUM] approve's row check is never moved to the gate

**Location:** D4.

**Issue:**
`approveVerb` in `internal/loomcli/approve.go` refuses unless `CurrentProducer == loomshed.NamePublish`.
D4 extends `approve` only to remove a pending rejection record.
After the split the run waits at the gate, so an implementation that follows the record literally has `approve` refusing every waiting run.
The help text and the `ly-drive` wording also name Publish as the approval point.

**Fix:**
Add to D4: `approve` accepts a run awaiting or blocked at the gate row instead of `Publish`, and its help names the gate; add the matching acceptance criterion.

### [MEDIUM] Bounce budget boundary and exhausted state disagree

**Location:** D3 rationale, D4 refusal table.

**Issue:**
The gate returns `Done` only on approval or merge, so its `episodeStuckCount` accumulates across every rejection, and `Routing.Bounces` reports that count for the gate row.
`shedengine` blocks the Stuck that arrives when the count already equals the budget, so five rejections bounce and a sixth would block.
D4 says `reject` refuses when the budget is "already exhausted" without the boundary.
If it refuses at count >= budget, no sixth Stuck is ever produced, and an exhausted run sits `Awaiting` at the gate under the no-record reason, not blocked as D3's rationale says.
If it refuses one later, `reject` records a rejection that the engine then blocks, which is what D3 describes but contradicts D4's intent.
Neither reading says "five" means five rejection rounds per task.

**Fix:**
State in D3 that the budget is five rejection rounds per task, since the gate's episode never resets between rounds.
State in D4 that `reject` refuses when `Routing.Bounces` reports count >= budget.
Correct D3's rationale: an exhausted run stays awaiting at the gate, and `approve` is the only way forward.

### [MEDIUM] Rework row: append-only baseline and blocked-row recovery

**Location:** D5.

**Issue:**
The check compares the plan after the session against "the parsed plan snapshot taken before the session".
A rework row that returned `Stuck` is blocked with no `on_stuck`, and `lyx loom start` re-calls it (shed step 3b); the failed attempt's uncommitted plan edits are still in the working tree, so the retry's snapshot already contains them and they pass.
The record also never says how the operator gets a run blocked at the rework row moving: `reject` refuses anywhere but the gate row, and `approve` checks only the gate.
With bad findings, the only way out is hand-deleting ephemeral state.

**Fix:**
Baseline the check on the plan as committed at HEAD, read with read-only git, so a leftover edit from a failed attempt is still caught.
State the recovery: `lyx loom start` re-runs the rework row; `reject` also accepts a run blocked at the rework row and replaces the pending findings, since nothing was committed and the local HEAD still equals the PR head.
Neither this replacement nor the retry counts against the budget, because the gate authored no new `Stuck`.

### [MEDIUM] Crash re-entry has no decided mechanism

**Location:** D5, Open risks, Acceptance criteria.

**Issue:**
Open risks says re-entry "has to detect that the pending rejection's round is already committed and only finish the tail", and an acceptance criterion tests it.
No decision says how the row detects it, so the plan writer must design the mechanism with nothing to judge it against.

**Fix:**
Decide it in D5: the per-round findings directory records the rejection's `pr_number` and `head_sha`.
On entry, when a round directory committed at HEAD carries the pending record's `head_sha`, the row skips the session and the commit, re-runs the idempotent re-baseline, deletes the record, and returns `Done`.
Drop the Open-risks entry, since the risk is then decided.

### [MEDIUM] Batten decision identity is undefined

**Location:** D8.

**Issue:**
D8 says "once per decision identity" without defining it.
Batten's current `approvalIdentity` in `internal/battenshed/innerrun.go` is `approved_at + " " + head_sha`, with RFC 3339 second resolution.
Reusing that shape for a rejection makes `reject` followed by `approve` at an unchanged head within the same second produce the same identity, so batten would never resume the child for the approval.

**Fix:**
Define the identity as the record kind plus its timestamp plus its head SHA (for example `approve <at> <sha>` / `reject <at> <sha>`), held in the one acted-on marker, so a different kind always differs.

### [LOW] The hand-made-fix behavior change is a decision, not an assumption

**Location:** Auto-mode assumptions, last bullet; Support log Q13.

**Issue:**
Today an operator's local commit reaches the PR on the next `lyx loom start` because Publish re-runs and pushes.
After the split the gate re-runs instead, so the commit is never pushed and `approve` refuses on the HEAD mismatch.
That changes observable CLI behavior that `docs/overview.md` and the `ly-drive` skill describe, so it is a decision with docs consequences; the plan writer needs it as one.

**Fix:**
Move it into D2 as a stated consequence and name it in the Constraints docs bullet; update the Q13 ledger entry to point at D2.

### [LOW] Acceptance criteria omit a D4 refusal

**Location:** Acceptance criteria, `reject` refusals.

**Issue:**
D4 lists "origin URL unusable" as a refusal; the acceptance criterion's refusal list does not.

**Fix:**
Add it to the criterion.

### [NIT] D6 rationale argues a rejected alternative

**Location:** D6 Rationale.

**Issue:**
"and `--fresh` would discard every completed batch" argues against an alternative (running Webster with `--fresh`) that `support-log.md`'s Rejected alternatives never lists.
The clause carries no constraint the plan writer needs beyond the resume behavior D6's Decision already states.

**Fix:**
Drop the clause from D6 and add the `--fresh` alternative to the support log's Rejected alternatives.

The D2 rationale's "no engine change" was checked for the same problem and kept: it echoes the Constraints bullet that `shedengine` is unchanged, not a rejected alternative.
