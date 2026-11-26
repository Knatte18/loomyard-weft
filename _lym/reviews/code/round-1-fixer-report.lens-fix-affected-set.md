```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set check of the fix change set (does each set-premised finding's fix cover every member).

**Findings left unchecked** (no search rerun): F6, F7, F8, F9. F7 and F8 name specific branches, and I only saw their new rows in context.

**Findings**

1. **MEDIUM: `internal/runpusher/seams_test.go` (F4).** The `NewDeps` conditions still have no test.
   - F4 names "tryLock, LockPath, NewDeps conditions, worktreeGone, pauseCovers, handedOver and RestartPusher". The new file covers all of these except `NewDeps`.
   - No `runpusher` test calls `NewDeps`. `run_test.go:73` fakes `Conditions`, and the only production caller is `loomcli/pusher.go:85`.
   - So dropping the refused-push or hand-over check from `Conditions` (`run.go:61` onward), or the pair-only `PauseCovers` wiring, fails no new test.

2. **MEDIUM: `battencli/wire.go` and `loomcli/wiring.go` `c.startPusher` closures (F3).** The finding's premise still holds for both resume closures.
   - The new tests call `startRunPushers` (`teardownpush_test.go:148`) and `startRunPusher` (`startpusher_test.go:44`) directly.
   - No test runs the `c.startPusher` that `wire` assigns. `resume_test.go:290` and `continue_pass_test.go:76,193` overwrite it with fakes.
   - Deleting the call inside either closure therefore still passes. F3 asked to "assert through the same seam that the closure passes the prime target and … the pair target".

3. **MEDIUM: `battencli/wire.go` `teardownPushSeams` (F2).** Part of the premise stays uncovered.
   - `TestTeardownPushError` tests the mapping function alone, and the only seam test runs with pushes off.
   - Removing the `teardownPushError(...)` wrap, the real `PushRun` call, or the `TipsOnOrigin` call (returning `true`) on a present pair fails no new test. I saw no existing one that would catch it either.
   - F2 names "nor the real PushRun and TipsOnOrigin calls". Its extraction alternative covers only the mapping.

4. **MEDIUM: pushes-off branch in `teardownPushSeams` against `pattern/PATTERN-pair-remote.md:94-95` (F1).**
   - The pattern says the teardown "removes it only while origin's tracking refs hold both tips". `battenshed/doc.go` says the same.
   - With git or pushes off, `Level` now returns true without reading any ref, and neither text states that exception.

5. **LOW: `internal/battenshed/deps.go` `Level` comment and `internal/battenshed/doc.go` Worktree-Teardown paragraph (F1).** Both are stale after the new bound.
   - They say "A false read means a tip moved since the push, so the teardown pushes again" and "A tip that moved since the push releases the second hold and pushes again".
   - Neither mentions `maxTeardownPushRounds` or the Stuck outcome.

6. **LOW: `beamcli/dobbydeps.go` `Runs` closure in `hubDobbyDepsStarting` (F3).** The new beam test asserts only the prime `Tick`.
   - F3 asked for "once for the prime and once for each owned pair" in a `beamcli` tick assertion.
   - If `Runs` passed `runpusher.StartIfAhead` instead of `startPusher`, no test would fail. The per-pair case is asserted only on `DobbyRuns` in `battencli`.

F5's search (pre-push check wording in code, `pattern/` and docs) turned up no further stale site.

**Tool calls**
- grep of `runpusher.Start`, `StartIfAhead`, `RestartPusher` and `NewDeps` sites, plus `NewDeps` in runpusher tests: my context lacked the full caller and test set.
- grep of `PushUntilLanded(` and `TipsOnOrigin(` callers: needed to find any other loop with the F1 shape; there is none.
- grep of pre-push and status-commit check wording across code and docs: F5's set was not in context.
- grep of teardown push-again wording across code, docs and patterns: the sibling texts of F1 were not in context.
- sed of `battenshed/doc.go`, `deps.go` and `PATTERN-pair-remote.md` spans, plus grep of `startPusher` in tests: I had not read those texts or the test overrides.
