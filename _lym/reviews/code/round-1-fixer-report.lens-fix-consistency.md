```yaml
lens: fix-consistency
outcome: reported
```

No findings.

I checked each sibling the change set touches:

- **Mark call site:** `markLaunchedRemoteControl` has one caller, `internal/orchcli/start.go:807`, so moving it after `saveFreshLaunch` leaves no twin call site behind.
- **Comments around the moved call:** the comments on `markLaunchedRemoteControl` and `launchRemoteControlCheckpoint` (`internal/orchcli/remotecontrol.go:92-111`) do not mention ordering, so none went stale.
- **Package doc:** `internal/shuttleengine/doc.go:170-172` says a connection is reported only once the provider records one. That still holds after the changed `TurnOnRemoteControl`: `connected` is true only on a recorded connection, and an unconfirmed paste returns it alongside `ErrSubmissionNotLanded`.
- **Creel sentinel:** the `ErrSubmissionNotLanded` mapping in `internal/creelcli/delivery.go` (`mapDeliveryError`, `ErrSubmissionNotLanded` to `ErrNotLanded`) matches the new doc text. The error is still returned after the mark, and `internal/creelengine/deliver.go:348` and `:426` treat `ErrNotLanded` as unconfirmed rather than requeued.
- **Helper twins:** `transcriptPathIn` replaces only the two sites F1 named, as its `fix` key said. The older literals in `transcriptread.go`, `background.go`, `audit.go`, `resume.go`, `usage.go` and `sessionname.go` were deliberately left alone.
- **Test fixture:** the `remoteControlRecord` fields used by the `TestRunner_AwaitRemoteControl` rows are unchanged in meaning (`checkpoint` still answers when `pasted` is nil), so the old rows agree.
- **Landing summary:** `_lym/landing/summary.md` "What changes" says creelcli marks "only when shuttle reports a connection". That is still true, but it omits that an unconfirmed paste which connected is now marked too. The summary is outside the code and not needed to use it, so I did not rate this a finding.

Further tool calls (1 of 6 used):
1. `grep` for `markLaunchedRemoteControl` and `launchRemoteControlCheckpoint` across `internal`, plus `internal/shuttleengine/doc.go` lines 160-180 and the `ErrNotLanded` uses in `internal/creelengine/deliver.go`: my context lacked other callers, the package doc text and creel's unconfirmed handling.
