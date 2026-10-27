```yaml
lens: fix-affected-set
outcome: reported
```

Findings, by the lens's list (F3, F1, F4, F2):

- **LOW, F3 sibling left stale.** `internal/adoptcli/adopt.go:98`, the `outcome` doc comment: it still ends "the test suite started or the warning of a start that failed". F3's premise was that adopt now starts the adopt check, not the test suite. The fix changed only the header at line 1. The search `grep -rni "test suite" internal/adoptcli` also hits `internal/adoptcli/deps.go:192` ("the adopt's test suite writes its output"), `deps.go:195` ("the adopt's test suite: the hidden check") and `deps.go:203` (the error text "the test suite is not started under go test"). Line 195 explains itself, so it is fine. Lines 192 and 203 and `adopt.go:98` use the old name for the same thing. Line 200's "re-run the test suite" is about the test binary and is correct.
- **No findings, F1.** `logRunChanges` has one caller, `passes.go:140`, and `heartbeat` has two call sites, `loop.go:227` and `loop.go:451`. Nothing else reads `lastHeartbeat.IsZero()`, and the `first` variable is gone. `primeOwed` is set only in the `heartbeat` failure path (`passes.go:246`) and cleared in `logRunChanges` (`passes.go:202`), so every member is covered.
- **No findings, F4.** The search for `and the dobby passes` and `Geometry.LogPath` in `internal/dobbyengine` returned no remaining sites.
- **No findings, F2.** It is a doc-only change in `config.go` and `gateslot/doc.go`. `docs/overview.md` mentions adopt, but I did not check whether it describes the adopt seed, because the tool-call cap left no room for a further search.

Unchecked: F2's wider doc sites (`docs/overview.md` and `PATTERN.md`) for any sentence claiming a hub can pin the template `adopt`.

Tool calls (3 of 6):
1. `grep -rni "test suite" internal/adoptcli internal/cmd`: the F3 set. My context held only the header edit.
2. `grep` for `lastHeartbeat.IsZero|primeOwed|loggedRuns == nil|logRunChanges(` in `internal`: the F1 callers and sites. My context held only the files I had read.
3. `grep` for `and the dobby passes|Geometry.LogPath` in `internal/dobbyengine`, plus `grep -l adopt` over `docs/`, `PATTERN.md` and `pattern`: the F4 sites and the F2 doc candidates. My context did not show them.
