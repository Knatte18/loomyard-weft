```yaml
lens: fix-affected-set
outcome: reported
```

no findings

Rerun per finding:
- **F1:** a grep for every reader of a session registry directory found only `readRegistry` in `resume.go`, reached through `checkResumable`, `sessionHolder`, `processLiveness` and `registryHoldsLiveEntry`. The central `ErrNotExist` change covers all four. No other test or code pins the old "unreadable" reading of a missing directory.
- **F5:** the other `~/` expansions (`claudeimports.go:172`, `tmuxconf.go:37`) are not account config directories and are out of the finding's set.
- **F6:** the `held` description in `reedengine/lifecycle.go:49-51` already names both causes. `reedcli/resume.go` was the only stale site.
- **F3:** `ReleaseStrandHold` has one production caller, `stop.go:128`. The change is confined to that call.

Tool calls made:
1. `grep` for registry and `sessions` readers across `internal`, to rerun F1's search.
2. `grep` for `~/` expansion sites and for held-strand help or doc text, to rerun F5's and F6's searches.
3. `grep` for `ReleaseStrandHold(` callers, to rerun F3's search.
4. `grep` and `sed` on `reedengine/lifecycle.go` and `battencli/wire_seams_test.go`, to check that `Held` text and the `sessions` registry test do not depend on the old missing-directory behaviour.
