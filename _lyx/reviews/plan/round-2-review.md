---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/05-landing-pre-merge-clear.md
    summary: card 5 adds a Stuck reason for an own leftover whose MergeAbort fails, but the premerge_test.go table it plans has no row that reaches it, against PATTERN-refusal-way-forward
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/plan/06-resume-start-own-leftover.md
    summary: card 6 adds a refusal when the recorded parent branch cannot be read over a parked merge, but plans no test row for it although TestRunDriverSpawnAndWait_MidMerge_ProbeErrorRefuses covers the sibling probe-error path
---

### [BLOCKING] The abort-failure Stuck reason is reached by no planned test

**Location:** `_lyx/plan/05-landing-pre-merge-clear.md`, the `preMerge.clear` decision paragraph and the `premerge_test.go` row list.

**Issue:** `preMerge.clear` gains a Stuck reason of its own for the row's own leftover when the stop succeeds but `AbortMerge` returns an error: `the parked merge-in of parent branch %q could not be aborted: <err>; way forward: conclude it with "lyx fabric merge --continue" or discard it with "lyx fabric merge --abort", then resume`.
The card also widens the `Publish or Finalize: merge in progress` refusal-spec row to name this trigger ("a parked merge-in the row's own abort could not discard").
The table the card plans for `premerge_test.go` has these rows: own leftover with no live session, own leftover with a live session, a failing stop, another verb or source, foreign state, a nil seam, a probe error, and Finalize's retry pass.
None makes `AbortMerge` fail, so no test reaches the abort-failure reason.
`PATTERN-refusal-way-forward` requires that a new refusal in a bound module "lands with its row in the same commit, and a test reaches it", and the decision record's Constraints section repeats this for D2.
The other-verb row reaches the same spec row with a different message, so it doesn't pin this branch.
That branch is also where a regression would matter most: a stop has already succeeded, and the merge must stay untouched.

**Fix:** Add a row to the planned `premerge_test.go` table: own leftover, stop succeeds, `AbortMerge` returns an error.
The row asserts a Stuck reason that names the parent branch, `lyx fabric merge --continue` and `lyx fabric merge --abort`, with no path list, and asserts that no merge-in follows (the fake resolver is not called).
This adds no target, because `premerge_test.go` is already a `Create` target on the card.

### [LOW] The parent-branch read error in the resume/start guard has no test row

**Location:** `_lyx/plan/06-resume-start-own-leftover.md`, the `refuseOverUnfinishedMerge` paragraph and the test row lists.

**Issue:** For a parked state, the narrowed guard reads the parent branch through the new `recordedParentBranch` seam.
A read error "refus[es] like a probe error".
This is a new error exit on a boundary read (the pair's origin record), but none of the planned rows makes the seam fail.
`TestRunDriverSpawnAndWait_MidMerge_ProbeErrorRefuses` (`internal/loomcli/start_midmerge_test.go:209`) already pins the sibling probe-error path, so the gap closes cheaply by extending that test.

**Fix:** Extend `TestRunDriverSpawnAndWait_MidMerge_ProbeErrorRefuses` with a case: a parked `merge-in` state, and a `recordedParentBranch` seam that returns an error.
The case asserts the refusal carries the error, nothing spawns, and the bootstrap lock is released.
The file is already an `Edit` target on card 6.

## Focus verdicts

- **Card 5's conflict-session stopper: not a defect.**
  `mergeresolve`'s spawn spec sets `Role: conflictRole` (`conflict`) and no name override (`internal/mergeresolve/spec.go:107`).
  Reed forms the name in `strandNameLocked` by numbering that role past every role held in the pair's own strand table (`internal/reedengine/strand.go:203-216`).
  That table is per pair: the reed geometry is anchored at the pair's `AnchorPath`, with the slug from the worktree name (`internal/hubgeom/hubgeom.go:78-99`).
  Card 5's `conflictSessionStopper` reads that table through `c.reed.Status` and parses each name through `agentname.Parse`.
  It matches the role with card 4's `MatchesRole`, so both `conflict` and `conflict-N` match and `conflicted` does not.
  It stops every match before returning `("", nil)`, and `preMerge.clear` aborts only after the stopper returns nil.
  The `TestConflictSessionStopper` table has rows for `conflict`, numbered `conflict-2`, two live conflict strands, a dead or retiring strand, no conflict strand, a status error and a stop error.
  Card 4's `TestNumberRole` assertions pin the numbered forms and the near-miss cases.
  One narrowing I weighed and did not raise: the stopper skips a retiring strand.
  A retiring strand is one whose `reed remove --detach` remover is already in flight, so its process is already being ended.
- **Cards 5 and 6, stated bounds: not a defect.**
  Card 5's Intent states what the abort discards: any parked parent-sourced `merge-in`, whoever started it, a hand-started one with its partial resolution included.
  It states that the stop ends only conflict-session strands of the run's own pair, that the abort follows only a fully successful stop, and that other verbs, other sources and foreign state are never touched.
  Card 6's Intent states that the guard lets through exactly what card 5 discards, a hand-started one included.
  It states how a conflict session that outlived a dead driver is handled, and that every other row, verb or source and all foreign state keep the refusal.
  Both bounds match D2's and D3's.
- **Card 3's recorded-failure Stuck reason: not a defect.**
  Today's failed reason ends with `fix forward on the task branch, then resume` (`internal/landingshed/verifygate.go:216`).
  Card 3 replaces that trailing clause with the goto-then-resume way forward instead of appending to it, so the recorded case names one route.
  `TestPublishVerify_FailureRecord` is to assert that the reason doesn't contain `fix forward on the task branch`.
  The unrecorded cases (Finalize with no recorder, and a failed record write) keep today's text, which `TestVerifyGate_CheckMapsVerifyResult` pins.

## Other checks that found no defect

- D1's route is live end to end.
  The Webster-Burler row renders the rubric through `shedadapters.ReadRubric` with `env.PublishFailure` (`internal/shedrecipe/entries_burler.go:43-44,212`).
  That is the same note `loomcli` wires from `loomshed.PublishFailureNote` (`internal/loomcli/wiring.go:552-553`).
- Publish commits its status file before the new pre-merge step.
  That commit lands on the records side, which a parked `merge-in` leaves unmoved (card 1's Intent), so it doesn't block the pre-merge step from running.
- The resolver merges `p.deps.ParentBranch` (`internal/landingshed/publish.go:144`), so a record `Source` equal to `parentBranch` is the row's own merge-in, as cards 5 and 6 assume.
- Card 8's claim holds.
  `Attach`, `AttachGated` and `ProbeGated` share `r.attach` (`internal/shuttleengine/attach.go:44-84`).
  The burler half probe reads only `ProbeGated`'s found flag (`internal/burlerengine/resume.go:62`), so a burler row for `stopped` would duplicate the existing not-found rows.
- D4's unsynced-parent case is already the row `a commit on the parent after the fork does not appear` in `TestReadConfigChanges`, so card 2 doesn't need to add it.
- The escalation brief (`bouncer-template-escalation.md`) runs `decision add --by parent` from a one-shot fork of the parent session.
  That fork inherits the parent's strand name, so card 7's check lets it through.
- The verify section runs in the template's order and covers every targeted package, with the `tmux` tier compiled and the `llm` tier compiled.

## Prior-round check

I read round 1's review and fixer report only after this file was saved.
All eight round-1 fixes (F1–F8) are present in the current cards, and none has regressed.
The fixer recorded no deferred or disputed items.
Neither of this round's findings repeats a round-1 key.
F1 here comes from the failure branch that round 1's F1 and F4 fixes created in card 5.
F2 here comes from the seam that round 1's F7 fix brought into the seam test.
