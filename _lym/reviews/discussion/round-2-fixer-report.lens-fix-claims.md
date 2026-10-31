```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD.

No findings.

Claims checked:
- **D5's `DenyPublish` claim holds.** `internal/landingshed/prsession.go:146-147` sets `Unawaited: true` and `DenyPublish: true` together, so an unawaited start can carry the publish row. The row's steer `steerPublishDeny` (`internal/shellguard/rows.go:100`) ends "commit your change and tell the operator", as the fix says. Only `claudeengine` renders the row, and it is the only engine under `internal/shuttleengine`.
- **D5's pointer to D8 holds.** D8's bullet "Run `lym loom sampler-done` only when the operator says the work is done, after committing" says what the new D5 text attributes to it.
- **D4's reopened-session sentence holds.** `internal/reedcli/resume.go` recreates the same persisted strand, which keeps the recorded name. So the "admitted only if it keeps the recorded strand name" branch is the one `lym reed resume` takes, and the refusal branch covers anything else.
- **D4's "committed in the call's one diff" holds.** It matches `internal/loomcli/decision.go:11`, "in the call's one committed diff".
- **D6's stop on entry is consistent.** "as the wake does before its deciding read" matches D6's own wake bullet. The entry outcomes and the new acceptance case agree with the stop-first order.

Tool calls:
1. grep for `DenyPublish` and `Unawaited` in `internal` and `prsession.go`: my context did not show whether an unawaited start can carry the publish row.
2. Listed the `shuttleengine` engines and found `reedcli`: my context did not show which engines render the row or where `lym reed resume` is defined.
3. Read the `internal/reedcli/resume.go` header: my context did not show whether a reed resume keeps the strand's name.
