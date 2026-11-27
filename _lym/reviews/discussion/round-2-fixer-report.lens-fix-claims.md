```yaml
lens: fix-claims
outcome: reported
```

Task: check the fix's new factual claims against the code at HEAD (fix-claims lens).

- **LOW.** Changed text: D5, "a reader that judges origin's batten record at Land or Worktree-Teardown, the run reader's past-the-merge read among them".
  The run reader reads the prime's local batten record, not origin's.
  `internal/pairteardown/runread.go:108` reads it through `ReadBattenRecord(prime, slug)`, and `runread.go:204-208` and `:270-271` set `PastParentMerge` from that local record's row.
  The two match on another machine only after its catch-up pulls the prime's records.
  On the landing machine the local record reaches Land before origin's does.
  The rule the fix states still applies to this reader, but the sentence names the wrong record.

These claims hold at HEAD:
- `boardengine.PullBolt` exists (`internal/battencli/cancelcmd.go:211`).
- `fabricengine.RefusedPushStands` exists, and the pusher's start and conditions refuse while the refused-push record stands (`internal/runpusher/start.go:122-125`, `run.go:65-68`).
- A `lym fabric push` that lands clears the record (`pushclassified.go` `landed` → `clear`).
- The merge record's field is `task_head` (`mergerecord.go:45`).
- The merge-in pass reads the task head before it merges (`parentalign.go:38` in `readMergeBefore`, called before `Resolve` at `finalize.go:1349-1353`).
- Batten's transitions push the prime's records through `PushRun(location, location, …)` (`commitstatus.go:182-185`).
- PR-Review's records push is `pairteardown.PushRun` (`landingdeps.go:473`).

Tool calls:
- One grep of `readMergeBefore` and the run reader's batten source. My context showed neither where `before.taskHead` is read nor where the run reader's batten record comes from.
