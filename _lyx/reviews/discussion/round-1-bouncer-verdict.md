---
verdict: APPROVED
rationale: "The reviewer approved with eight findings (F1-F8), and the current decision-record.md resolves all of them: D2 now states its bound, D3's bound credits PlanEditError and run entry's fingerprint compare, D5 keeps rejected commits unreset and records the pre-fix bisect under -1, D1 names the run-exit every-batch-done check, and Auto-mode assumptions is cut to a statement of which picks were autonomous. An independent pass found no unbounded edge or guard, and nothing that belongs in support-log.md."
---

# Round 1 verdict: APPROVED

The round-1 report has no `## Focus departures` section, so there is nothing to ratify or reject.

## Findings against the current artifacts

Every finding was checked against the decision record as it stands now:

- **F1 (D2 unbounded):** resolved.
  D2 now has a Bound: begun non-terminal cards lose their resolve pass in `validate`, bounded by the first `begin-batch` validation, the pinned `CardHashes` and the `record-batch`/`recover-batch` done-checks.
- **F2 (D3 misattributed guard):** resolved.
  The bound names the foreign-edit check (`PlanEditError`, or run entry's fingerprint compare) as what refuses a foreign edit, and states that `rebaseline` never adopts an edit to a begun card.
  The hash comparison is now described only as keeping an already-stale card refused.
- **F3 (D5 rejected commits):** resolved.
  "Go never resets the branch", and summary.md and the way-forward name the pre-fix head and the commits.
- **F4 (D1 abandoned batch):** resolved.
  The bound names the run-exit check that refuses a done outcome; `verifyEveryBatchDone` exists in `internal/websterengine/runlevel.go`.
- **F5 (deleted-test hole):** resolved.
  The hole and its mitigations now sit in D5's bound, and Open risks no longer duplicates them.
- **F6 (Auto-mode narrative):** resolved.
  "No new config key", `recovery_timeout_min` (a real key in `config.go`) and the standalone-mode note have moved into D5, and the section is two sentences.
- **F7 (bisect order):** resolved.
  The text now reads "recording the pre-fix bisect result under the reserved `-1` record without bisecting again".
- **F8 (merge wording):** resolved.
  The non-merge rule over pre-fix head..`head_sha` and the merge-only reconcile of HEAD are now stated separately.

## Independent pass

- **Attack surface.**
  D1, D2, D3 and D5 each state what they let through and what bounds it.
  D4 is a prompt rule inside the existing self-fix cap.
  It removes no guard: if a fix edits files outside the card's targets, the fork still lists them in `deviations`, and `ScopeGuard` still checks them over the `record-batch` delta.
  D6 narrows exposure rather than widening it.
- **Relocation.**
  Rejected alternatives are in `support-log.md`.
  The Decisions carry no deliberation narrative a plan writer could do without.
- **Symmetry.**
  The writer stencil says not to gather exact signatures or `file:line` citations.
  The optional Notes for the plan writer name functions as a head start, which the stencil allows, so nothing is flagged as missing.
