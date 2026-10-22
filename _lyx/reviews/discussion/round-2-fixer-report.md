# Fixer report — discussion, round 2

## Changes

- **F1 (BLOCKING), fixed as proposed, then widened at the self-check.**
  D6's carve-out now decides the case: a run whose Finalize already merged into the local parent keeps its lock, is not held by the owner pause while Finalize runs, and finishes its Finalize on the old machine.
  The bound names the board-done mark, the parent push and the pull request's close.
  D6 states both orders: the old machine pushes first and the adopter reads the entry ended or takes its already-landed path, or the adopter lands first and the old push is refused and recorded as a failed push.
  The sentence naming `lyx adopt <slug>` as the way forward is gone.
  Fixed differently in one part: the proposal's "the local parent reads diverged at the next hub-scope reconcile, whose stop names the repair" was not kept.
  The lock the stuck run holds makes the prime read busy (`fabricengine/doc.go`: the prime's branches move "only while `OriginSeams.PrimeParentLockFree` finds no live owner"), and the diverged stop's repair is a merge that lands the task twice (`fabricengine/failedpush.go`, `codeDivergedWayForward`).
  Instead, the exemption ends when Finalize stops, so the next pause read releases the lock, and the halt note names `git reset --keep origin/<parent-branch>` in the parent's worktree.
  The PATTERN-owner-machine bullet, the Goal and D7's Bound carry the exception, and acceptance criteria cover both orders, with the general lock-release criterion qualified.
- **F2 (MEDIUM), fixed as proposed.**
  D5 step 1 stops an owned_elsewhere slug whose pair reads live and says why; `--keep-local` records the owner and `--drop-local` refuses a live pair, naming `--keep-local` (`run-process-ownership`).
  The flag bullet, the envelope line and the adopt-back acceptance criterion name the live case.
- **F3 (MEDIUM), fixed as proposed.**
  D3 now states one rule over InnerRun's `Spawn` seam: every spawn on a run whose status file exists reads an owner refusal as an owned_elsewhere verdict, and only the spawn with no status file is the fresh bootstrap.
  D6's reference and the acceptance criterion are widened to any spawn on an existing run.
- **F4 (MEDIUM), fixed as proposed.**
  D7's run-not-owned and both-fail halts name the `--keep-local` form, and say why a plain adopt stops at D5 step 1, the live-pair stop from F2 included.
  The criterion now says the named call resumes the run, and that a code branch both machines moved stops at the resume's reconcile.
- **F5 (LOW), fixed as proposed, then corrected at the self-check.**
  D3 states the offline bound, and Open risks carries it; both say that either side's push can be the refused one and that only Finalize's reconcile is certain to catch the run.
- `support-log.md` has a Round 2 entry and two question-ledger lines.

## Self-check

- fix-consistency, MEDIUM, the general lock-release criterion contradicts the carve-out: fixed as proposed.
- fix-consistency, LOW, the Goal and D7's title state the landing rule with no exception: fixed as proposed (Goal sentence, D7 Bound line).
- fix-consistency, LOW, `lyx adopt <parent-slug> <slug> --keep-local` also keeps the parent slug's local commits: fixed as proposed (D7 says so).
- fix-claims, MEDIUM, "reads diverged … stop names the repair" fails while the lock is held: fixed with own change (merged with the way-forward finding on the duplicate landing): the exemption ends when Finalize stops, and the halt note names the reset.
- fix-claims, LOW, the board-done mark is best-effort: fixed as proposed (D6 names the already-landed path when the mark is missing).
- fix-way-forward, MEDIUM, the diverged stop's repair merges a second landing: fixed as proposed (reset to origin's tip, never a merge; Open risks names it as a manual step).
- fix-way-forward, MEDIUM, the carve-out run's lock has no release after a refused push: fixed as proposed (released at the next pause read once Finalize stops).
- fix-way-forward, LOW, no outcome for the adopter's run or the leftover owner pause on a landed slug: fixed as proposed (D6 names the ended teardown and `lyx resume <slug>`).
- fix-proposal, MEDIUM, the "one parent push" bound omits the board mark and the PR close: fixed as proposed.
- fix-proposal, MEDIUM, the parent-not-owned adopt on the parent's owner stops at step 1 after F2: fixed as proposed (D7 and its criterion name the stop and its forms).
- fix-proposal, MEDIUM, the offline bound covers only the adopter-pushes-first order: fixed as proposed.
- fix-proposal, LOW, "which lands it" is unmet when a code branch diverged: fixed as proposed (criterion says resumes, and names the code-branch stop).

## Disputed

None.

## Deferred

None.
