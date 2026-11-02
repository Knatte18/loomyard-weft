```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Findings I rechecked:
- **F1 (callers that drop `posted`):** `NotifyPrime` has two callers outside `notify.go`. `internal/hibernatecli/deps.go:115` now warns when `posted` is false. `internal/orchcli/pushnotifier.go:46` returns the result to its own caller, so it is not a dropped-notice site of this review.
- **F3 (callers of `reconcileAtWake`):** `watch.go:147` is the only caller, and it now passes `target`. `liftOwned` is also called from `verbs.go:95` and `verbs.go:196`, the cancel and request paths. Those run outside the daemon, hold no `kept` entry, and the diff did not change them.

Not checked: F2 and F4 add test rows only, so no site set applies. I did not compare the new rows against sibling rows beyond what my context already shows.

Tool calls:
1. One `Bash` call with two greps (`NotifyPrime(` callers; `reconcileAtWake(` and `liftOwned(` callers). My context lacked a repo-wide caller listing.
