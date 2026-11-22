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
    disposition: fixed-differently
    reason: "The reviewer's change appends the restore note in the fail closure; the fork lenses showed it then lands after the way forward, so the note is prefixed to the cause instead, and the way forward stays last."
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
---

## Changes

- F1: the `BoardWithNoUpstreamIsNeverPushed` expectations build the namespace root and the board branch from `fabricengine.RecordsNamespaceRoot` and `fabricengine.OneRepositoryBoardBranch`.
- F2: `refuseUnseededBoard` fetches origin's board, then calls origin's board this hub's seed whenever it shares history with the local tip (`sharesHistory`, a `git merge-base` that reads exit 1 as no common history).
  It names the join only for an unrelated board.
  The doc comment and the `doc.go` paragraph state the new rule, and the descendant row covers it.
- F3: rows for a committed board facing a seed ancestor (with a successful push after the settings), an unreadable origin, and a committed board facing an unrelated origin board.
- F4: a board push that ended unanswered and was not withdrawn wraps `gitrepo.ErrLeaseRejected`, so `seedPushCause` names the join; the new row `BoardMovedToAnotherTipBeforeTheKill` moves the board ref in the hook before the kill and asserts the other board survives.
  `killPushClientHook` takes a `beforeKill` shell snippet for that.
- F5: the board cause no longer claims the restore; `fail` prefixes "the records branch was returned to what origin held before" only when the restore succeeded, and keeps the also-failed clause otherwise.
- F6: the `CurrentSHA`, merge-base and `PrimeRecordsBranchKey` read errors carry the refusal and a way forward; a failed fetch is named as a failed fetch.
- F7: an unborn board reads "origin holds another board at <tip> while this board holds no commits", with the join way forward.

Commits: one for F1, F2, F3, F6, F7 (shared files), one for F4, F5 (shared files), then two follow-ups from the self-check.
The gate's reach findings named `pushCloneSeed`, `withdrawUnansweredSeedPush` and `sharesHistory` as reached by no mapped test.
The tests that reach them already exist, so no test was added: the coverage map of `internal/fabricengine` was stale, and `lym gate test --map internal/fabricengine` rebuilt it.
A last commit repairs the earlier comment findings of the gate: the F2 comments in `doc.go` and `pushclassified.go` are rewritten one sentence per line, with none wrapped mid-sentence or over 119 characters.

## Self-check

- fix-consistency, LOW: the `pushCloneSeed` doc, the `doc.go` paragraph and the killed-push test doc omit the other-board outcome. Fixed (F4, F5 follow-up commit).
- fix-proposal, LOW: F5's note lands after the way forward. Fixed differently (note moved ahead of the cause).
- fix-code-correctness, LOW: the same F5 wording point. Merged with the fix-proposal finding, fixed the same way.
- fix-proposal, LOW: F6's fetch failure message names an unreadable records store. Fixed (own message for the fetch).
- fix-proposal: no finding for the shared-history rule, the `ErrLeaseRejected` reuse, or the added fetch.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
