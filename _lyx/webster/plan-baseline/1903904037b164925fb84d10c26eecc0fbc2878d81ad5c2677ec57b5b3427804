# Card 2 — creel-orch-receiver

**Edit:**
- `internal/creelengine#Store.checkMayAct`
- `internal/creelengine#Store.Hold`
- `internal/creelengine#Store.deliverMailLocked`
- `internal/creelengine#InboxResult`
- `internal/creelengine#Store.Inbox`
- `internal/creelengine#deliverableReceiver`
- `internal/creelengine#Scheduler`
- `internal/creelengine#worker.remind`
- `internal/creelengine#composeReminder` -> `func composeReminder(unread []Message, holdOwner string) string`
- `internal/creelengine#pool`
- `internal/creelengine#newPool`
- `internal/creelengine#pool.apply`
- `internal/creelengine#eventWithheld`
- `internal/creelengine#record`
- `internal/creelcli#inboxCmd`
- `internal/creelengine_test#reminderFor`
- `internal/creelengine_test#TestStore_Mail`
- `internal/creelengine_test#TestStore_InjectAndHold`
- `internal/creelengine_test#TestScheduler_Reminders`
- `internal/creelengine_test#TestScheduler_Injects`
- `internal/creelengine_test#TestSweep_RemovedHolders`
- `internal/creelengine_test#TestDaemon_Lifecycle`
- `internal/creelengine#TestJournal_Fold`
- `internal/creelengine/doc.go`
- `internal/creelcli/doc.go`
- `tools/sandbox/SANDBOX-CREEL-SUITE.md`

**Delete:**
- `internal/creelengine#Store.appendWithheldLocked`
- `internal/creelengine#withheldEntry`
- `internal/creelengine#noticesName`
- `internal/creelengine#reminderSubject`
- `internal/creelengine#maxReminderSubjectRunes`

**Intent:**
Creel delivers into orch strands as into any strand (decision record D1) and its reminder carries no content (D2).
Bound: EXTERNAL senders, programs and other orch strands may now inject into and interrupt an orch strand, as on any strand, and creel reminds an orch strand of its unread mail; every such sender is a process of the operator's user behind the 0700 lyxtemp root, and a hold, and card 3's command inject, on an orch strand stay refused to any sender but a program.

- `checkMayAct` loses its orch refusal: EXTERNAL senders, programs and orch strands may inject into and interrupt an orch strand, and any other strand acts on its own strand only, as before.
- `Store.Hold` refuses a hold on an orch strand from any sender that is not a program, naming that only a program may hold an orch strand and the way forward: post it mail with `lyx creel post`, or cycle the orch with `lyx orch distill` or `lyx orch refresh`; a program's hold on an orch strand and every hold on a non-orch strand keep today's rules.
  Input reaching the refusal: an EXTERNAL sender or the other orch strand `ly:orch-2` holding `ly:orch`, both privileged, so each passes `strandTarget` and the privilege check; a non-orch strand is refused earlier by the privilege check.
- `deliverMailLocked` journals no `withheld` line; `appendWithheldLocked`, `noticesName` and `withheldEntry` go, and so do `InboxResult.Withheld` and `InboxResult.Notice`, so `Inbox` answers counts only and `lyx creel inbox` (`inboxCmd`) prints no `withheld` or `notice` key.
- `pool` loses its `withheld` map; `pool.apply` still accepts a `withheld` record from a stage 1 journal, acts on none and keeps it in the log (`PATTERN-tolerant-records`); `eventWithheld` and `record.Notice` stay, documented as read from old journals only.
- `deliverableReceiver` admits every strand name, orch strands included, and still no program, so `hasPending` and `pendingReceivers` start a worker for an orch strand and mail left unread to one from stage 1 is reminded from the daemon's first probe; the `Scheduler` comment drops its orch sentence.
- `composeReminder` renders `[creel] N unread: #id from <sender>, … — run lyx creel read`, at most five entries and `+K more`, with no subject; `worker.remind` passes the snapshot's hold owner, and under a hold the command reads `lyx creel read --from <owner>`.
  `reminderSubject` and `maxReminderSubjectRunes` go; the journal's `subject` and `lyx creel log` are unchanged.
- `tools/sandbox/SANDBOX-CREEL-SUITE.md` rewrites scenario C9 (Orch): its Goal, Watch and Verdict say mail to the orch-role strand is reminded in its pane, a text inject and an interrupt from EXTERNAL are accepted, a hold from EXTERNAL or from another orch strand is refused with the program-only message, and `inbox` shows plain counts; the Verdict is `FAIL` on a hold accepted from a non-program, or on a keystroke in an orch pane that no creel verb caused.

Tests:
`reminderFor` builds the new line and takes the hold owner, empty for none, and every reminder row of `TestScheduler_Reminders` drops its `«subject»`; its row `ReminderSubjectIsCutAndItsDelimitersReplaced`, whose cutting goes with `reminderSubject`, becomes the row asserting a message whose body holds delimiters and a second line is reminded with no subject; its row `HoldMutesOtherSendersLetsTheOwnerPassAndEndsAtExpiry` expects the reminder under the hold to name `lyx creel read --from <owner>`; its rows gain: an orch strand is reminded and a text inject to it is typed; the row that asserted nothing is typed into `ly:orch` and `ly:orch-2` now asserts their reminders.
`TestScheduler_Injects`' row `InjectsAreTypedBeforeRemindersInQueueOrder` drops the `«hi»` subject from its `reminderFor` entry.
`TestSweep_RemovedHolders`' rows `PostWithLessThanTheBaseLeftDoesNotDelayTheSweep` and `PostInsideTheConfirmIntervalKeepsTheConfirmDeadline` read every clock wait on the premise that orch mail has no delivery worker, which no strand's mail keeps once every strand gets a worker, so they read only sweep waits through `e.sweepWaits`.
Its row `MailReturnsOneStepAtATimeThenIsUndeliveredAndAnswersItsWaiter` drops the `withheld` events from its expected list, and its assertion that `ly:orch-2` got no `Idle` call and was never typed into changes to the reminder it now gets.
`TestStore_Mail`'s rows `Inbox_OrchCallerSeesWithheldReminders`, `Reply_ToOrchIsWithheld` and `Post_ToOrchJournalsOneWithheldLine` change to an orch caller's plain counts and a post or reply to an orch journaling `posted` only, and its row `Read_SetsAndInbox` drops its `inbox.Withheld` check.
`TestStore_InjectAndHold`'s rows `Inject_SelfInjectPassesExceptForOrch` and `Inject_AndInterruptIntoOrchAreRefused` assert the refusals this card removes, so they are rewritten to the new rule: an orch strand's self-inject, and a program's inject and interrupt into an orch strand, succeed.
It gains rows: EXTERNAL and another orch strand inject into and interrupt `ly:orch`; a hold on `ly:orch` from EXTERNAL and from `ly:orch-2` is refused, each row asserting the refusal's full text, the program-only rule and its way forward, and a program's succeeds.
`TestDaemon_Lifecycle` gains a row: a journal written before start holding a post to `ly:orch` and its stage 1 `withheld` record folds, and the started daemon types the reminder for it.
`TestJournal_Fold`'s row folding a `withheld` record drops its `p.withheld` assertion and keeps folding the record with nothing acting on it.
`internal/creelengine/doc.go` replaces "Orch strands in stage 1" and the reminder paragraph, states the hold rule on orch strands, and rewrites "Unbuilt: stages 2 and 3" to the stage 3 remainder with the scoped `PATTERN-orch-pane-single-writer` text; `internal/creelcli/doc.go` states inbox's output.

**ImpactSummary:** Creel-internal privilege, reminder and fold changes plus one CLI field removal, and the creel sandbox suite's scenario C9 rewritten to match; every assertion on the old reminder text, the orch refusals and the withheld record lives in `TestScheduler_Reminders`, `TestScheduler_Injects`, `TestStore_Mail`, `TestStore_InjectAndHold`, `TestSweep_RemovedHolders`, `TestJournal_Fold` and `TestDaemon_Lifecycle`.
