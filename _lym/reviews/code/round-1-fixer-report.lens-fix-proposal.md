```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals the fixer carried out as proposed (F2, F3, F4, F6, F7, F8), using only what my context holds.

**Findings**

1. **MEDIUM** — `checkNewGitEntries` / `describeGitEntry` (F2 proposal): a sighting that carries `ownInDir` but whose binary did not write the entry is still blamed.
   - `ownInDir` is true when this binary or any descendant has `cwd == dir` at snapshot time.
   - Go's own `t.TempDir` and `os.TempDir` users do not chdir into `/tmp`. But a test that runs `exec.Command(...)` with `Dir: os.TempDir()` will fail the package, wrongly, whenever another package's daemon writes `.git` in that window.
   - The sighting text says "so this package wrote it", which is stronger than the evidence (presence, not authorship).
   - The old rule had the same weakness, but the new text states it as fact.
   - Suggested fix: soften the line to "so it may have written it", or require the process in `dir` to be an actual `git` argv.

2. **MEDIUM** — `snapshotProcesses` (F2 proposal): the self-detection check is vacuous for the failure it targets.
   - `listedSelf` is set from `Pids()`, before `ProcArgv` is checked.
   - If `/proc` lists pids but `ProcArgv`, `ProcCwd` and `ProcPPID` all fail (for example under `hidepid`), then `procs` holds nothing but `listedSelf` is true.
   - The result is `attributed: true` with empty `others` and `inDir`.
   - `ownInDir` is false, so nothing fails. The package keeps its code and prints "test binaries running beside this one: none", which wrongly says nothing else was running.
   - Suggested fix: set `listedSelf` only when this binary's own `ProcArgv` is read.

3. **LOW** — `tmuxkit.go` package doc, the `PATTERN-test-isolation.md` `.git` bullet and `_lym/landing/summary.md`: the old landing-summary and pattern statements that the finding "names the binaries running then" remain, now beside a rule where the others no longer decide the code. The pieces are not contradictory, but the `others` list now carries no blame logic; a reader may still assume it does.

4. **LOW** — `TestRunCheckingGitEntries`, rows `RunMovingTheEntryAwayTakesNoSnapshotForTheMove` and `DirectoryWithNoWatchLeavesTheOtherWatched` (F7 proposals): the "no extra snapshot" assertion reads `len(snapshotted)` right after the run returns.
   - The check is vacuous for the create-only filter. The rename event, if unfiltered, would trigger a snapshot before the create (ordering), so the run's single receive would take the rename's snapshot, and the create's snapshot would remain in the channel.
   - That case does fail the `remaining != 0` assertion, so the row works.
   - For `missingDir` the row proves only that the other directory still works. It does not prove the missing directory was skipped rather than hung on, but the 10s timeout covers that. No defect.

**Fixes with no counter-case found:** F3 (keep directive), F4 (wording), F6 (all sightings kept), F8 (rune cut). F3 matches the repository's existing `//testtiming:keep` style.

**Further tool calls:** none.
