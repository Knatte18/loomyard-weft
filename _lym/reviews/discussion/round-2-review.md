---
verdict: BLOCKING
findings:
  - id: F10
    severity: BLOCKING
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md#Open risks
    summary: The Open risks bullet on parent commit 7c23591 still says the run's parent decides whether the arm ignores or aligns with it, and the support log's Q6 is still open, though both added entries settle it and the arbiter's entry orders the bullet removed and Q6 marked settled.
    fix: In _lym/discussion/decision-record.md, delete the first bullet under ## Open risks whole, the one that begins with Parent commit `7c23591`, including its indented second line; keep the bullet on the hand `lym loom start` race. In _lym/discussion/support-log.md's ## Question ledger, change the Q6 line to read that Q6 is settled by the parent's added entry, so this bench arm designs against base f98a1ba0 and ignores commit 7c23591. Leave both Added after Discussion entries untouched.
prior:
  - id: F1
    status: resolved
  - id: F2
    status: resolved
  - id: F3
    status: resolved
  - id: F4
    status: resolved
  - id: F5
    status: resolved
    answer: "Settled by the added entry titled Ignore commit 7c23591 (F5), and restated by the added entry titled This bench arm designs against its own base and ignores parent commit 7c23591."
  - id: F6
    status: resolved
  - id: F7
    status: resolved
  - id: F8
    status: resolved
  - id: F9
    status: resolved
---

### [BLOCKING] Open risk and Q6 still ask the parent a question both added entries settle

**Location:** `_lym/discussion/decision-record.md#Open risks`, with the Q6 line of `_lym/discussion/support-log.md`'s Question ledger.

**Issue:** The record now carries two added entries, `Ignore commit 7c23591 (F5)` and the arbiter's `This bench arm designs against its own base and ignores parent commit 7c23591`.
The arbiter's entry orders the first Open risks bullet removed and Q6 marked settled.
The bullet still says "the run's parent decides whether this bench arm ignores that commit or aligns with it", and Q6 still reads "open, for the run's parent".
The plan writer reads the decision record, so an open question the parent has already answered sits beside its answer.
The rubric names this case: an Open risks line an added entry settles is a `consistency` finding whose fix drops the line.
It sits on this round's change, the added entries that settle it, in a file the changes file lists.

## Prior findings

- F1: D3 now reads `reed.json` through `reedengine.LoadState`, names the window from the strand's group and the `WindowGroups` shared record, and pins both reed sites; a criterion covers `window_share_max` 0.
- F2, F3: D1's re-add and the bootstrap's add both go through `AddStrandUnless`, a skip counting as done; the Constraint and criteria match.
- F4: the re-add runs whatever the runs sidecar write returns, and a failed replace names `lym loom start`; both paths have criteria.
- F5: settled by the two added entries; its leftover Open risks line is F10.
- F6: the lock Constraint names reed's shared read lock through `ReadJSON`.
- F7: D1 states the `wireCLI` evidence, and the support log credits `wireCLI`.
- F8: the failure criterion covers the re-add and its sidecar write.
- F9: the structural clause is gone from the D2 criterion.

## Rejected

- walkthrough LOW (D2 and the Goal claim a recipe-named window that is `loom-status` at `window_share_max` 0): not BLOCKING, so an incremental round cannot raise it; D3 and its criterion already handle that case, and the default config gives the recipe name.
- interactions LOW (the bootstrap's replace path logs a wrong warning when it loses a race to D1): not BLOCKING, so an incremental round cannot raise it; the checklist is restored and only a log line is wrong.
- bound-breaker B1 MEDIUM (following the `lym loom start` way forward also resumes or respawns a halted pawl run's driver): not BLOCKING, so an incremental round cannot raise it. It fires only on a reed failure path, and the record's Scope already names `lym loom start` as today's way to restore the checklist.
- bound-breaker B2 LOW (Scope Out and a Rejected alternatives line claim `lym loom resume` re-adds the checklist, but only `lym loom start` does): not BLOCKING, and it sits on an unchanged Scope line, so an incremental round cannot raise it.
- existing-paths: reported no findings.
