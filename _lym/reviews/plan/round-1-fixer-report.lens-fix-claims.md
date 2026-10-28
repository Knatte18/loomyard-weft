```yaml
lens: fix-claims
outcome: reported
```

Task: the fix-claims lens. I checked each sentence the fix added or rewrote that states a bound, a behaviour or a fact about the code against HEAD.

No findings.

- **Card 5, reconnecting wait:** `Client.resume` turns a dropped post into `VerbWaitReply` (`internal/creelengine/client.go:180-181`). A reply to a sender that is not EXTERNAL goes through `deliverMailLocked` and never journals `delivered` (`internal/creelengine/verbs.go:202-203`). That supports "that reply stays mail to the strand, so no `delivered` record is journaled for it".
- **Card 5, `noticeFits`:** the 8 KiB limit matches `boundaryClaimBudget = 8 << 10` (`verbs.go:253`), and `LineFits` measures the full `MessageLine` against it (`verbs.go:35-36`). The claim of "a few dozen bytes tighter" matches the length of the marked-form text.
- **Card 2, `TestParentOrchStrand`:** it exists in `internal/orchcli/notify_integration_test.go`. It builds the beam `b`, the member `m` and the pair `p` forked from the prime, and runs the member walk with the beam orch live and stopped. That supports the claim that the dropped rows' walks are covered.
- **Card 7:**
  - The chase wait band of 60s to 8m stays clear of `isProbeWait` (`d <= 8s`) and of the narrowed sweep band (`8s < d <= 50s`, with the sweep's 30s→50s backoff and 10s confirm wait).
  - `agedPersistInterval = time.Minute` (`deliver.go:22`).
  - Reading `ParseProgram` for the `CREEL` exemption matches how `isProgramMailbox` reads the sender (`fold.go:496-498`).
  - The probe running reed status and so tmux comes from the review's cited code, which I did not re-read.
- **Card 11:** neither `internal/creelengine` nor `internal/agentname` is on `allowedImports` today, as the fix states.
- **Card 12:** the cited comment above the advisor's history polling is at `smoke_seats_test.go:125-126`, and the header's session-channel line is at line 3.

Tool calls:
1. Read `TestParentOrchStrand`'s location, `LineFits`, and where `eventDelivered` is used. My context did not show `TestParentOrchStrand` or the 8 KiB limit.
2. Read `boundaryClaimBudget`, the file holding `TestParentOrchStrand`, and `Store.Reply`'s delivery branch, to check the "no `delivered` record" claim.
3. Read `TestParentOrchStrand`'s subtests, to check the coverage claim for the dropped card 2 rows.
