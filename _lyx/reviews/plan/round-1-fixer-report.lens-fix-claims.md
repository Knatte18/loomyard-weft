```yaml
lens: fix-claims
outcome: reported
```

Checked: the claims this round's fix added or rewrote, against the code at HEAD and the decision record.

**Findings**

1. **MEDIUM.** Card 8, the new Intent sentence: "…those misses are bounded by the whole runs at merge-in and finalize and by the whole fallbacks of Decision 3."
   - Summary: the sentence says Decision 3's whole fallbacks catch the closure's misses. They do not. The fallbacks cover only cases the derivation can detect, and the misses are by definition the cases it cannot.
   - Evidence: `_lyx/discussion/decision-record.md:70` reads "every case the derivation can detect as beyond the name closure's sight runs whole; the misses Decision 2's Bound lists are left to the whole gates of Decision 10."
   - Evidence: the merge-in and finalize whole runs are a recommendation in `docs/dotnet-hub.md`, not something lyx enforces (`decision-record.md:181-182`). The sentence should say the misses are left to the whole gates of Decision 10, the recommended `merge-in` and `finalize` gates at scope `all`. It should not cite Decision 3.

**Claims verified as holding**

- **Card 10:** an inert-only group returns the same scope as the `anyWhole` branch: shares, `guardStep` set, `guardDir` `tier0.Dir`, `guards` nil (`internal/gatecli/scope.go:179-182`).
- **Card 10:** nil guards under `go` trigger the module scan (`internal/gatecli/run.go:80`, `scanGuards` at `scope.go:318`).
- **Card 10:** the `none` scoper has an early whole return (`cfg.Scoper != "go"` in `computeScope`).
- **Card 11:** the build-start refusal mirrors `runInvocations`' condition `cmd.Process == nil && ctx.Err() == nil` and its way-forward wording (`internal/gatecli/tierrun.go:165-166`).
- **Card 8:** `configRefusal` always appends the `lyx config gate` way forward (`tiers.go:215-216`). The `none` refusal text matches `tiers.go:118`.
- **Card 2:** the verdict's opening step "fix the hanging test on the task branch" matches `internal/landingshed/verifygate.go:271`.
- **Card 7:** `QuoteArg` refuses a percent sign under Windows only (`internal/verifyrun/run.go:47-65`). `runInvocations` has `run.number` (`tierrun.go:166`).
- **Card 6:** a failed git read returns `Whole`, matching Decision 3 (`decision-record.md:64`).
- **Card 1:** the `Config.Scoper` comment currently reads "go or none" (`internal/gateslot/config.go:34`).

**Tool calls**

- `sed`/`grep` on `internal/gatecli/tierrun.go` and the guard-scan sites: to check the start-refusal condition and the module-scan trigger, which my context had not shown.
- `grep` on `_lyx/discussion/decision-record.md` for merge-in, finalize and Bound: to check card 8's new bound sentence against the decisions, which my context had shown only in part.
