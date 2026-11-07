```yaml
lens: board-scope
outcome: reported
```

Board-scope lens: checking the decision record's fit with the board entry, the peers and the parent commits.

**Findings**

1. **LOW, consistency, D10 and Open risks.** D10 assumes landing `gate-tier-binaries-only` is enough for the "about 48 s" pass, but its saving only appears once the hub's gate config is switched.
   - Evidence: that peer's record carries an `Added after Discussion (parent, 2026-11-07)` entry: "Keep {test_targets} opt-in in the code… the landing names that hub step (lym config gate --set for tier0, tier1 and tier2)".
   - The 48 s expectation therefore holds only once `{test_targets}` is set in this hub's tier commands.
   - Fix: D10 and the open risk should name that condition, so the re-measure is read against the right baseline.

2. **LOW, scope, D7 (`newMergePairFixture`).** The entry lists `newMergePairFixture` among the fixtures to move. D7 narrows that to its layers repeated at least three times.
   - The narrowing is reasoned and backed by evidence: the fixture is `CopyHub` plus `OpenFabric`, with no pair or commits.
   - It is a narrowing of an entry goal point, not a contradiction. It is correctly recorded under Auto-mode assumptions, so I'm noting it here and not raising it as a defect.

3. **LOW, scope, D4.** The entry says "after checking every test binary runs under `HermeticGitEnv`". D4 keeps the per-repository layer and writes the same keys from Go, so the check is no longer needed.
   - This meets the entry's intent, which is the item's processes removed with no coverage risk.
   - The support log records the rationale, so there's no contradiction.

No `decision`-class finding.

**Entry goal points**
- Item 2, `worktree list` through `dotgit`, with a parity test on a detached, a locked and a prunable worktree: met (D1).
- Item 3, registered templates (key, build function, built once per binary, lock per key): met (D2).
- Item 3, the listed fixtures moved, one package per task: met (D7, D9). `newMergePairFixture` is narrowed (finding 2); `internal/fabricengine` is split into two tasks, each still one package.
- Item 3, tests that exercise `Add` keep building their pairs: met (D7).
- Item 4, folds that keep what each row alone pins: met (D6).
- Item 5, the slimmer `CopyHub` with the fingerprint kept on `SharedHub`: met (D3).
- Item 6, quiet config written from Go: met (D4; see finding 3).
- Item 8, `CopyHub` in the `NewHub` fixtures and the hook without `dirname` and `basename`: met (D5).
- Item 10, tier 1 tests that run no git move to tier 0 as each is touched: met (D8).
- Item 1 left out (`gate-tier-binaries-only`): met (Scope, Out).
- Wanted, each change with its packages, coverage cost and guard: met across D1 to D8 and their bounds.
- Wanted, task order with dependencies, each task sized for one darn run: met (D9).
- Wanted, the saving to re-measure, at most one whole pass and one `internal/fabricengine` pass: met (D10), with finding 1's caveat.

**Entries, peers and commits checked**
- `gate-tier-binaries-only` (peer, Finalize): no conflict. It changes `internal/gatecli`, `internal/gateslot`, `internal/impactset` and `pattern/PATTERN-test-speed.md`, and the record changes none of these. D8's tier moves only add to what its derived lists would gain. The only issue is the opt-in in finding 1.
- `test-real-time-waits` (from the parent's history): no conflict; the record leaves report items 7 and 9 to it.
- `skill-load-hold-unchecked` (peer): no conflict; it touches only `internal/shuttleengine/startup_test.go`.
- `machine-claude-defaults` (peer): no conflict; it has no changed paths.
- Parent commits e1e25cb (drift capture metadata) and 7c77e08 (batching hook transcript wait): no conflict.

**Decision notes:** the board list held no `Decision for fabric-test-speed:` note, so there was none to read.

**Tool calls**
- `lym loom peers`, first call: it printed only the help text, because the session's working directory had moved into `.scratch`. My context held no peers list.
- `lym loom peers`, rerun from the worktree root: the peers, their changed paths and the parent commits, which my context lacked.
- `lym loom peers show gate-tier-binaries-only --record decision`, filtered by grep: the closest peer's decisions, to check for a conflict with D8 and D10. My context lacked them.
