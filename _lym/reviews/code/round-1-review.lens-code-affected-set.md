```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff forces outside the lines it touches.

**Per change, the set found and whether it is covered**

1. `isBackgroundLaunch` now counts an Agent or Task launch with `subagent_type: fork` as background.
   - Readers:
     - `transcriptHasUnmatchedLaunch` is reached from `outstandingBackgroundTasks`, the fallback when a Stop payload carries no `background_tasks` list.
     - That fallback feeds `events.go:49` (Stop becomes `EventWaiting`) and `signals.go:154` (`judgeTurnEnd` in `sessionstate.go:223` gives busy/background).
     - Downstream of those: `wait.go:883` and `wait.go:1017`, `hibernateengine/watch.go:411`, `tendercli/deps.go:543`, `tenderengine/resume.go:631`, `websterengine/strand.go:89` and `shuttlecli/state.go:85`.
   - Behavior: a session that never calls await-forks and whose payload has no list now reads a fork launch without `run_in_background` as outstanding until a notification names it. That is consistent with the payload path, where `payloadTask` already counts subagents, so I found no defect.
   - Docs: `claudeengine/doc.go` and the `background.go` header are updated. Tests: `TestTranscriptHasUnmatchedLaunch_BashAndAgentLaunches` gains a fork row. The existing fixtures with fork launches (`parent-spawns.jsonl`, `fork-spawn-boundary.jsonl`) go through the audit path, not this scanner. Covered.
2. New `OutstandingForkReader` capability and `Claude.OutstandingForks`.
   - Set: `capabilities.go` (assertion added), `shuttleengine/doc.go` and `claudeengine/doc.go` (both updated), `Runner.AwaitForks` (the only consumer). `TestOutstandingForks` covers it. Covered.
3. New `Runner.AwaitForks` and `awaitForks`.
   - Set: `TestAwaitForks` covers `awaitForks`. The `AwaitForks` wrapper itself (unknown guid, engine without the capability) has no test. That is the tests lens's matter, so I only note it.
   - PATTERN-daemon-wakeups: the wait wakes at most once a second (`pollFloor`) and has its own cap deadline, so it falls outside the "unbounded wait" definition. No entry or guard-list change is needed.
4. New verb `lym shuttle await-forks` (`awaitforks.go`, `cli.go`, `reed` field, `taskFields` extracted from `state.go`).
   - Set:
     - `cli.go` registration and file header are updated.
     - `PersistentPreRunE` still resolves for this verb, correctly, since it needs reed and the engines. The `awaitVerb` skip applies to `await` only.
     - The `AudienceRole` annotation matches the `await` sibling.
     - `stateFields` now uses `taskFields`; the `state` tests cover that.
     - `docs/overview.md` is updated for the verb list and the description.
   - Gap: `internal/shuttlecli/cli_test.go` has no `await-forks` case. `TestRunCLI_PositionalArgValidation` (line 239) has `AwaitNoArgs` and nothing for `await-forks` rejecting an argument. Nothing tests the "no shuttle strand" error. See finding F1.
5. Three stencils (`burler-cluster-fan`, `burler-fix-self-check`, `twill-directive`) and the doc paragraphs in `burlerengine/doc.go` and `twilldirective/doc.go`.
   - Dropped or weakened rules: none. "Write nothing" and "Edit nothing while any fork is outstanding" and the verbatim lens-report rule all stay. The fallback "when it fails, end your turn" keeps the old behavior.
   - Wording: the fix-self-check and twill-directive bullets are identical. The cluster-fan paragraph differs slightly: it says "Then wait" after the handler's own holistic work, and omits "in the message after the spawn". That placement is right for the handler, so I count it as agreeing.
   - Other stencils that spawn forks: `twill-writer-fork.md` and `webster-template-master.md`. The Goal names reviewers and fixers, and webster's Master already ends its turn deliberately, so I count them out of scope.
6. Commit messages: no `Pattern-Departure` trailer, and none is needed. No PATTERN entry is declared changed or touched. `PATTERN-shuttle-provider-seam` holds: the capability sits behind the engine interface and the Claude specifics stay under `claudeengine`.

**Findings**

- F1, LOW, class `scope`, `internal/shuttlecli/cli_test.go:239`. The shuttlecli verb tests (`TestAwaitVerb`, the positional-argument test) were not extended for `await-forks`. The new verb's argument rejection and its no-strand error path are unpinned. A row in the positional-argument table (`await-forks` plus an argument is refused) and a run with `LYM_STRAND_NAME` unset would pin both. Evidence: `awaitforks.go:130` (`Args: cobra.NoArgs`) and `awaitforks.go:138-142`.
- No other set member is left wrong.

**Tool calls** (3)
1. grep for `subagent_type` fork fixtures and tests outside `background*`, to find other readers of fork launches. My context lacked the fixtures.
2. grep for other docs or lists naming `shuttle await` or the verb set, to find stale verb lists. My context lacked the doc search.
3. grep of stencils for other fork-spawning stencils, to find stencils the diff left unchanged. My context lacked that list.
