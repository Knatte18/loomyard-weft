```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens — for each change the diff makes, the set of callers, readers, tests and docs it forces, and whether each is covered.

**Per-change sets**

1. **No stop at the first failing run (loop in `run.go`).**
   - Docs and text that said "stops at the first failing tier" or "skips the guard step after a failure": the `test` Long text, the `--guards` flag usage, the `doc.go` Verb, Guard-step and Exit-codes sections, the `runTest` doc comment and the `docs/overview.md` gate entry. All updated.
   - Tests that pinned the old stop: the two-tier test, the failing-tier guard row and the card-timeout row. All updated.
   - A repo-wide grep for the old phrasings (stops at the first / first failing / first tier that fails / after the listed tiers pass / guardRunOf) found no stale member outside `_lyx`, `.lyx` and benchmarks. Covered.
   - Readers of the gate log: `gateslot.FailedTier` (`internal/gateslot/runlog.go:188`) returns the last failing tier summary line. `internal/landingshed/verifygate.go:57`, `internal/landingshed/doc.go:120`, `internal/verifytree/doc.go:34` and `pattern/PATTERN-verified-tree.md:17` all say "the last failing tier the log names". **Not covered:** see finding 1.
2. **Closing line `gate test: failed runs: ...`.**
   - It does not match `tierSummaryPattern`, which keys on `tier%d`, so `FailedTier` is unaffected. Covered.
   - It goes to `request.stderr`. The test harness hands the verb's stdout as its error stream (`test_integration_test.go:323`), so the stdout assertions read it. Covered.
3. **`runEnd` replaces `int` returns (`runListed`, `runGuard`, `runGreenAware`, `reportCarried`, `runWithSlots`, `report`).** All callers are inside `run.go` and were updated. `guardRunOf` was removed, and a grep found no caller or comment left behind. Covered.
4. **Exit-code contract.** `cli.go`, `doc.go` and the `runTest` doc comment agree on "first failing run's own". Covered.
5. **`overview.md` and `doc.go` contract text.** Updated in the same diff, as CLAUDE.md requires. There is no stencil, gate.yaml or `PATTERN.md` text about stopping at the first tier, so the stencil-wording rule has nothing to check. Covered.

**Findings**

1. **MEDIUM, class `design`:** `FailedTier` and its consumers now disagree with the exit code when two or more runs fail.
   - Location: `internal/gateslot/runlog.go:188-203`. Consumers: `internal/landingshed/verifygate.go:57`, `internal/landingshed/doc.go:120-121`, and the "failing tier" wording in `internal/verifytree/doc.go:34` and `pattern/PATTERN-verified-tree.md:17`.
   - Before this change at most one tier could fail in a call, so the last failing summary line was the failing tier. Now a call with failures in tier0 and tier6 exits with tier0's code, and the landing failure record names tier6, the last failing line.
   - The diff left this reader unchanged and did not decide whether last or first is right. The same applies when a guard run follows a failed listed tier: the guard run's summary line carries the same `tierN` and wins.
   - The fix is to decide which tier the record names, and to align `FailedTier`, the landing record and the docs that say "last failing tier" with the exit-code rule. Alternatively the diff could state why last is right.

No other member of the affected set is left wrong.

**Tool calls**
1. Grep for stale "stops at the first" phrasings across `.md`, `.go`, `.yaml` and `.tmpl`. My context lacked a repo-wide search outside `internal/gatecli`.
2. Grep for "failing tier" readers and `SlotBusyExit`. This found the `FailedTier` and `landingshed` consumers, which are outside the files read.
3. Read `FailedTier` and `TierSummaryLine` in `internal/gateslot/runlog.go`, to see what the closing line and later-tier summaries do to the parsing.
