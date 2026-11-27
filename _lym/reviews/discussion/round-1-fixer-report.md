---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed-differently
    reason: "The proposed synchronous list keeps hubreconcile, but its board push is a detached boardengine.SpawnSync spawn that the walk never waits on (internal/hubreconcile/walk.go:218, 231-233), so under the by-kind rule it moves to the pusher and D1 leaves it off the synchronous list."
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed
  - id: F18
    disposition: fixed
  - id: F19
    disposition: fixed
  - id: F20
    disposition: fixed
  - id: F21
    disposition: fixed
  - id: F22
    disposition: fixed
  - id: F23
    disposition: fixed
  - id: F24
    disposition: fixed
  - id: F25
    disposition: fixed
  - id: F26
    disposition: fixed
  - id: F27
    disposition: fixed
---

## Changes

All changes are in `_lym/discussion/decision-record.md` unless the entry names the support log.

- F1: D5 opens with the owner verdict on origin: the owner checks' reconcile, `syncParent`'s fetched tips read and `handedOverStop` stop as `git-transport` while origin is unreachable, and the landing merges once origin answers.
  The Goal's landing sentence and the D5 criterion now say this, and the D6 criterion pins the `syncParent` read too.
- F2: D5 lands on a local parent ahead of origin by only owed tips, as on an equal parent; a new D5 criterion pins the second landing.
- F3: the second option of the fix: the parent target's pusher makes the `done` board write once the parent push landed and removes the landing entry after it.
  D1 states that the board target's rules are unchanged.
- F4: D3 keeps a refused target's entry, marked refused, until its refused-push record clears; the dobby's ahead check writes no entry for it; a new D3 criterion pins the bar and the all-clear.
- F5: D6 names `shedrun.DriverLock` in the wait-record rules, states that a waiting run holds no run lock, and the D6 criterion reads the driver lock.
- F6: D5 records the landing entry on every pass that finds the local parent holding the work while origin's lacks it; the run reader's past-the-merge read covers that state; the resume criterion covers the crash.
- F7: the landing entry holds the run's records commit; the pair target pushes nothing past it until the parent push landed.
- F8: D5 defines the parent target's owed push once, as its landing entry standing, refused or not; D13 and the pull-request check read it.
- F9: D3's summary also triggers on a wait record kept past the threshold, the owed period counts from the first of either, and the D3 criterion covers a waiting run with an empty ledger.
- F10: D13 states that with git or pushes off the teardown sets no mark, reads no tracking ref and removes the pair as today.
- F11: D1 sorts pushes by kind: the three detached spawn kinds start or wake the pusher, and each synchronous push is listed by the reason its caller reads the result.
  tendercli is gone from the list. Fixed differently: hubreconcile is also dropped, as the frontmatter reason says.
- F12: D2 replaces `PushUntilLanded`'s loop and its ceiling notice, and `pushClassified`'s transport backoff, with one push per interval; the D2 criterion pins no extra attempt.
- F13: D6 says a step the Go driver runs writes no halt note for a network-class stop, and the driver writes one when it stops waiting on a stop that left that class.
- F14: D1 creates a landing target only for the prime's code branch; a beam landing records on the beam pair's target, and D5's "parent target" names either one.
  A new D1 criterion pins it.
- F15: D9 evaluates the rules with `git check-ignore --no-index -z --stdin` in an empty scratch repository holding the base's ignore files for the path chain, names the rule sources,
  never removes a symlink or nested repository, and cites git-check-ignore(1) and gitignore(5) with a pinning test.
- F16: D9 takes its base from the merge record's pre-merge task head and runs before the before-the-merge-in clean check on a resume; the D9 criterion covers the resume and the fast-forward.
- F17: D7 gains a bound citing git-gc(1)'s reflog expiry defaults, a reflog test in `internal/fabricengine`, the known gap, and a way forward for the refusal and the takeover stop.
- F18: the PATTERN-pair-remote change text binds detached pushes only and names the synchronous exception.
  After the gate's `pattern-change-over-cap` finding, the text is shortened under the 280-character rule-text cap;
  "where a target has both" moved out, since the Constraints line on PATTERN-pair-remote already limits the order to targets with both sides.
- F19: a red-first D1 criterion for issue #238 in `internal/fabricengine`, tier1.
- F20: D6's bound names the misconfigured host or proxy case and the GitHub classes that leave the rule at once.
- F21: D7 says the classifier already reads authentication failures and a GitHub 401 as refusals; the criterion pins the pusher's stop.
- F22: D7 says the rule lives in `resetUnnamedTip`, and why the hand-over and destroy moves through `updateRemoteBranch` need none.
- F23: D8 names the prime's and the pair's pushers for each owed failure.
- F24: the reproduction commits on the local code branch before the push.
- F25: D9's bound says every file the base's rules ignore there is removed, settings and secrets included, each removal logged.
- F26: Open risk 1 is dropped; the Auto-mode assumption carries the reason; the support log's Q11 is recorded as an auto-pick.
- F27: D2 names all three keys, and D3 refers to D2 for the notice key.
- Support log: a Round 1 entry under Review rounds, and the D9 line under Rejected alternatives now names the base.

## Self-check

- fix-proposal, BLOCKING: the board `done` hold leaks through the synchronous board pushes. Fixed: the `done` write moved to the parent target's pusher after the parent push (D5, D1).
- fix-proposal, MEDIUM: a refused landing push freezes the board. Fixed by the same move. The pair's hold on the run's own records commit stays, and the settlement rule below lets it clear.
- fix-proposal, MEDIUM: with pushes off the landing entry never goes. Fixed: D5 writes none with git or pushes off; new D5 criterion.
- fix-proposal, MEDIUM: D9's scratch run still reads global excludes. Fixed: D9 passes `-c core.excludesFile=` with an empty file and empties `info/exclude`.
- fix-proposal, LOW: a step outside the Go driver loses its halt note. Fixed: D6 limits the rule to steps the Go driver runs.
- fix-way-forward, MEDIUM: a refused prime code push's way forward (`git push` by hand) never clears the record. Fixed: D5 settles a landing entry whose owed tip origin's parent holds, clearing the record.
- fix-claims, MEDIUM, merged with fix-consistency LOW 3: `syncParent`'s fetch failure is a plain error today. Fixed: D5 states the new `git-transport` marking; the D6 criterion pins it.
- fix-consistency, MEDIUM: D1's wake rule and D5's "done write wakes no board target" contradict. Fixed: that sentence is gone, since the `done` write now wakes the board target like any board write.
- fix-consistency, MEDIUM: the beam twin of every "landing target owes" rule. Fixed: D5 defines the parent target, and D5 and D13 use it.
- fix-consistency, LOW: the ceiling notice's removal stated twice. Fixed: D3's sentence cut.
- fix-consistency, LOW: D1's catch-up reason disagrees with D8. Fixed: reworded to landed, owed or refused.
- fix-consistency, LOW: the support log's D9 rejected-alternative line names the first parent. Fixed.

## Disputed

None.

## Deferred

None.

## Rulings

None.
