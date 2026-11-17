```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix set through the fix-consistency lens for stale twins and disagreeing siblings.

**Findings**

1. **LOW** — `internal/testcover/runner.go:48-51`, the `GuardKilled` field doc.
   - It says the func "reports whether a memory guard ... killed the test binary with a pid".
   - After the F3 fix, `startMemoryGuard`'s `guardKilled` also reads true for an outer test binary that held a killed inner one (`memoryguard.go:291`).
   - The twin doc still describes only the direct kill.
   - Evidence: the old text, plus `kill.enclosing` being fed to `onKill` in `killTree`.

2. **LOW** — `_lym/landing/summary.md:37`, the design decision "One cap holds the tier's builds and every test binary together".
   - F7 qualified this claim in `doc.go` and `template.yaml` for the data-segment fallback.
   - The summary decision was left unqualified, and it is the same claim.

3. **LOW** — `internal/proc/doc.go:55-59`, the `RunMemoryCapped` paragraph ("so the kernel kills the command alone. The call reports that kill"), and `RunMemoryCappedVia` ("reports whether the kernel killed any process of the tree at the cap").
   - Both are unqualified, while F4 made `treeKilled` a kill at the scope's own limit only.
   - The `runMemoryCapped` doc comment was updated.
   - The package doc still reads as any kill at the cap, so the twin was not updated.

I found no stale twin for F1/F2. `PATTERN-daemon-wakeups.md` needs no entry, because the `poll(2)` watch is event-driven with no timer. Nothing else references `oomKillPollPeriod`, and `markerReadBound`'s docs agree with its new use.

**Tool calls (2 of 6)**
1. `grep` over `internal/proc/doc.go`, `memorycap.go`, `memorycap_windows.go`, `docs/overview.md`, the wakeups pattern and `summary.md`, plus `guardKilled` users across `internal`. I needed the twins the fix did not touch.
2. `sed` of `internal/proc/doc.go:50-70`, `internal/testcover/runner.go:44-52` and the wakeups pattern, with a `grep` for `memory.events`. I needed the exact wording of those twins.
