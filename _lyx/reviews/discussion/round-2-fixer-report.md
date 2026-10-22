# Fixer report — discussion, round 2

Every finding's premise was checked against the code it cites and holds.
All fourteen are fixed in `_lyx/discussion/decision-record.md`, and the round is logged in `_lyx/discussion/support-log.md` (Round 2, Q18 revised, Q19 to Q21).

## Changes

- F1 (fixed as proposed, the gate option): D3 adds a second gate behind the run lock. Batten's shed `Run` and `Step`, once they hold `shedrun.RunLock`, call a pre-drive check batten installs, which re-reads the cancel marker and the seed and refuses, writing nothing. The unregistered `step` case is named in D3 and in a new acceptance criterion. Scope names the hook under `internal/shedengine` and `internal/battencli`.
- F2 (fixed as proposed): the tackler-marker read moves from D3 step 3 to step 5, after the end and the lock take. D4 states that the stop follows that read. The plan-writer note is updated.
- F3 (fixed as proposed): D5 runs `RemovePair` only while some part of the pair is left, reads `ErrPairNotFound` as done, and runs the task-branch archive-and-delete whenever a branch is left. The interrupted-cancel criterion names the `--remove` re-run after step 6.
- F4 (fixed as proposed): D4's split keys on the task worktree. With the worktree gone, the phase runs steps 1, 2, 3 and 6, skipping 4, 5 and 7. Steps 1 and 2 run alone only when no reed session and no part of the pair is left.
- F5 (**fixed differently**):
  - Marker side: the proposed bounds were the marker's age against the cancel's longest bound, or the writer's image. I made neither change. The cancel's network steps (the reconcile, the Bolt pull, `RemovePair`'s pushes) have no stated upper bound, so an age bound would misjudge a slow live cancel. `internal/proc` has no Windows image read (`proc_windows.go`). Instead, the marker's writer holds an exclusive lock beside it, and that lock is the liveness. The OS releases it on death, on every platform.
  - Records side: I made no image comparison. A pid reused by another `lyx` process matches the image, and that is the likely reuse. Instead, a record whose pid's process was created after the record's registration time reads gone, through a new `internal/proc` Windows creation-time read. It prints no `taskkill` line. The bound and its clock caveat are stated, and Scope names `internal/proc`.
- F6 (fixed as proposed): the Windows list moves to D3 step 4, after the marker is written. Its refusal releases the marker's lock and removes only a marker it wrote fresh (see self-check).
- F7 (fixed as proposed, with a correction from the self-check):
  - D5's wrong sentence is replaced. A start now refuses a present task worktree in the pre-run, beside D10's check.
  - With the worktree gone, Worktree-Create maps `*ErrBranchExists` and an `*ErrRemoteLeftover` naming the task branch to D11's precondition prefix. Both name `lyx batten cancel <slug> --remove`.
  - D11's title, body, rationale and bound follow, as do the Goal and two criteria.
- F8 (fixed as proposed): D2's ended refusal fires only while batten's run record is present here. Otherwise the cancel goes on to the removal. A criterion is added.
- F9 (fixed as proposed): with `--remove`, D2 counts a local or origin task branch as something left. D5 states that such a cancel runs only the archive-and-delete. A criterion is added.
- F10 (fixed as proposed): D3 places the marker check in `armAt` ahead of the start owner check, beside D10's dirty check. The start criterion says the refusal comes before the board pull and writes no claim.
- F11 (fixed as proposed): D3 step 5 and D2 state that an end-only cancel runs only step 9's lock release and notice.
- F12 (fixed as proposed): D4 step 2's held-lock refusal names the lock's path, with `lyx batten cleanup` among its possible holders, and the cancel again. D4's survivor paragraph defers to it.
- F13 (fixed as proposed): D5 archives the local and the observed origin tip, each under its own tag, unless one contains the other. The bound covers both.
- F14 (fixed as proposed): D5 runs `RemovePair` with `Remote` set alongside Force and Snapshot.

## Self-check

| Lens | Severity | Finding | Disposition |
|---|---|---|---|
| fix-claims, fix-proposal, fix-way-forward (merged) | MEDIUM | A kept pair's task worktree is adopted by Worktree-Create without fabric's add (`internal/battencli/wire.go:822-829`), so the F7 halt never fires for it | Fix with own change: a pre-run refusal on a start with the task worktree present; the precondition halt stays for the worktree-gone cases |
| fix-consistency | MEDIUM | D2's nothing-to-cancel test omits the reed session D4 now ends | Fix as proposed: reed session added |
| fix-proposal | MEDIUM | The F6 Windows refusal would remove a marker taken over from an interrupted cancel | Fix as proposed: removes only a marker it wrote fresh |
| fix-proposal | MEDIUM | F8's go-on case contradicts D5's never-removes-landed-work bound | Fix with own change: D5's bound is restated to a run here, and an ended pair with no record here is removed with every tip archived. The proposed landed-on-parent narrowing was not made, since the recorded parent lives in a weft worktree that may be gone and nothing here is left to finish |
| fix-consistency | LOW | Scope's `fabricengine` line and the `--remove` criterion still name one archived tip | Fix as proposed |
| fix-consistency | LOW | D11's title, rationale and the Goal read dirty-source only | Fix as proposed |
| fix-consistency | LOW | Auto-mode assumptions miss this round's picks | Fix as proposed: D3, D4 and D5 entries added |
| fix-consistency | LOW | D4's "a lock holder no record names is a survivor of step 2" contradicts reworded step 2 | Fix as proposed |
| fix-claims | LOW | D5's list of pair parts omits the launcher entry `Remove` counts (`remove.go:41`) | Fix as proposed |
| fix-claims | LOW | The step criterion says it waits for the run lock; shed only try-acquires it | Fix as proposed: busy refusal or pre-drive refusal, D3 worded to match |
| fix-way-forward | LOW | A reader can read a just-finished cancel's marker as interrupted | Fix as proposed: the reader re-checks the marker under the shared lock |

## Disputed

None.

## Deferred

None.
