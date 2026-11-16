```yaml
lens: way-forward
outcome: reported
```

Way-forward review of the plan's refusal, stop, reset and recovery ends.

**BLOCKING — D1 owed push / card 7 `FailedPushesOwedBy` with card 1's way forward: the owed push's way forward cannot progress.**
- Card 1 makes the `push_owed` refusal say "run &lt;retry&gt; again once the owed push lands". Card 7 then counts any `transport` record on the prime's records branch as owed by every slug ("a records-side record of the prime's records branch, which holds every run's batten record").
- A record is cleared only when a push of that same side and branch lands (`internal/fabricengine#pushJob.landed` → `pushJob.clear`).
- In this part, nothing pushes the prime's records branch once the process that owed the push is gone:
  - The respawns are retired in card 4.
  - Tender's owed intent returns nil in card 9 and leaves the record standing.
  - The catch-up (D4) is in another part.
- Every reader that sets `PushOwed` aside refuses before it acts: `internal/battencli#runAside` is `{ShedPause}`, the dobby uses `Aside{}` (dobbyreads.go:191) and loom's owner check uses `{ShedPause}`. So no run reaches a transition whose push would clear the record.
- Result: one owed tender intent, or one batten process that died while its push was owed, holds every run of the hub, and the refusal names no step that lands the push.
- Fix: the refusal names a verb that pushes the owed side, such as `lym fabric push` from the prime or the owed pair, and/or `FailedPushesOwedBy` counts a prime records record only for the slug that wrote it. A row should reach the refusal from a state where no retry loop runs.

**BLOCKING — D1 "with origin unreachable, `lym batten cancel` acts here at once and owes its push" / cards 8 and 9: cancel stops before it ends anything.**
- Card 8 moves `internal/battencli#commitCancelRecord` onto the synchronous `Push` and says nothing about an owed result.
- `endRunPhase` calls `CommitRecord` before `EndForCancel` (cancel.go:374-379). An error there returns "could not commit the cancel … run … again" with no process ended.
- With origin unreachable, the push is owed and becomes that error. The cancel ends nothing, and today's end-only path (cancel.go:414, `cancelJudgingEndOnly` at :242-250) is never reached.
- Card 9 handles an owed push only in `clearRunPhase`'s `PushRecords`.
- Fix: `commitCancelRecord` treats an `IsPushOwed` error as success-with-owed, and `TestCancelRun` gains an unreachable-origin row asserting `EndForCancel` ran.

**MEDIUM — Card 12 `ReturnToPreviousTransition`: a single-transition run waits for ever.**
- It "reports false when origin's records branch holds a single transition, leaving the run as it is".
- The run's verdict stays `code_tip`, whose way forward is "run again once the code push lands". If the owing machine is dead, the tip never lands.
- No next step is named for that state, and no row of `TestSettleCodeTip` reaches it.

**MEDIUM — Card 11 `adopterFirstRepair`: the way forward names something no operator or agent can run.**
- Card 11 has the note name "`ArchiveParentMerge`'s repair in place of the by-hand reset".
- `ArchiveParentMerge` is a Go function with no verb. The current text (finalize_owner.go:170-176) names runnable git commands and `lym fabric remove`.
- A rejected parent push whose origin lacks the work stays Stuck under a way forward that cannot be performed.
- Either keep the git commands or expose the repair behind a verb.

**MEDIUM — Card 11 `Deps.OwnedElsewhere`: the halt on an unreachable origin has no stated way forward.**
- An unreachable origin "is an error, so the Finalize does not go on unread".
- The card names no halt text, way forward or retry for it.
- The `TestFinalize_OwnerChecks` row only asserts that it "halts".

**LOW — Card 11 `HandOver(owner, subject)` versus card 5's mark match.**
- Finalize's hand-over builds a takeover entry from the Bolt owner, so the mark's `At` is made up.
- Card 5 skips a later hand-over only when the mark names the same machine and `At`. A later rejected push over the real entry therefore runs a second hand-over (processes, archive, notice).
- The skip should match on machine alone, or Finalize should read the entry from origin's records.

**LOW — Card 9 tender owed push: the operator is not told.**
- An owed intent push is logged at Warn and returns nil, so `lym tender pause` reports success with no line saying the intent is not on origin.
- Cancel's output names the re-run; tender's does not.
- The re-run itself does progress: `Record` runs even when `AlreadyPaused` (pause.go:165).

**LOW — Card 6 `HandOver` inside the pair: the removal waits on an actor from another part.**
- The local removal is left to the parent's dobby (D7, another part).
- In this part, a loom-side hand-over leaves the pair standing with no named next step. This is acceptable as out of part, but the halt text should say the pair is left for the parent.

Tool calls made:
- Read cancel.go:228-262 and :360-392, finalize_owner.go:136-178, and grepped pause.go's already-paused path. My context lacked the cancel commit-before-end order, the current repair note text and whether Record runs on an already-paused slug.
- Read pause.go:156-172 and grepped cancel.go's `EndOnly` setter. My context lacked whether Record is gated by `AlreadyPaused` and where end-only is set.
