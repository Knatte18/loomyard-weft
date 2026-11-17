# Round 2 fixer report

## Changes

- **F1** (BLOCKING), fixed with my own change.
  The proposed `internal/fswatch` route is impossible: `internal/logger` imports `internal/proc`, and `internal/fswatch` imports `internal/logger`, so `proc` importing `fswatch` is an import cycle.
  The proposed one-second poll can lose a kill in the last second before the scope is removed.
  Instead `watchForOOMKill` in `internal/proc/memorycap_linux.go` blocks in `poll(2)` with `POLLPRI` on the scope's `memory.events`, which the kernel wakes on each change, plus a pipe that `runMemoryCapped` closes once the command's wait returns.
  There is no timer, so the loop wakes only on the scope's own events, and the 20 ms `oomKillPollPeriod` is gone.
  The final read after wait stays.
- **F2** (MEDIUM), fixed as proposed.
  `scopeCgroup` reads without a deadline while the command runs.
  `runMemoryCapped` sets a `markerReadBound` read deadline only after wait returns.
  The doc comments of `markerReadBound` and `scopeCgroup` match.
- **F3** (MEDIUM), fixed as proposed.
  `run` in `internal/gatecli/memoryguard.go` sets each kill's `enclosing` pids from the sample's parent links, `killTree` notes them with the killed binary, and the `TestStartMemoryGuard` row "an inner binary's kill notes the outer binary that started it too" asserts both pids.
- **F4** (LOW), fixed as proposed.
  `killedAtOwnLimit` reports a kill only when both `oom` and `oom_kill` are above 0, with `TestKilledAtOwnLimit` rows.
  The doc comments of `scopeHadOOMKill`, `runMemoryCapped` and the `proc` package doc say the count is of kills at the scope's own limit.
- **F5** (MEDIUM), fixed as proposed in `_lym/landing/summary.md`, not committed.
- **F6** (LOW), fixed as proposed in `internal/gatecli/doc.go` and `docs/overview.md`.
- **F7** (LOW), fixed as proposed in `internal/gatecli/doc.go` and `internal/gateslot/template.yaml`.

Commits: three fix commits, one for F1, F2 and F4 together since they share one function, one for F3, one for F6 and F7, and a fourth for the self-check findings.
`lym gate test` tier 0 and tier 1 over `internal/proc`, `internal/gatecli` and `internal/gateslot` passed after the fixes, with the real-scope tests `TestRunMemoryCapped_AKillAtTheCapEndsWhatTheChildLeftInItsScope` and `TestRunMemoryCapped_AKillOfAChildLeavesTheCommandRunningAndEndsItsScope` among the passes.
The final `pread` change was checked with tier 1 over `internal/proc` only.

The comment gate then flagged five comment lines of this round, in `internal/gatecli/memoryguard.go` and `internal/proc/memorycap_linux.go`: three wrapped breaks and two over-long lines.
They are re-broken at sentence boundaries in a fifth commit, with no change to code or wording of substance.

The verify run then failed `TestTierTags_OneTierPerTestFile`: `internal/proc/memorycap_linux_test.go` held the tier 0 `TestKilledAtOwnLimit` beside tier 1 tests.
A sixth commit moves `TestKilledAtOwnLimit` unchanged into `internal/proc/memoryevents_linux_test.go`.

## Self-check

- fix-code-correctness, MEDIUM: `ReadAt` may read twice and swallow a wake-up. Fixed as proposed: one `unix.Pread`.
- fix-consistency, LOW: `GuardKilled` field doc in `internal/testcover/runner.go` is stale. Fixed as proposed.
- fix-consistency, LOW: `_lym/landing/summary.md` memory_mb decision lacks the fallback qualification. Fixed as proposed.
- fix-consistency, LOW: `internal/proc/doc.go` reads as any kill at the cap. Fixed as proposed.
- fix-proposal, LOW: a noted outer pid nobody reads stays in the guard's map, so a reused pid could read as killed. Deferred, see below.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

- fix-proposal's pid-reuse finding: the guard cannot know which pids a caller will read, so forgetting an unread pid needs a lifetime rule the review did not ask for.
  The inner pid carried the same exposure before this round, and it needs pid reuse inside one guard's lifetime.

## Rulings

None.
