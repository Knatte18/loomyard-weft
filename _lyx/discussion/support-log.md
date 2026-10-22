# Support log — batten-cancel-verb

## Interview

The session ran autonomously, so every turn below is a self-pick after exploration; no advisors were seated.

### Turn 1 — scope and verb shape

- Question: one verb with state-chosen behaviour, or abandon (refuses live) and abort (ends live) as two verbs or one flag?
  Recommendation and pick: one verb, `lyx batten cancel`, whose path the run's state chooses; the 2026-10-22 operator decision says the one verb ends a run whatever its state.
- Question: pair removed by default, as the abandon note had, or only with `--remove`, as the abort note had?
  Pick: kept by default, `--remove` to remove; the board brief says "optionally the pair".
- Question: mark batten's record `aborted`, or remove the records?
  Pick: remove; nothing would read an `aborted` state once the records are gone, and git history keeps them.

### Turn 2 — ending a live run

- Question: how to stop strands coming back, as on 2026-10-21?
  Pick: the order pairteardown already uses: `runproc.End` first (batten, driver, loop, pushes), then the loop end, then the session end, so no reviver or report writer is left alive.
- Question: wait for the run lock before ending, as a skipped teardown does?
  Pick: no; a cancel ends the holder, so the run-lock wait moves after the end.
- Question: hold a tender pause during the cancel to stop revivals?
  Pick: no; the end order already stops them, and a pause's grace and Esc passes only slow the verb.
- Question: protect records repositories from a process killed mid-commit?
  Pick: hold the combined records write lock of the prime, and of the pair when present, across the end.

### Turn 3 — refusals and network

- Question: refuse a run whose work reached the parent?
  Pick: yes, naming `lyx batten run <slug>` to finish the landing.
- Question: what with an unreachable origin?
  Pick: refuse, naming `lyx tender pause <slug>` as the hold meanwhile; the board clear needs origin, and a half-cancel leaves a muddier state.
- Question: what under a hub pause, where the prime's network gate waits?
  Pick: refuse naming `lyx tender resume`, rather than wait silently.

### Turn 4 — dirty source

- Question: where does the start check run?
  Pick: in the pre-run before seed, board pull and claim, and in the `--window` caller before the window opens.
- Question: tracked only, or untracked too?
  Pick: tracked only, matching fabric's add probe, so the check and the add never disagree.
- Question: how does a dirty source at Worktree-Create avoid the tackler?
  Pick: a shed precondition reason prefix and a hand-back row beside the budget-exhausted row; the prefix survives a driver restart through the status error, as the budget-exhausted one does.

### Turn 5 — lens review

All seven forks returned; findings judged together and merged across lenses.

- interactions, walkthrough, existing-paths, bound-breaker (merged), MEDIUM: a resume or fresh start could slip between the end and the records removal, and with `--remove` the claim was cleared before the records went — fixed: D4 takes batten's driver, run and status locks right after `End` and D3 holds them through the claim clear; the removal leaves the claim to the cancel.
- bound-breaker, BLOCKING: an `llm` child's driver strand outlives `runproc.End` and could write a report — fixed: D4 stops every live strand of the pair's session through shuttle before the loop end.
- interactions, existing-paths, bound-breaker, leanness (merged), BLOCKING: "no run named" could never hold after a cancel — fixed: D2 now reads no records, no run-form status, no pair and no live registered process; the criterion is narrowed to `--remove`.
- goal-scope, interactions (merged), MEDIUM: a rerun after a kept pair meets the existing branch — fixed: D5 names `lyx fabric remove <slug>` first, in envelope and notice; criterion corrected.
- walkthrough, existing-paths (merged), MEDIUM/LOW: the kept pair kept the teardown's loop mark — fixed: D4 step 7 withdraws it.
- interactions, bound-breaker, existing-paths (merged), MEDIUM/LOW: write-lock bound wrong and acquisition unbounded — fixed: held only around `End`, acquisition bounded, busy-reading callers named.
- bound-breaker, MEDIUM: a `--remove` re-run lost the claim clear — fixed: the verb always makes its own idempotent clear (D7).
- existing-paths, MEDIUM: `lyx batten step` auto-seeds too — fixed: D10 covers every auto-seeding verb.
- goal-scope, MEDIUM: unreachable origin left a live run unstoppable — fixed: D2 goes on with a Warn, as a batten resume does; owner read from the local board.
- goal-scope, MEDIUM: way-forward texts still name `lyx board set-status` for abandoning — fixed: added to Scope.
- board-fit, MEDIUM: a cleared entry reads ready to `umbrella-daemon` and `batten-ready-overlap` — fixed: D7 states it and leaves the hold to the caller.
- board-fit, LOW: source worktree and prime-only wording against `pair-from-parent` and `orch-any-worktree` — fixed in D1 and D10.
- board-fit, LOW: D10 tied to the `--window` flag against `batten-window-default` — fixed: any path that opens a window.
- board-fit, LOW: cancel phase as shared primitive for the umbrella-conversion verb — rejected: that verb is unbuilt and keeps the run's records; naming it here designs for a hypothetical caller.
- interactions, walkthrough (merged), LOW: D3 step-1 order contradicted D2 — fixed.
- walkthrough, interactions (merged), LOW: notice state on a re-run — fixed in D9.
- bound-breaker, LOW: tackler strand source misnamed — fixed: the in-flight marker's `StrandGUID`.
- bound-breaker, LOW: hub pause added after the check — fixed: D2 states the bound.
- bound-breaker, LOW: `RemovePair` refusal after the session end unnamed — fixed in D5.
- goal-scope, LOW: Windows bound only in risks — fixed: stated in the Goal and D4, removed from risks.
- goal-scope, LOW: D8 licence and orch rules file — rejected: D8 follows from "for good", and no orch rule names the manual workaround.
- leanness, MEDIUM: Fact bullets duplicated in Constraints — fixed: folded into D4, D10 and D11 as evidence.
- leanness, MEDIUM: Constraints restating PATTERN entries the decisions already apply — fixed: only task-specific ones kept.
- leanness, LOW: Scope mirrored the decisions — fixed: module-level.
- leanness, LOW: Auto-mode assumptions restated decisions — fixed: pointers.
- leanness, LOW: CLI skill note was an exclusion — fixed: moved to Scope Out.
- leanness, LOW: Interview repeats the ledger — rejected: the ledger records picks, the Interview the reasoning, as the spec asks.

## Rejected alternatives

- Two verbs, `abandon` and `abort`: two names for one outcome, and the operator asked for one verb.
- A `--live` flag required to end a live run: the operator decision ends a run whatever its state, and the orch is the usual caller acting on an operator instruction.
- An `aborted` batten state kept on disk: a new state every reader must learn, for records the verb removes anyway.
- Removing the pair by default: loses uncommitted work in the common pre-inspection case; `--remove` with a snapshot covers the for-good case.
- Refusing the whole verb when origin is unreachable: leaves a live run unstoppable offline, the 2026-10-21 symptom; a batten resume already goes on with a Warn.
- Letting `RemovePair`'s own settle clear the claim under `--remove`: clears it before the records go and is lost on a re-run after the pair is gone.
- A tender pause around the cancel: adds a grace wait and Esc passes, and lifting it would send resume lines to sessions the cancel is about to end.
- A typed `OutputPointer` field for the precondition hand-back: needs a new persisted status field; the reason prefix has a working precedent in the budget-exhausted row.
- Folding the dirty check into fabric only: fabric's add runs after the seed and claim, so the run record and claim would already exist.

## Review rounds

### Round 1

Fifteen findings, all fixed in the decision record.

- F1: a pair-less run reads its owner from the pulled Bolt, as batten's start check does, with no reconcile.
- F2: `pairteardown` takes the snapshot before an origin-mode removal too; added to Scope.
- F3: `--remove` archives the task branch's tip on the warp origin and deletes the branch locally and on origin; a kept pair reruns after `lyx batten cancel <slug> --remove`, since `lyx fabric remove` keeps the unlanded branch.
- F4: the cancel phase owns the `End` and calls batten's lock take through an after-end hook, so the PATTERN text holds.
- F5: the reconcile runs judge-only.
- F6: an ended verdict refuses as landed, or goes on when the entry left the board.
- F7: an unreachable origin makes the cancel end-only, with or without `--remove`, since records and claim writes could land on another machine's adopted run.
- F8 and F9: a cancel marker, checked by batten's pre-run before `armSeed` and removed last, bars starts and lets an interrupted cancel finish.
- F10: Windows refuses at once on any live registered process.
- F11: the verb exempts its network gate, as `lyx fabric remove` does, and holds the status lock only around the records removal.
- F12: the split `cleanupSlug` removal takes no lock of its own.
- F13 to F15: lock owners named, the board push placed in the detached sync, the tackler marker path corrected.

### Round 2

Fourteen findings, all fixed in the decision record.

- F1: a second gate in batten's shed `Run` and `Step`, behind the run lock, re-reads the cancel marker and the seed, so an unregistered `step` still in its preflight cannot reseed the run.
- F2: the tackler marker is read after the end and the lock take.
- F3: `ErrPairNotFound` reads as done, and the task-branch archive-and-delete runs whenever a branch is left.
- F4: D4's split keys on the task worktree, as D2's does.
- F5: the cancel marker's liveness is a lock its writer holds, and Windows records whose pid's process was created after their registration read gone.
- F6: the Windows refusal lists processes after the marker is written.
- F7: a start refuses a present task worktree, and Worktree-Create maps the left-task-branch refusals to the precondition prefix, both naming the cancel with `--remove`.
- F8: the ended refusal fires only while batten's run record is present.
- F9: a left task branch counts as something to cancel under `--remove`.
- F10 to F14: the marker check placed ahead of the start owner check, end-only's step 9 actions named, the held-lock refusal reworded for `lyx batten cleanup`, both diverged tips archived, and `Remote` named on the removal.

### Round 3

Five findings, all fixed in the decision record.

- F1: the pre-drive check gains a call site inside batten's pre-run hooks' seeding closures, under the status lock, with the scratch-marker clear moved behind it; the run-lock call site stays for a resume.
- F2: the tackler marker is claimed only for a tackler the driver had recorded; a strand started before `recordStartedTackler` is an open risk, since its prime-side name carries no slug and no waiter enforces its deadline.
- F3: on Windows the cancel re-lists the slug's registered processes after the lock takes and refuses on any left; on Linux `End`'s survivors error does this.
- F4: the start's pre-run refuses a left weft worktree or local weft branch as well as a task worktree.
- F5: the nothing-left refusal names the cancel with `--remove` when a task branch is left locally.

## Question ledger

- Q1 Verb name: auto-pick `lyx batten cancel`, after the task slug and issue.
- Q2 One verb or two: auto-pick one, state-chosen.
- Q3 Pair default: auto-pick kept; `--remove` removes with a snapshot.
- Q4 Records: auto-pick removed, no `aborted` state.
- Q5 Live-run confirmation flag: auto-pick none.
- Q6 Pre-end run-lock wait: auto-pick skipped; the wait runs after the end.
- Q7 Pause during the cancel: auto-pick none; slug pause entries are lifted after.
- Q8 Commit safety: auto-pick hold the combined records write locks across the end.
- Q9 Landed or past-merge run: auto-pick refuse.
- Q10 Unreachable origin and hub pause: auto-pick refuse both; revised after the lens review: an unreachable origin goes on with a Warn, a hub pause still refuses.
- Q14 Race with a resume during the cancel: auto-pick hold batten's prime-side locks from just after the end through the claim clear.
- Q15 `llm` child driver: auto-pick stop the pair's strands through shuttle before the loop end.
- Q11 Notice when the orch is the caller: auto-pick post it anyway.
- Q12 Dirty probe scope: auto-pick tracked only.
- Q13 Worktree-Create hand-back: auto-pick a shed precondition reason prefix and a driver row.
- Q16 Unreachable origin, revised in round 1: auto-pick end-only, leaving records, claim and pair for the cancel once origin answers.
- Q17 Task branch under `--remove`: auto-pick archive its tip on the warp origin, then delete it locally and on origin.
- Q18 Starts during a cancel: auto-pick a cancel marker read before `armSeed`; revised in round 2: read ahead of the start owner check, and re-read behind the run lock by a pre-drive check; revised in round 3: the pre-drive check also runs inside the pre-run hooks' seeding closures.
- Q19 Cancel marker liveness: auto-pick a lock the writer holds, over the pid, which Windows cannot tell from a reuse.
- Q20 Windows reused pid of a registered process: auto-pick the process's creation time against the record's registration time, over its executable image, which a reuse by another `lyx` process matches.
- Q21 Rerun after a kept-pair cancel: auto-pick a pre-run refusal on a present task worktree, since Worktree-Create adopts a complete one without the add, and the precondition halt at Worktree-Create for a task branch left once the worktree is gone; both name the cancel with `--remove`.
- Q22 Unrecorded tackler strand: auto-pick an open risk over stopping the prime's tackler strands by name, which would end other runs' tacklers, since the prime-side name carries no slug.
