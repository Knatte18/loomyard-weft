```yaml
lens: fix-claims
outcome: reported
```

Task: check that the claims the fix added or rewrote hold against the code at HEAD (fix-claims lens).

Findings:

1. MEDIUM. Changed text: `decision-record.md` D6 Bound, "so the fork can run every orch-guarded verb on any run, `lym loom decision replace` and `lym loom decision store` included."
   Summary: the guards admit a caller only when its strand equals the target run's resolved parent. The fork's reach is every run whose parent resolves to this orch, not any run.
   Evidence: `internal/loomcli/decision.go:685-689` (`parent, err := deps.parent()` … `case deps.strandName == parent.Name: return ""`). The add and replace paths reach this guard at `:879` and `:1254`. `decisionStoreVerb` at `:1137-1145` refuses with `deps.strandName != parent.Name`. The fix copied the over-claim from F11's own summary ("on every run").

2. LOW. Changed text: `decision-record.md` D2 facts, "The run status's `history` holds one entry per producer call, naming the producer and the time its verdict was recorded."
   Summary: history folds consecutive identical calls into one entry and appends goto entries that no producer call made, so it is not one entry per call. The asking rule still works, because a goto to Discussion-Write appends a `Discussion-Write` entry after the ask. The fact as stated is wrong, though, and the rule's "recorded after" should name `At`/`LastAt`.
   Evidence: `internal/shedengine/run.go:721-730` (a fold keeps the first `At` and increments `Repeats`, and `LastAt` takes the newer time). `internal/shedengine/goto.go:140` appends `HistoryEntry{Producer: req.Target, Outcome: OutcomeGoto, …}`.

3. LOW. Changed text: `decision-record.md` D2 facts, "After a goto back to Discussion-Write over an unsettled round, the perch halts on that round again, and a `continue` sends the rewritten record to the next round."
   Summary: the cited code says this only for an undecided escalated round. The fix states it for every unsettled round.
   Evidence: `internal/shedadapters/bouncer.go` (around `:367-369`): "The clear does not fire on an undecided escalated round, so a segment re-entered after `lym loom goto` to an earlier row halts Awaiting again on that round, and a `continue` then sends the rewritten artifact to a fresh review round."

All other claims I checked hold:
- NotifyParent's fallback, its posted-false-with-nil-error return, its missing receiver, and its signing rule (`notify.go:41-50`, `:188-218`).
- The CapAssistTimeout chair-timeout rule.
- Generation 1 for a missing record (`liveSettleRecord`).
- CarryRecords carrying the generation record after an archive.
- entryClear archiving and re-seeding.
- seedCall writing the round-1 focus.
- The `decision.go:11` mistaken-call statement.

Tool calls:
- grep for `Repeats`/`LastAt` and the guard functions: my context lacked history folding and which verbs call the guard.
- grep plus a read of `decision.go:1130-1145`: my context lacked the `decision store` guard.
