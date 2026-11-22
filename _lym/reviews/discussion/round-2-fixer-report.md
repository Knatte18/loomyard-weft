---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: "The proposed read, a pair ahead whenever its local code branch is ahead, was made and then narrowed after the self-check. It counts code commits no transition names yet, so each dobby tick would run D7's reconcile and push work in progress, since `PushPairAnchored` pushes every unpushed code commit once the records side lands (internal/fabricengine/pushanchored.go:106-121). The tick now counts a pair ahead when its local code branch holds the code tip its local records tip's `Warp-SHA` names and origin's tracking ref lacks it, which is the state the review's counterexample leaves. D6 names the tick as the carrier of the restart, as the fix asked."
---

## Changes

- F1, fixed: D7 names the pause the pusher writes as `tendercli.PausePair` (`tenderengine.Pause`), a slug-scoped entry in the hub pause record.
  It defines "a pause covers the run", for D3's and D5's start conditions and D7's own, as that record's coverage read through `pauserecord.ReadCoverage`, as `pauseCovered` reads it.
  The pusher waits for `lym tender resume` to end the entry.
  The sentence on `lym loom pause` is replaced: the driver meets the entry through `runfsm`'s act rule, which reads the coverage as `PauseCovers` and holds the run.
- F2, fixed differently: D5's dobby tick counts a pair ahead when its local records branch is,
  or when its local code branch holds the code tip its records tip's `Warp-SHA` names and origin's tracking ref lacks it.
  Code commits no transition names are not counted.
  The pruned-tracking-ref rule covers the code branch too.
  D6 states that the owner dobby's code-tip settle sees no `code_tip` verdict there, so D5's tick carries the restart of the held tip's push.
  The D5 criterion covers a level records branch with that code tip unpushed, and names no start for a code branch ahead only by unnamed commits.
- Self-check, fixed: D3's start condition and D5's refusal bullet read a refused-push record on a pair's code branch as well as its records branch.
  The D5 refusal criterion now names both sides.
- `support-log.md` gains a Round 2 entry recording these changes.

## Self-check

- fix-way-forward, MEDIUM, a refused code-side push restarts on every tick, since D3's gate reads only the records branch: fixed (D3's start condition and D5's refusal bullet and criterion read the code branch too).
- fix-consistency, MEDIUM, the same gap as the fix-way-forward finding, merged with it: fixed by the same change.
- fix-consistency, LOW, the support log's Turn 5 still says D7 names the pause `lym loom pause` writes: disputed, every fix worse.
  `support-log.md` Turn 5 records what that turn did, and the Round 2 entry records the correction; rewriting Turn 5 would falsify the log's history.
- fix-proposal, MEDIUM, the proposed code-branch read counts work-in-progress commits, so each tick reconciles and pushes code no transition names: fixed differently from the review's proposal, as F2's reason states.
- fix-claims: no findings.

## Disputed

None of the review's findings.

## Deferred

None.

## Rulings

None.
