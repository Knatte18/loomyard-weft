---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/discussion/decision-record.md (D7, Auto-mode assumptions bullet 1)
    summary: The baseline as first parent of the first accumulated card SHA can land inside batch 1, so a regression from batch 1's earlier commit is excused as pre-existing; no rule covers an underivable baseline
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (D6, D10, D11)
    summary: failures holds only the first run's failures, so an identity that fails only in the rerun is classified with no entry or tail, and the record never says which run's tail a regression's summary section and StuckReason carry
  - id: F3
    severity: LOW
    location: _lyx/discussion/decision-record.md (D3, Acceptance criteria)
    summary: D3 lists only the triage section and report fields for a Master stuck over a non-regression verdict; the warnings, friction note, and log line are unstated there and the AC omits the summary section
  - id: F4
    severity: LOW
    location: _lyx/discussion/decision-record.md (D2 rationale)
    summary: D2 says a stuck run was never required to write summary.md, but the Master stencil requires it on every outcome; what a stuck run lacks is the summary validation and the run-exit audit
  - id: F5
    severity: LOW
    location: _lyx/discussion/decision-record.md (D2); _lyx/discussion/support-log.md (Rejected alternatives)
    summary: D2 never says what happens to Master's outcome.yaml on a demotion, and the rejected-alternative argument that outcome.yaml would contradict the result applies equally to the chosen demotion
  - id: F6
    severity: LOW
    location: _lyx/discussion/decision-record.md (D9, Open risks)
    summary: D9 leaves unstated how an identity absent, not run, or failing under a different identity at a bisect SHA counts, and the resulting mislocalization risk is not listed
  - id: F7
    severity: LOW
    location: _lyx/discussion/decision-record.md (D4)
    summary: Nothing clears a prior run's fork log, so a fork that skipped the redirect leaves a stale log that Go parses as this run's first-run failures instead of taking the missing-log path
  - id: F8
    severity: NIT
    location: _lyx/discussion/decision-record.md (Auto-mode assumptions)
    summary: Five of the six assumptions restate D5, D6, D7, and D11 and carry nothing the plan writer needs beyond those decisions
  - id: F9
    severity: NIT
    location: _lyx/discussion/decision-record.md (D11, Notes for the plan writer)
    summary: The exact friction.NotePath call shape in D11 and the ParseReport decoder-option note are plan-time implementation facts the writer stencil says not to gather
  - id: F10
    severity: NIT
    location: _lyx/discussion/decision-record.md (D11)
    summary: A RunResult warning per verdict is ambiguous, since a run has one verdict but can record both flaky and pre-existing identities
---

### [BLOCKING] The D7 baseline can sit inside batch 1

**Location:** `decision-record.md`, D7 ("The baseline is the first parent of the first accumulated card SHA in execution order"), and Auto-mode assumptions bullet 1.

**Issue:** `accumulatedCardSHAs` returns each batch's `CardSHAs`.
`record-batch` sets that field to `[]string{actualHead}`, the worktree HEAD after the whole batch (`internal/websterengine/recordbatch.go`), and `recover-batch` sets it to the recovery digest's head the same way.
It is not a per-card commit.
The implementer body says "One commit per card is the norm", not a guarantee, and a fork that commits a card, runs the card's `verify:`, and then commits a fix leaves two commits in batch 1.
The first parent of batch 1's recorded head is then batch 1's own first commit.
A test that batch 1's first commit broke then fails at that "baseline" too and is classified pre-existing, so the run continues over a real regression.
That breaks D7's own guarantee that triage "can only remove halts it can prove unwarranted".
The record also never says what triage does when no baseline can be derived, which the focus asked about.
Webster already records the exact tree the plan started from: every batch's persisted state carries the HEAD captured before its implementer began.
Webster runs batches sequentially in one worktree (package doc), so the first batch's recorded start is that tree regardless of commit count, and parallel batches do not arise.
A parent sync merged mid-run is outside a webster run, since nothing in `internal/websterengine` merges the parent.
A baseline taken before such a merge makes a parent-caused failure look like a regression, which halts rather than excuses.

**Fix:** Redefine the baseline as the HEAD webster recorded before the first batch in execution order began.
When that value is absent or empty, treat the baseline as unavailable exactly as D8 does, with a red rerun yielding regression plus the "baseline unavailable" warning.
Update the matching auto-mode assumption and the support log's interview turn 7 and Q7.

### [MEDIUM] Which run's failures and tails the report and regression carry

**Location:** `decision-record.md`, D6, D10 (`failures: # the first run's failures`), D11 (regression section "with the regressing identities and their tails").

**Issue:** D6 makes the rerun's failure set the head set that D7 classifies.
D10 says `failures` holds the first run's failures.
An identity that passes in the first run and fails in the rerun is therefore a head identity, and possibly a regression listed in `triage.regressions`, but it has no `failures` entry and so no tail.
D11's regression summary section, and D2's `StuckReason`, need tails the report does not carry for that identity.
For an identity that failed in both runs, the record never says which run's tail is kept.

**Fix:** Define `failures` as the union of the head set and the first-run-only identities.
Each entry's tail comes from the rerun when the identity failed there, and from the first run otherwise.
The regression section and `StuckReason` then read identities and tails from `failures`.
Update the D10 comment to match.

### [LOW] Master `stuck` path: which D11 records apply

**Location:** `decision-record.md`, D3 ("the triage section and report fields are still written"), and the AC "A Master `stuck` over a FAILED report still gets triage fields…".

**Issue:** D11 says every non-regression verdict produces all four records regardless of Master's outcome.
D3 then names only two of them, which reads as a narrower rule for the `stuck` path.
The AC checks the report fields and the outcome only, not the summary section.
Master's stencil requires `summary.md` on every outcome, and `AppendIntegrationFailure` already appends to it on today's `stuck` path, so the summary section needs no new rule, only a statement.

**Fix:** In D3, say that D11's recording applies unchanged under a Master `stuck`.
Extend the AC to require the triage section in `summary.md` on that path.

### [LOW] D2 rationale misstates the `stuck` summary contract

**Location:** `decision-record.md`, D2 rationale ("promoting a Master `stuck` to `done` would need a `summary.md` Master was never required to write").

**Issue:** `webster-template-master.md`'s final-action section requires Master to write both `outcome.yaml` and `summary.md` "whether it finished cleanly, got stuck, or was paused".
The real difference is in Go: `mapMasterDone` validates `summary.md` with `summaryparser.Parse` and runs the run-exit audit cross-check only for `done`, and treats the summary as best-effort for `stuck`.
The other two current-behavior claims hold.
`mapMasterDone` finishes before `runIntegrationStage` is called, the stage has an unleased localization phase, and `shedadapters/webster.go`'s `done` case forwards only the summary path.

**Fix:** Reword the rationale: a `stuck` run's `summary.md` is never validated and a `stuck` run never passed the run-exit audit, so promotion would have to add both.
Make the same correction in support-log interview turn 4.

### [LOW] `outcome.yaml` after a demotion

**Location:** `decision-record.md`, D2; `support-log.md`, Rejected alternatives ("outcome.yaml would contradict the result").

**Issue:** After a demotion, Master's `outcome.yaml` still says `done` while `RunResult` says `stuck`.
The record never says whether Go rewrites the file.
The support log rejects promotion partly because `outcome.yaml` would contradict the result, and the chosen demotion has the same contradiction.
`outcome.yaml` is read only by `mapMasterDone` and archived at the next run's entry, so the impact is small, but the plan writer has to pick a behavior.

**Fix:** State in D2 that `outcome.yaml` remains Master's unedited contract file.
The report's `triage.verdict`, the `-1` record, and the summary section are the durable record of the demotion.
Drop the `outcome.yaml` clause from the rejected alternative.

### [LOW] D9 predicate at a SHA where the identity is absent or not run

**Location:** `decision-record.md`, D9, Open risks.

**Issue:** "None of the regressing identities fails" is well-defined only if "fails" means "appears in that SHA's parsed failure set".
The record does not say so.
Read that way, an identity that is absent (the test does not exist yet) counts as passing, which correctly blames the card that introduced the failing test.
It also counts as passing when the identity did not run because an earlier `&&` link stopped the chain, or when its package failed to build and surfaced only the package identity.
In those cases bisect can move past the introducing card.
This is not a contradiction with D7: both use the same membership test, and only the consequence differs.
D7 declines to excuse a failure it cannot prove, and D9 only localizes.
The asymmetry should be stated, not inferred.

**Fix:** In D9, define a regressing identity as failing at a SHA when it appears in that SHA's parsed failure set, and absent or not run as passing.
Add an open risk that a test not run at an intermediate SHA can shift the blame to a later card.

### [LOW] A stale fork log can pass for this run's log

**Location:** `decision-record.md`, D4.

**Issue:** D4's fallback for a missing log assumes a missing log is missing.
The log path is fixed in webster's scratch dir.
If a fork ignores the redirect in a later run, the previous run's log is still on disk, and Go parses it as this run's first-run failures.
With a green rerun, D6 then records those stale identities as flaky.

**Fix:** State that webster removes any prior integration log before the integration prompt is rendered, so a log present at triage time was written by this run's fork.

### [NIT] Auto-mode assumptions duplicate Decisions

**Location:** `decision-record.md`, Auto-mode assumptions.

**Issue:** The whole-command rerun is fixed in D6/D7, opaque never pre-existing in D7, friction for both categories in D11, and the 40-line cap in D5.
The baseline choice is fixed in D7, and its "not the parent head" reasoning already sits in the support log's rejected alternatives.
Under the completeness-before-leanness test none of these carries a requirement the decisions lack, and the question ledger already marks every decision an auto-pick.
The parser-placement bullet is the only one with content found nowhere else.

**Fix:** Replace the five duplicates with one line saying D1–D11 are all auto-mode self-picks, and keep the parser-placement bullet.

### [NIT] Plan-time implementation facts in D11 and the Notes

**Location:** `decision-record.md`, D11 (`friction.NotePath(deps.FrictionDir, "webster-verify-triage")`), Notes bullet 3 (`ParseReport` decodes with `KnownFields(true)`).

**Issue:** The writer stencil forbids gathering exact signatures and interface shapes, since the plan computes them.
D11 hard-codes a call expression, when the requirement is only the note name and the non-empty-dir condition.
Notes bullet 3 records a decoder option whose requirement D10 already carries ("batch reports keep their strict shape").
Notes bullets 1, 2, 4, and 5 name files and functions at module-boundary level, and are acceptable as the optional head start.

**Fix:** In D11, say "a friction note named `webster-verify-triage` in the told friction dir, written only when that dir is set".
Drop Notes bullet 3.

### [NIT] "A warning per verdict"

**Location:** `decision-record.md`, D11 bullet 1.

**Issue:** A run has exactly one verdict, but a `pre-existing` verdict can also carry flaky identities (D6), so "per verdict" leaves the warning count ambiguous.

**Fix:** Write "one `RunResult` warning per non-empty category (flaky, pre-existing), naming its identities".
