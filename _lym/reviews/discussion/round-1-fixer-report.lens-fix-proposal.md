```yaml
lens: fix-proposal
outcome: reported
```

Restating my task: take each review proposal the fixer carried out as written, and look for a case where that proposal is wrong or breaks something else in the decision record.

Findings, by the finding id whose proposal they challenge:

- **F4, BLOCKING.** Changed text: D1's act rule, the new first bullet "`act` when the run is `landing` past its parent-side merge, which finishes here under a hold and when another machine owns it".
  The proposal puts this bullet ahead of every `wait` condition, not only `hand_over` and the hold. That breaks the owed-push wait.
  Case: a landing past its parent-side merge on this machine, with origin unreachable and its last transition's push owed. The act rule now answers `act`, so the next producer starts with the push still owed.
  Evidence: D2's first bullet says "The next producer starts only once the push has landed", D2's Bound says "no run acts while origin is unreachable", and the D2 criterion says a transition offline "starts no producer".
  It also stops the act rule holding such a landing under the operator's machine-local hub pause or hibernate's. Today a hub entry is the one entry that still holds a run with a parent lock (`internal/pauserecord/doc.go:52,89`).
  The exemption should cover the hold and `hand_over` only, and leave the owed-push and pause `wait`s ahead of it.

- **F3, MEDIUM.** Changed text: D8's "`lym adopt --release` clears any standing hold, `adopt` or `adopt-fix`, whichever machine wrote it". The new D8 criterion says the releasing machine's "adopt then goes on".
  Case: machine A's adopt is live, at step 7 (the check). The operator releases A's hold from machine B and runs `lym adopt` there. B's step 1 writes B as owner.
  A's adopt reads nothing after its own step 1, so it goes on writing takeover entries naming A (step 3) and runs `orchcli.BeamUp` (step 8) on subjects Bolt now gives to B.
  Evidence: D8 step 1 is the only owner and hold read in the sequence. The criterion "two machines adopting at once end with one owner" rested on the hold refusal, which `--release` now bypasses for a live adopter. No Bound states this.

- **F6, MEDIUM.** Changed text: D3's "A cancel keeps the batten record, in place of today's removal of the run's records", and D1's `cancelled` read from that finish.
  Case: a task is cancelled and later released again under the same slug. Its batten record still holds the finished cancel, so D1 reads `cancelled`, which outranks `starting` and `queued`. `queued` needs "no batten record", so the task can never run again.
  Evidence: today's removal is what lets the slug run again ("`lym batten cancel <slug> --remove` must run before the slug runs again", `cancelcmd.go`). No Decision says who removes or archives a finished cancel's record, or when.

- **F7, MEDIUM.** Changed text: D3's offline `lym batten cancel` ("ends the run's processes and strands and commits the intent; the retry finishes the cancel … once origin answers"), and its Bound "until the owed push lands … another machine reads the run … not cancelled".
  Case: the operator cancels offline, and meanwhile another machine adopts the subject. When origin answers, the owed push meets the takeover, and D4 hands over: "this machine pushes nothing more for it".
  The cancel intent never lands, yet the operator saw the processes end, and the new owner runs the task on. The Bound's "until the owed push lands" never ends.
  D4's hand-over notice names only the machine, not the dropped cancel. The offline pause intent has the same gap.

- **F14, MEDIUM.** Changed text: D7's "A released task's entry with no batten record is dropped too, and the task stays `queued`: it had not started, so its claim resumes nothing."
  Case: the upgrade lands while a pre-upgrade `adopt-hold` stands over a failing check. Running runs get pause intents, but queued tasks lose the entry that kept them from starting.
  No Bolt hold replaces the retired adopt record, so the dobby claims and starts them.
  Evidence: D7's Rationale says "nothing happens in the hub until adopt is done and its failing tests are fixed". The safe fix is to convert a standing `adopt-hold` into a Bolt `adopt-fix` hold at the upgrade, not to drop it.

- **F16, LOW.** Changed text: D8 step 8's "The try count lives in the `adopt-fix` hold", and D7's hold fields "the fix task's slug and the try count".
  Case: one `lym adopt` with two subjects. The hub has one hold with one count, but the limit is per subject ("The try limit is `adopt_fix_tries` of the config stack of the subject's worktree", `internal/adoptcli/fixloop.go:356`), and today's count is per subject too (`readTries(deps.Dir)`).
  A hold with one count can't carry two subjects' counts against their own limits.

The proposals for F1, F2, F5, F8, F9, F11, F12, F13, F15, F17 and F18 held up: I could not build a case that breaks them.

Tool calls: none.
