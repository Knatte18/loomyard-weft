# Fixer report — discussion, round 4

## Changes

- F1 (MEDIUM), fixed differently: D3 now stops `episodeStuckCount` at a `budget_capped` entry unless the entry right after it is an `OutcomeGoto` entry.
  The proposed test was a goto entry anywhere after the capped entry, which I did not adopt.
  The fix-proposal fork showed that test wrong: after a resume's grant, any later admitted goto would make the walk pass the capped entry and count the pre-halt Stucks again, which takes the grant back.
  Evidence: `internal/shedengine/run.go:545-574` skips `OutcomeGoto` entries, and only `Goto` and `Relaunch` append history entries outside a step (`internal/shedengine/goto.go:99,204`).
  D3 now says a resume writes no history entry, so the row's next run follows the capped entry, and that a goto onto the halted run grants no fresh budget, nor does a resume after it.
  D1's list of episode enders, D3's Bound, the D3 criterion and the plan writer's note all agree with this.
  A new criterion pins the goto's non-grant, the resume's grant, and a later goto keeping the grant's spent count.
- F2 (LOW), fixed as proposed and extended: D5 writes the generation record at every seed, generation 1 included, and on every `approveWithoutReview` path of a directory that holds none, so a directory without one is an older binary's.
  Case 3 is now an unharvested verdict, a settled verdict with no settle record under a generation record.
  Case 4 keeps today's clear for a directory with no generation record.
  Each judge spawn records the subject identity and base in the generation record.
  Case 3 harvests only when the live subject equals that identity, and otherwise becomes case 2 against the recorded base, which D6 now names.
  D5's liveness test reads a settle record with no generation record as generation 1.
  The standalone-run sentence now points at case 4, as do the clear's owner sentence and the Open risk, which also names the unharvested verdict in an older binary's directory.
  The no-settle-record criterion is narrowed to no generation record, and a new criterion covers the unharvested case.
- F3 (LOW), fixed as proposed, with one clause dropped: D5's code-segment bullet now says the told list names exact worktree-relative paths of harness-written files, never a pattern or a directory.
  It exempts a path only while it is untracked, and a change confined to listed files reads as unchanged, bounded by the verify tiers.
  I dropped the proposed "which run over the whole tree", because the verify gates run the diff's impacted set (commit `01ccaf6d`, `contracts/recipes/loom-recipe.yaml:453`).
- `support-log.md`: added the Round 4 entry.

## Self-check

- fix-consistency, MEDIUM: an `approveWithoutReview` settle left a settle record with no generation record, so the liveness test had no generation to read. Fixed as proposed: the record is now written on those paths.
- fix-consistency, LOW: D3's Bound and its criterion promised the resume's grant unconditionally. Fixed as proposed.
- fix-consistency, LOW: the plan writer's note named `rerouteCount` as the model without the goto condition. Fixed as proposed.
- fix-way-forward: no findings. Its outside-lens note, that an older binary's directory clears an unharvested verdict, was fixed as proposed in the Open risks.
- fix-proposal, MEDIUM: under F1's rule, a goto after a resume's grant took that grant back. Fixed differently: the test is now a goto directly after the capped entry, not one anywhere after it.
- fix-proposal, MEDIUM: case 3 approved a subject that changed after the verdict. Fixed differently: the proposal suggested a judge-recorded identity or a no-goto condition, and I adopted the judge-spawn identity in the generation record, with case 2 on a mismatch.
- fix-proposal, LOW: a settle record with no generation record had no generation for the liveness test. Fixed as proposed: it reads as generation 1 for that test.
- fix-claims, MEDIUM: "which run over the whole tree" is false for the diff-scoped verify gates. Fixed as proposed: the clause is dropped.

## Disputed

None.

## Deferred

None.
