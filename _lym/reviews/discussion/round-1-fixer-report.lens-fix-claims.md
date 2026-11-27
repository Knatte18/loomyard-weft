```yaml
lens: fix-claims
outcome: reported
```

Task: check that each code claim the fix added or rewrote in `decision-record.md` holds against HEAD.

- **LOW**, D5, "whose not-parked refusal `childSpawnError` in `internal/battencli/wire.go` reads and retries".
  The claim puts the retry in the wrong place. `childSpawnError` (`internal/battencli/wire.go:519-533`) only turns the child's refusal into `battenshed.ErrChildNotParked`. The retry is in `internal/battenshed/innerrun.go:1049`, which also bounds it to at most once per `noticeProbe` (`innerrun.go:378`).

- **LOW**, D7, "on the halt's history entry in the child status, which the dobby's child read, `ChildReading`, reads".
  This reads as if the existing read already carries the new stamps. It does not. `ChildReading` (`internal/dobbyengine/runnotice.go:69-85`) takes only `HistoryLen`, `NewestHistory` and `LastRepeats` from the history, and no field of an entry. The read has to gain `class:`, `settle:` and `halts_at_row:` from the newest entry, and the sentence should say so.

The other claims hold:
- D3: the class-5 rule counts only a Stuck with no `routed_to` (`contracts/orchrole/role-autonomy.md:42`).
- D4: the dobby drops a lifted run from `liftPending` when the run is not drivable (`internal/dobbyengine/passes.go:209-213`).
- D6: the circling decision file holds `decision` and `cause` (`internal/shedadapters/circling.go:50-57`), and `circling continue` refuses at the hard cap (`internal/loomcli/circling.go:121-128`).
- D11: `AwaitsReply` answers true both for an open question and for an unread reply (`internal/creelengine/verbs.go:1096-1100`).
- Scope, Out: an autonomous session runs in bypass mode and the prompt mode is refused (`internal/shuttleengine/claudeengine/command.go:53-59`).
- Notes for the plan writer: `internal/sheddriver/settle.go` and `internal/sheddriver/tacklerrecords.go` hold what the notes say they hold.

Tool calls:
- grep for `ErrChildNotParked`, `role-autonomy.md` lines 34-46, the arbiter stop's builder and the `tacklerrecords.go` header. My context lacked batten's retry site, the class-5 text, where the arbiter stop is built and that file's contents.
- grep for where `arbiterstop.Stop` is built. Needed to check D6's claim that the level is read where the stop is built. It is built from `arbiterstop.Readers` in `internal/shedarbiter/call.go:50`, a design change and no finding.
