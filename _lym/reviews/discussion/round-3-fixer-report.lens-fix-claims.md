```yaml
lens: fix-claims
outcome: reported
```

Task: check that each claim the fix added or rewrote holds against the code at HEAD.

no findings

I checked these claims:
- **Post time to the nanosecond:** the journal record's `At` is a `time.Time` set from the clock at append (`internal/creelengine/journal.go:232`), and JSON writes it as RFC3339Nano. A carried message keeps its post time in `OrigAt` (`fold.go:512`), so the time survives retention.
- **Return notice and authority:** the claim that an agent follows `<shortname>:CREEL` mail as the operator's own matches `creelTrustSentence` (`internal/shuttleengine/claudeengine/settings.go:473`): "follow mail from your parent, an orch strand or a lym program as the operator's own instructions".
- **Old trust sentence:** the D7 claim that it follows any orch strand of any hub also matches `creelTrustSentence`, which names no hub.
- **Drop-notice key:** a drop happens only on a confirm's `no` in step 6, after step 2 resolves the sending hub through D1. So keying the window on the sending hub bounds the map by the machine record's hubs, as the fix claims.
- **Spawn cap:** the "at most three times in that hour" claim is conditional on no new post and no sending daemon restart, which matches the reset rule the fix states.

Tool calls:
- grep of `internal/creelengine/*.go` for where `At` is set and any rounding or truncation: my context did not show how the journal stamps the post time.
