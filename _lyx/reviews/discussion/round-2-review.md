---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:49
    summary: Decision 4's enforcement rule refuses any net change to a locked entry's non-status field with no exception, which refuses the prune pass that Decision 2's bound and an Acceptance criterion require
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:56
    summary: The refusal's way forward names lyx batten status, which refuses outside the prime and reports persisted shed states, so a refused agent in a task pair cannot run it and no reader can map its output to no run in progress
---

### [MEDIUM] Decision 4's rule contradicts Decision 2's prune bound

**Location:** `_lyx/discussion/decision-record.md:49`, against `_lyx/discussion/decision-record.md:37` and the Acceptance criterion on `prune`.

**Issue:** Decision 4 states the enforcement rule without exception: each entry a write touches is judged by its net change, and "the write is refused when the loaded entry is locked and the net change touches a field other than `status` or removes the entry"; it lists prune among the callers that get this check.
`Store.Prune` (`internal/boardengine/store.go:646`) rewrites every survivor's `DependsOn`, so on a locked entry that depends on a `done` one its net change touches `depends_on`, and Decision 4 as written refuses the prune.
Decision 2's bound and the Acceptance criterion say the same prune passes.
A plan writer who implements Decision 4 literally, as one check in the store's write path, fails the Acceptance criterion; one who follows Decision 2 needs an exception Decision 4 does not state.
The record does list `prune` among the mutating paths (Scope) and does decide it passes (Decision 2, Auto-mode assumptions, Acceptance criteria), so the focus question is answered, but the two Decisions disagree.

**Fix:** In Decision 4, state the exception at the rule: a removal from a locked entry's `depends_on` of a slug that the same write removes as `done` (only `prune` does this) is not a scope change and passes; or move the prune bound into Decision 4 and have Decision 2 point to it.

### [MEDIUM] The refusal's `lyx batten status` check is not runnable where the refusal lands and has no stated reading

**Location:** `_lyx/discussion/decision-record.md:56` (Decision 5), with Decision 3's bound at line 44 and Open risks at line 116.

**Issue:** The record bounds an unlock of a live run solely by the refusal telling the reader to clear the status only "when `lyx batten status <slug>` shows no run in progress".
Two facts make that bound unusable as written:

- Every batten verb runs from the hub's prime worktree only and refuses from a task worktree, the fabric sibling or `_board` (`internal/battencli/cli.go:199`).
  Board writes are accepted from any worktree, and the refusal's most frequent recipient, which Open risks names, is an agent in a task pair recording a finding mid-run.
  The named check refuses for that agent, and the refusal does not say to run it from the prime.
- `lyx batten status` reports the persisted shed state (`running`, `paused`, `blocked`, `failed`, `awaiting`, `done`), or refuses a slug with no seed (`internal/battencli/cli.go:117`).
  "No run in progress" maps onto none of these.
  A crashed process still reads `running`; a run stopped at a failed Worktree-Create reads a halted state that Decision 8 says is resumable, yet Decision 8 also routes its abandonment through this same unlock.
  The reader cannot tell from the record which outputs license clearing the status.

The bound the attack-surface rule requires for the unguarded `set-status` unlock (Decision 3) therefore rests on a check the reader can neither run nor read.

**Fix:** In Decision 5, name where the check runs (from the prime, `lyx batten status <slug>`) and state which outcomes mean no run holds the entry, for example: the slug has no batten seed, or its status is `done`, or the run is halted and the operator has decided to abandon it.
Match the Acceptance criterion on the refusal's message to that wording.

## Focus verdicts

- **`Store.Prune` and the lock.** `prune` is in Scope's mutating paths, Decision 2 decides it passes when a locked entry depends on a done one, and an Acceptance criterion pins it; the remaining defect is F1.
- **`abandoned` and the `"status":"active"` help examples.** Decision 1 names `abandoned` and hand-set words such as `active` as unlocked, the Auto-mode assumptions repeat it, the rejected alternatives record the consequence that motivated the run-status predicate, and an Acceptance criterion covers `done`, `abandoned` and `active`. Not a defect.
- **A claim followed by a failed Worktree-Create.** Decision 8's last bound names the status afterwards (`running · Worktree-Create` stays) and its disposition (resume with `lyx batten run <slug>`, or the manual unlock to abandon), Open risks records it, and an Acceptance criterion pins it. Not a defect.
- **The comparison base of a multi-entry write.** Decision 4 defines it as the entry as loaded under the board lock, judged on the combined change across every step, and names the duplicate-slug batch and the merge with `set_status`; Decision 3 judges its status-only pass by the same net change, and the Acceptance criteria include the clear-then-edit batch. Not a defect.
- **How a refused reader checks that no run holds the entry.** Decision 5 names `lyx batten status <slug>`, Decision 3 and Open risks bound the residual risk; the check itself is defective, F2.
