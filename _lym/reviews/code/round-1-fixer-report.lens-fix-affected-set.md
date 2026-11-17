```yaml
lens: fix-affected-set
outcome: reported
```

No findings against the fix-affected-set lens. Checks against the review's three set-premised findings (F2, F3, F1):

- **F2 (every `RunAlone` caller):** All three guarded callers pass `GuardKilled`: `redundantmutants.go:469`, `redundancy.go:545` and `testmappackage.go:675`. The only unguarded `RunAlone` caller is `cmd/testtiming/redundancy.go:331`, which starts no guard, so it correctly leaves the field nil.
- **F2 (`MemoryCapped`/`GuardKilled` consumers):** All consumers handle `GuardKilled` explicitly, in `redundancy.go`, `redundantmutants.go` and `testmappackage.go`. Nothing else reads `RunResult.MemoryCapped`.
- **F3 (test-binary start sites):** The three `lym gate test` paths that start test binaries are covered: tiers (per-tier guard), the redundancy report step (`run.go:216`) and `lym gate redundant`/`lym gate testmap` (`guardOwnTests`). No other verb outside `cmd/testtiming` calls `RunAlone` or `RunMemoryCapped`.
- **F1 (`RunMemoryCappedVia` consumers):** The only consumer is `child.go:65`. `RunMemoryCapped` consumers use `killed`, which is unchanged.

Out-of-scope notes, not findings:

- [LOW] `redundantmutants.go:349` (`planMutants` → `ProfileBlocks`) is a test-binary run under the guard with no `GuardKilled`. A guard kill there surfaces as a "block table run" error, never as a false verdict, so nothing wrong ships. F2 named only `RunAlone`.
- [LOW] `pattern/PATTERN-daemon-wakeups.md:45-46` lists where the memory guard runs ("a tier's command, `lym gate redundant` or `lym gate testmap`"). The fix adds the redundancy report step as a fourth, so the list is slightly stale. The sibling text in `doc.go`, `template.yaml` and `overview.md` was updated.

Tool calls, all within the 6-call cap:
1. Grep of non-test code for `Toolchain{`, `RunAlone`, `ProfileBlocks`, the guard starters and `RunMemoryCapped*` callers: F2, F3 and F1 each need their site set, and my context only held the files the fixer edited.
2. Grep of `MemoryCapped` and `GuardKilled` across non-test code: to find consumers of the changed result flags.
3. `sed` of `internal/proc/doc.go:52-75` plus a grep for files naming the memory guard: to check for stale sibling prose.
4. Grep in `gateslot` and the PATTERN file for guard mentions: to see which of them enumerate the guard's sites.
5. `sed` of `PATTERN-daemon-wakeups.md:40-56` and `gateslot/config.go:30-37`: to confirm the one stale enumeration.
