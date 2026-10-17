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
- `internal/creelengine_test#TestSweep_RemovedHolders`
- `internal/creelengine_test#TestDaemon_Lifecycle`
- `internal/creelengine#TestJournal_Fold`
- `internal/creelengine/doc.go`
- `internal/creelcli/doc.go`

**Delete:**
- `internal/creelengine#Store.appendWithheldLocked`
- `internal/creelengine#withheldEntry`
- `internal/creelengine#noticesName`
- `internal/creelengine#reminderSubject`
- `internal/creelengine#maxReminderSubjectRunes`

**Intent:**
Creel delivers into orch strands as into any strand (decision record D1) and its reminder carries no content (D2).

- `checkMayAct` loses its orch refusal: EXTERNAL senders, programs and orch strands may inject into and interrupt an orch strand, and any other strand acts on its own strand only, as before.
- `Store.Hold` refuses a hold on an orch strand from any sender that is not a program, naming that only a program may hold an orch strand; a program's hold on an orch strand and every hold on a non-orch strand keep today's rules.
  Input reaching the refusal: an EXTERNAL sender or the other orch strand `ly:orch-2` holding `ly:orch`, both privileged, so each passes `strandTarget` and the privilege check; a non-orch strand is refused earlier by the privilege check.
- `deliverMailLocked` journals no `withheld` line; `appendWithheldLocked`, `noticesName` and `withheldEntry` go, and so do `InboxResult.Withheld` and `InboxResult.Notice`, so `Inbox` answers counts only and `lyx creel inbox` (`inboxCmd`) prints no `withheld` or `notice` key.
- `pool` loses its `withheld` map; `pool.apply` still accepts a `withheld` record from a stage 1 journal, acts on none and keeps it in the log (`PATTERN-tolerant-records`); `eventWithheld` and `record.Notice` stay, documented as read from old journals only.
- `deliverableReceiver` admits every strand name, orch strands included, and still no program, so `hasPending` and `pendingReceivers` start a worker for an orch strand and mail left unread to one from stage 1 is reminded from the daemon's first probe; the `Scheduler` comment drops its orch sentence.
- `composeReminder` renders `[creel] N unread: #id from <sender>, … — run lyx creel read`, at most five entries and `+K more`, with no subject; `worker.remind` passes the snapshot's hold owner, and under a hold the command reads `lyx creel read --from <owner>`.
  `reminderSubject` and `maxReminderSubjectRunes` go; the journal's `subject` and `lyx creel log` are unchanged.

Tests:
`reminderFor` builds the new line, and every reminder row of `TestScheduler_Reminders` drops its `«subject»`; its rows gain: an orch strand is reminded and a text inject to it is typed; under a program's hold on a strand the reminder lists only the owner's mail and names `lyx creel read --from <owner>`; the row that asserted nothing is typed into `ly:orch` and `ly:orch-2` now asserts their reminders.
`TestSweep_RemovedHolders`' assertion that `ly:orch-2` was never probed or typed into changes to the reminder it now gets.
`TestStore_Mail`'s rows `Inbox_OrchCallerSeesWithheldReminders`, `Reply_ToOrchIsWithheld` and `Post_ToOrchJournalsOneWithheldLine` change to an orch caller's plain counts and a post or reply to an orch journaling `posted` only.
`TestStore_InjectAndHold` gains rows: EXTERNAL, a program and another orch strand inject into and interrupt `ly:orch`; a hold on `ly:orch` from EXTERNAL and from `ly:orch-2` is refused with the program-only refusal text, and a program's succeeds.
`TestDaemon_Lifecycle` gains a row: a journal written before start holding a post to `ly:orch` and its stage 1 `withheld` record folds, and the started daemon types the reminder for it.
`TestJournal_Fold`'s row folding a `withheld` record keeps passing with the record acted on by nothing.
`internal/creelengine/doc.go` replaces "Orch strands in stage 1" and the reminder paragraph, states the hold rule on orch strands, and rewrites "Unbuilt: stages 2 and 3" to the stage 3 remainder with the scoped `PATTERN-orch-pane-single-writer` text; `internal/creelcli/doc.go` states inbox's output.

**ImpactSummary:** Creel-internal privilege, reminder and fold changes plus one CLI field removal; every assertion on the old reminder text and the withheld record lives in the four creelengine test functions listed.
