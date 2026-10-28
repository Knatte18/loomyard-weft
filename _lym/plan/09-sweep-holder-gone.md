# Card 9 — sweep-holder-gone

**Edit:**
- `internal/creelengine#Store.sweepSubjects`
- `internal/creelengine#Store.sweepNeeded`
- `internal/creelengine#Scheduler.sweepLoop`
- `internal/creelengine#Store.releaseGoneHolder`
- `internal/creelengine_test#TestSweep_RemovedHolders`
- `internal/creelengine/doc.go`
- `pattern/PATTERN-daemon-wakeups.md`

**Uses:**
- `plan:internal/creelengine#isChaseable`
- `plan:internal/creelengine#Store.settleUnansweredLocked`
- `plan:internal/creelengine#UnansweredHolderGone`
- `internal/creelengine_test#env.sweepStep`
- `internal/creelengine_test#env.sweepLast`

**Intent:**
A holder that disappears after reading a marked message is found gone (D10, the added-after-Discussion entry for review F1).
`sweepSubjects` widens its set to strands holding a chaseable message, beside those holding unread mail or a queued inject, so `sweepNeeded`, which reads it, keeps the sweep running while any such strand exists, with its existing backoff from `sweepBackoffBase` to `sweepBackoffCeiling`.
`releaseGoneHolder`, run once the two-miss rule finds the holder gone, settles each chaseable message it holds `unanswered` through `settleUnansweredLocked` with `UnansweredHolderGone`, which answers a waiting `--wait-reply` with `OutcomeUnanswered` and posts the sender's notice under the same rule as a chase-limit outcome; no chase goes to the gone holder.
Its unread mail and queued injects are moved as before.

`TestSweep_RemovedHolders` gains rows, driven by `env.sweepStep` and `env.sweepLast`, whose `isSweepWait` card 7 narrowed to the sweep's own waits so the chaser running beside the sweep is never stepped in its place: a strand that read a marked message, posted with a wait, and holds no other mail or inject is removed from the fake delivery's strands, and after the sweep's two misses the message is `unanswered` with reason `holder-gone`, the wait gets `OutcomeUnanswered` and no chase was posted; the same with the holder also holding unread mail, which goes back along its path as before.
`internal/creelengine/doc.go`'s `# Removed holders` section names the widened subjects and the `holder-gone` outcome.
`pattern/PATTERN-daemon-wakeups.md`'s creel removed-holder sweep bullet says the sweep runs while a strand holds unread mail, a queued inject or a read, marked, unanswered message.
The doc comments of `sweepNeeded`, `sweepSubjects` and `sweepLoop` name the same widened run condition.

**ImpactSummary:** The removed-holder sweep gains one subject kind and one settle step; its handling of unread mail and injects is unchanged.
