```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens, searching each finding's set for members the fix missed.

**Unchecked list:** F1 and F5 (a single test gap each, no set).

**F3 — LOW** — `internal/tenderengine/doc.go:97`
- **Summary:** The package doc for `SyncBoltHold` still says "A slug with a live entry of its own scope gets none." The fix changed that: a `bolt-hold` entry that records another machine is now replaced.
- **Evidence:** the `SyncBoltHold` doc comment in `internal/tenderengine/bolthold.go` got the replace sentence, but `doc.go:94-99` did not.

**F4 — no finding.** The search covered the `OwnerCaller` and "owned by another machine" wording across Go and markdown outside `_lym`.
- `internal/pauserecord/doc.go:17` and `record.go:22` are reworded as legacy.
- `record.go:101` and `record.go:243` only say what `OwnerHeld` reads.
- `internal/adoptcli/doc.go:268` says an `OwnerCaller` entry is converted in place, which agrees with legacy handling.
- The other hits describe run ownership, not the caller.

**F2 — no finding.** The only `Deps.Notice` caller is in `internal/dobbyengine/upgrade.go`. The other `Notice` hits are unrelated notice helpers.

**F3 call sites — no finding.** `SyncBoltHold` has one production caller, `internal/beamcli/dobbydeps.go:123-138`. It takes `req.Machine` from `owners.Holds[0].Machine`, which fits the new machine comparison. The other readers of `BoltHoldCaller` (`resume.go:145` and `159`, `pauserecord/hold.go:117`) only read the entry.

Calls made:
1. Grep of the `OwnerCaller` and "owned by another machine" wording, the `BoltHoldCaller` call sites and the `Notice(` callers, in one Bash call. My context lacked a repo-wide search for these.
2. `grep` of the `SyncBoltHold` mentions in doc files, plus a read of `internal/beamcli/dobbydeps.go:120-140`. My context lacked the package doc text and the caller.
