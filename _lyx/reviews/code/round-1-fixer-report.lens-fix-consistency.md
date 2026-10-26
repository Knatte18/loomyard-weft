```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens over the F1, F2 and F3 change set.

Findings:

- **LOW** — `internal/pawlengine/doc.go:15` and `internal/pawlengine/landrequest.go` (`AwaitMarkerName` and `Paths.AwaitMarker` comments).
  - F2 gave the await marker a second job: it now carries the line `dobby-down-notified` and is rewritten when the dobby is seen live or a refusal is answered.
  - The twins were left as they were. `Paths.AwaitMarker` reads "the marker file Beam-Await writes on its first entry". `AwaitMarkerName` reads "the marker that says Beam-Await was entered before". `doc.go:15` says a re-entry whose dobby is down "post[s] one notice". A reader of the package doc cannot tell that the notice is once per down episode, or that the marker holds that record.
  - Evidence: the new `downNoticeLine`, `recordDownNotice` and `clearDownNotice` calls in `noticeDownDobby` and `refuse`.

No other finding. The F3 wording twins are fixed: the grep over `orchengine` and `orchcli` shows no remaining "orch start" doc comment about `StartAction`, `BeamUpReport.Action` or `reconcileHub`. The hits are unrelated comments or test strings. `Beam-Up` still removes the marker, which also clears the record.

Calls:
1. `grep "orch start"` over `internal/orchengine` and `internal/orchcli`, plus a grep for the marker and notice wording over `internal/pawlengine/doc.go`, `PATTERN.md` and `docs`. Together they were one Bash call, because my context lacked `doc.go` and the other twin comments.
