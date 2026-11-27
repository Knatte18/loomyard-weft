---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed-differently
    reason: "The proposed rule that the landing target runs only once the dobby's catch-up has run after an absence was not made. That catch-up is run-scoped and runs before a run starts (Goal :14; pushOwedForCatchUp(prime, slug) in internal/battencli/catchup.go), so no landing target ever triggers it, and the landing push and done writes would wait forever. Instead, the landing target pulls the Bolt from origin through boardengine.PullBolt before each owner read, so no push or write rests on a verdict read before an absence. A parent owned elsewhere writes a refused-push record (D7), not a bare notice, so dobby ticks do not restart the pusher or repeat the notice (runpusher/start.go:118-130). Its way forward is named, including the case where origin's parent diverged meanwhile."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
---

## Changes

All changes are in `_lym/discussion/decision-record.md`, plus a Round 2 entry in `_lym/discussion/support-log.md`.

- F1:
  - Scope In names batten's Land row and every other pull of the prime.
  - D5 makes every pull of the prime read a local code branch ahead of origin by only owed tips as level, `lym adopt`'s deploy included.
  - Batten's Land row reads an unreachable origin at its pull as `git-transport`, whether or not a push is owed.
  - A tier1 D5 criterion in `internal/battencli` or `internal/battenshed` covers both cases.
- F2:
  - D5 defines one landing entry per landed run: the run, its tip, its held records commit and its pending `done` write.
  - A later landing adds its own entry, and the parent target pushes the newest tip.
  - Entries settle in landing order once origin's parent holds their tips.
  - In D3, a parent target's ledger entry stays while any landing entry stands.
  - The second-landing criterion pins both `done` writes and both releases.
- F3:
  - D5's hold binds every push of the run's pair records, and D1 and D8 state the same.
  - D13's Worktree-Teardown and the dobby's landed-pair teardown both remove the pair only once the run's landing entry is gone.
  - The batten record option taken was the narrowed claim. Batten's record sits in the shared prime records and is not held. A reader of it at Land or Worktree-Teardown, the run reader's local read included, reads past-the-merge only when the parent it judges holds the work.
  - Criteria cover a catch-up during an owed push, and Worktree-Teardown with tips on origin but the entry standing.
- F4 (fixed differently; the reason is in the frontmatter):
  - D5 and the PATTERN-owner-machine constraint give the parent target's pusher an owner read before each parent push and each `done` write.
  - A parent owned elsewhere is a refusal with a record, whose way forward is `lym adopt` then `lym fabric push`, plus a merge through the prime's worktree when origin's parent diverged.
  - A D5 criterion covers the case.
- F5:
  - D3 stores no refused mark: a target reads as refused from its own branches' refused-push records, read per target, the prime's code branch included.
  - A dobby tick drops any entry whose target its ahead check finds level, whoever pushed it.
  - The D3 refused criterion covers a hand `lym fabric push` clearing the record.
- F6:
  - D9 takes its base from the pre-merge task head the merge-in pass reads, and writes the merge record before the after-the-merge-in clean check.
  - The D9 criterion adds a first merge-in with no earlier record.

## Self-check

- fix-consistency, MEDIUM: D13's dobby teardown condition and its criterion disagreed with Worktree-Teardown's new landing-entry condition. Fixed as proposed: both now read "the parent target holds no landing entry of the run".
- fix-consistency, LOW: the batten-reader sentence pointed at "the next paragraph". Fixed as proposed: it now names the past-the-merge paragraph below. Merged with fix-proposal's LOW finding 4.
- fix-consistency, LOW: D1 :46, the D1 beam criterion and the first D5 criterion still said "owed tip". Fixed as proposed: they now say "landing entry".
- fix-consistency, LOW: D1's synchronous-push list and D8 did not state the records hold. Fixed as proposed: D1 gains one sentence, D8 states how the catch-up treats held commits, and D5 refers to D8 instead of restating it.
- fix-claims, LOW: the run reader reads the prime's local batten record, not origin's. Fixed as proposed: the sentence names the local record or origin's.
- fix-proposal, MEDIUM: `fabricengine.RefusedPushStands` skips the prime's code branch (`pusherreads.go:162-178`). Fixed as proposed: D3 reads refusals per target, naming the gap, and a D5 criterion pins a refused landing push in the summary.
- fix-proposal, MEDIUM: the catch-up gate for the landing target is never triggered. Fixed as proposed: it is replaced by a Bolt pull before each owner read. This is the F4 fixed-differently reason.
- fix-proposal, MEDIUM: Land's transport rule held only while a push was owed. Fixed as proposed: it holds whether or not a push is owed, and the criterion covers both.
- fix-proposal, LOW: the wrong paragraph pointer. Merged with fix-consistency's LOW finding above.
- fix-way-forward, MEDIUM: adopting back did not lead to a push when origin's parent had moved on, and adopt's deploy pull stopped. Fixed as proposed: the rule now covers every pull of the prime, `lym adopt`'s deploy included, and the diverged case has a named way forward.
- fix-way-forward, MEDIUM: nothing kept the pusher stopped or the notice to one. Fixed as proposed: owned-elsewhere writes a refused-push record, and the criterion pins that later ticks start no pusher.

Every fork returned a report.

## Disputed

None.

## Deferred

None.

## Rulings

None.
