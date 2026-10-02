---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md D2
    summary: D2 drops begun non-terminal cards from validate's resolve pass, a downgraded guard, and states no bound for what that lets through
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md D3 Bound
    summary: D3's bound credits the hash comparison with refusing a foreign edit, but the adopted hashes never hold one; the existing PlanEditError precondition at every rewrite site is what refuses it, and the bound does not say so
  - id: F3
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md D5
    summary: D5 does not say what happens to the strand's commits on the task branch when Go rejects them before escalating
  - id: F4
    severity: LOW
    location: _lyx/discussion/decision-record.md D1 Bound
    summary: D1's bound relies on record-batch/recover-batch done-checks, which never run for a begun batch that never reaches a terminal record; the run-exit every-batch-done check is the real backstop and goes unnamed
  - id: F5
    severity: LOW
    location: _lyx/discussion/decision-record.md D5 Bound
    summary: D5's bound presents post-fix triage as the success judge without stating that a deleted or skipped test reads as cleared; that hole lives only in Open risks
  - id: F6
    severity: LOW
    location: _lyx/discussion/decision-record.md Auto-mode assumptions
    summary: Auto-mode assumptions restates D1/D3/D5 with rejected-alternative narrative, while two requirements it carries (no new config key, recovery_timeout_min) are in no Decision
  - id: F7
    severity: LOW
    location: _lyx/discussion/decision-record.md D5
    summary: D5's escalation path says exactly as today (bisect, ...), leaving open whether a post-fix escalation bisects again or records the pre-fix bisect result
  - id: F8
    severity: NIT
    location: _lyx/discussion/decision-record.md D5 Bound
    summary: the head check says the strand's own non-merge commits under the existing merge-only rule, which reads as both forbidding and tolerating merge commits on top
---

### [MEDIUM] D2 downgrades `validate`'s guard with no bound

**Location:** `decision-record.md`, D2.

**Issue:** `lyx webster validate` scopes today by `webstercli`'s own `completedCards` (terminal batches only, `internal/webstercli/validate.go`), so a begun non-terminal card still gets its resolve pass there.
D2 moves `validate` to `begunCards`-style scoping, so drift in a begun, unrecorded card's own targets is no longer reported by any of the three sites.
The rubric's attack-surface rule requires a Decision that downgrades a guard to state what it lets through and what bounds it; D1, D3 and D5 do, D2 does not.
The bound exists in code: the card was fully validated at its first `begin-batch`, its bytes are pinned by `CardHashes` (`batchCardEditError` at `record-batch`/recovery, `rebaseline`'s begun-card check), and `record-batch`/`recover-batch` run done-checks and drift detection on it.

**Fix:** Add a Bound to D2 naming what is let through and those three bounds.

### [MEDIUM] D3's bound misattributes what refuses a foreign edit

**Location:** `decision-record.md`, D3 Bound; acceptance criterion "#330: a foreign edit to a begun card is still refused … including when a sanctioned rewrite ran after the foreign edit".

**Issue:** The two hashes are comparable: `batchCardHashes` and `planFileHashes` (`state.go`, `fingerprint.go`) both take SHA-256 of the same card file's bytes, keyed by `NN-<slug>` id and by file name respectively.
But the comparison cannot catch a foreign edit.
An unadopted foreign edit is in neither the recorded begun hash nor the adopted `PlanFileHashes`, so the two still match and the card's hash would move onto bytes carrying the edit.
`rebaseline` refuses to adopt an edit to a begun card, so the adopted hashes never hold one either.
What refuses the acceptance criterion's case is the existing precondition: every rewrite site runs `PlanEditError` (`begin-batch`, `record-batch`, recovery, `validate`, which skips its restamp on an edit) or run entry's fingerprint compare before rewriting, so the sanctioned rewrite never runs after an unadopted foreign edit.
The comparison only keeps a card whose hash an earlier, untracked rewrite already moved (a pre-deploy stale run) at its old hash.
A plan writer reading the bound as written could treat the comparison as the guard and miss that the hash move must ride only on a call whose edit check passed.

**Fix:** Rewrite the bound: the update runs only inside a call whose foreign-edit check passed, which every sanctioned rewrite site already has, and `rebaseline` never adopts an edit to a begun card; the hash comparison, keyed by the card's file name in `PlanFileHashes`, keeps an already-stale card refused.

### [MEDIUM] D5 leaves rejected strand commits unaddressed

**Location:** `decision-record.md`, D5 Bound, third bullet.

**Issue:** When Go rejects the strand's commits (plan directory or `_lyx` touched, HEAD moved otherwise), "the attempt counts as failed" and the run escalates, but the Decision is silent on whether the commits stay on the task branch, are reset, or are reported.
A rejected commit that touched `_lyx` or the plan directory is exactly what an operator must see before resuming, and an automated reset is unsafe when HEAD moved by something other than the strand.

**Fix:** State it: Go never resets the branch; the commits stay, and summary.md and the escalation's way-forward name the pre-fix head and the rejected commits so the operator decides.

### [LOW] D1's bound omits the case it was asked about

**Location:** `decision-record.md`, D1 Bound.

**Issue:** The bound names the producing card's done-checks at `record-batch`/`recover-batch`.
A begun batch that never reaches a terminal record (abandoned, or the run ends or escalates first) never runs them, and `begin-batch` does not require a predecessor to be terminal.
The run still cannot end done: `verifyEveryBatchDone` (`runlevel.go`) refuses a done outcome while any batch is begun but not recorded terminal.
The bound is sound but incomplete as worded.

**Fix:** Add that a batch that never reaches a terminal record keeps the run from an outcome of done (the run-exit every-batch-done check).

### [LOW] D5's bound hides the deleted-test hole in Open risks

**Location:** `decision-record.md`, D5 Bound, fourth bullet; Open risks, first item.

**Issue:** "The post-fix triage deciding success" reads as a full check, but triage only sees failures, so a deleted or skipped test reads as cleared.
The attack-surface rule asks the Decision to state what it lets through; D5 leaves it to Open risks.

**Fix:** Move the hole and its bounds (prompt prohibition, summary.md recording the fix commits and cleared identities, later review rows) into D5's bound, and drop the duplicate from Open risks.

### [LOW] Auto-mode assumptions carry narrative and two orphaned requirements

**Location:** `decision-record.md`, `## Auto-mode assumptions`.

**Issue:** Applying the completeness-before-leanness test item by item:

- "D1 over the board's …", "rather than making `rebaseline` canonicalization-aware" and "not an in-session fork or a Webster-Review route" are rejected-alternative narrative, already in `support-log.md`'s Rejected alternatives.
- "No new config key" is in no Decision.
- "`recovery_timeout_min`" names the key D5 only calls "the recovery timeout from webster config".
- "Any regression verdict, card hint only when localized" and "no migration" are already in D5 and Scope Out; "standalone mode has no bisector" is a useful reason for the hint being optional and is nowhere else.

**Fix:** Move "no new config key", `recovery_timeout_min` and the standalone note into D5, and cut the section to a list of which Decisions were autonomous picks, without the rejected alternatives.

### [LOW] D5's escalation wording leaves the bisect order open

**Location:** `decision-record.md`, D5 Decision, last line.

**Issue:** The fix uses the bisected card as a hint, so bisect runs before the strand; "escalates exactly as today (bisect, …)" then reads as a second bisect, which Open risks' last item says cannot attribute the fix commits.

**Fix:** Say the escalation records the pre-fix bisect result under `-1`, without bisecting again.

### [NIT] Merge-only wording in D5's head check

**Location:** `decision-record.md`, D5 Bound, third bullet.

**Issue:** "HEAD moved past the pre-fix head by the strand's own non-merge commits under the existing merge-only rule" mixes two rules; the merge-only rule tolerates merge commits between a report's `head_sha` and HEAD, while the acceptance criterion fails the attempt for anything but the strand's non-merge commits.

**Fix:** Split it: the strand's commits run from the pre-fix head to its report's `head_sha` with no merge commit among them, and HEAD is reconciled against `head_sha` under the existing merge-only rule.
