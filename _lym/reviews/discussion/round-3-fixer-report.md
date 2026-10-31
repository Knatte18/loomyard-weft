# Round 3 fixer-report — discussion

All changes are to `_lym/discussion/decision-record.md`.

## Changes

- F1, fixed as proposed: D4's Darn-Write missing-record bullet names one route, `lym loom goto --to Record-Write` then `lym loom resume`, whatever the task branch holds, with the reason from `internal/driftengine/doc.go`: a run with commits past the branch point passed Drift-Await, so its scope record is off or holds `awaited_at`, and Drift-Baseline and Drift-Await then pass with no capture.
  The `step` / `goto --to Darn-Write` route is gone from D4 and its acceptance criterion; the Open risks line names the one route.
- F2, fixed as proposed: the darn text of `decisionWayNoRecord` names `lym loom resume` at or before Record-Write, and `goto --to Record-Write` then `resume` on Drift-Baseline, Drift-Await or Darn-Write; the D4 acceptance criterion reaches both.
- F3, fixed as proposed: the differs-from-committed-copy bullet covers both causes: a failed `decision add` commit, whose way forward is `lym fabric commit` then re-running the refused call alone, and any other edit, whose way forward is restoring the committed copy. The D4 criterion covers the first case.
- F4, fixed as proposed: D13 drops the module-level load check. `configengine.Spec` gains a per-key way-forward text that `ValidateLocal`'s retired-key refusal appends, so the local-file refusal names the file, the key and the local `shed.yaml`. `PrepareRunSettings` keeps its own check, after `moveRunSettings` and before `module.Migrate`, using the same Spec text. The D13 criterion and Auto-mode assumption follow.
- F5, fixed differently: D5 replaces Darn-Write's re-prompting `discussion` gate with a validate-only guard that the one validator runs once the session returns. A guard failure halts the run, Stuck, with no re-prompt, naming the findings.
  I did not make the proposed way forward (remove the damaged file, then `goto --to Record-Write` and `resume`). A record written again from the board entry drops every committed `Added after Discussion` entry. D5's history check then refuses it, and D4's missing-historical-entry refusal of `decision add` blocks re-recording the entry.
  The way forward instead restores each failing file, in the `_lym` weft repository, to its latest version that `lym loom validate-discussion` accepts (the committed copy when the damage is uncommitted), commits with `lym fabric commit`, then `lym loom resume`. The route sits in D5, the guard's decision, rather than in D4. D6 says its stop marker halts ahead of the guard, and the D5 criterion reaches the halt and the restore.
- F6, fixed as proposed: D4 names `fsx.AtomicWriteBytes` as the writer, the cleanup removes `.tmp-` leftovers, and "a file that exists is whole" is scoped to a process crash, since the helper does not fsync.
- F7, fixed as proposed: D4 states the closing-line clause (nothing after the run but spaces or tabs) and the up-to-three-spaces indent clause, scoped to top-level fences (see Self-check).

## Self-check

- fix-consistency, LOW, Scope bullet named only how a fence closes: fixed as proposed; it now says a top-level fence opens and closes by CommonMark's rule as D4 states it.
- fix-consistency, LOW, D4 criterion lacked the appended `## PATTERN changes` bullet case: fixed as proposed.
- fix-claims, MEDIUM, and fix-way-forward, MEDIUM (merged duplicates): re-running after `lym fabric commit` could append the failed call's entry twice. Fixed with my own change: the bullet says the commit records the failed call's entries, and only the refused call is re-run, never the failed one. I did not drop the re-run as fix-claims proposed, because the refused call is a new call whose entry was never appended.
- fix-claims, LOW, and fix-proposal, LOW #4 (merged duplicates): the CommonMark claim was still wider than the stated rule. Fixed with my own change: D4 adds the info-string clause and scopes the rule to top-level fences, since the parser reads no list item or blockquote. Scope and D4's Bound now cite D4's rule.
- fix-proposal, MEDIUM #1, the goto route was wrong when the working record was removed but `_lym` HEAD holds it: fixed as proposed. The missing-record refusal at Darn-Write, and `decisionWayNoRecord` on a later row, name restoring from HEAD in that case; the D4 criterion reaches it.
- fix-proposal, MEDIUM #2, the new fence rows sat inside Record-Write's outer fence and could not detect the clauses: fixed as proposed. The rows moved to the D5 criterion, on a Discussion-written record with top-level fences.
- fix-proposal, LOW #3, `lym fabric commit` could commit a hand-appended entry that bypassed the caller rule: fixed as proposed. The commit case is bounded to entries a failed call's commit-failed refusal reported, and any other appended entry takes the restore.

## Disputed

None.

## Deferred

None.
