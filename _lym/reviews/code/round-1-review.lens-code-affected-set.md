```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens. For each changed signature, key, output or behaviour, I list the set I found and whether the diff covers it.

**Per-change list**

| Change | Set found | Covered? |
|---|---|---|
| `RunState.Outcome` stays `running` while relaunching; one predicate `runIsLive`/`relaunching` | Every non-test `runOutcomeRunning` site, per my grep: `activity.go` 112/162/180/202, `sessionread.go` 68, `agentwait.go`, `carryover.go` 125, `attach.go` 282/540/546/554, `stepstop.go` 117/153/160, `stop.go` 58/169 | Covered. The outcome stays `running`, so the equality sites read a relaunching run as running. Every site that tests liveness (`runIsLive`, `unfinished`, `noteWaiterlessRun`, `StopOrphanedStepRuns`, `sweepOrphans`) takes `now` and the bound. `soleFinishedCandidate` (`attach.go:282`) compares only the outcome and is gated on the output files all existing, so it is correct. |
| `sweepOrphans`, `dispositionCandidate`, `runIsLive`, `unfinished`, `noteWaiterlessRun` take a bound | Callers: `run.go` sweep, `attach.go`, `stepstop.go`, `carryover.go`, `activity.go`, `agentwait.go`; test call sites in `enginedispatch_test.go`, `attach_asking_test.go`, `rundir_test.go` | Covered. |
| `ReedOps` gains `EndStrandProcess`, `ReleaseStrandHold`, `RelaunchStrand` | Other `ReedOps` users: `websterengine` (reset, verifyfixer, recoverbatch, planfixer), `gatefixer` (spawn, registrar). All take the real `*reedengine.Engine`, which now has the methods. | Covered, and a missing method would fail to compile. |
| `reedengine.StrandStatus.Held` and `ResumeResult.Held` now also lists caller-held strands | `internal/reedcli/resume.go`: help text and envelope field `held` | **Not covered:** F1. |
| `shuttleengines.New`/`NewForWorktree`, `SessionNamers(dirs)`, `RecordReaders(dirs)`, constructor signature | A grep for the old zero-argument `SessionNamers()` and `RecordReaders()` found no remaining caller in `internal`, `cmd` or `tools`. | Covered. |
| `claudeProjectDirFor`, `projectsRoot`, `sessionRegistryDir` removed | A grep found no remaining reference in `internal`, `cmd` or `tools`. | Covered. |
| `ScrubbedEnv` gains `CLAUDE_CONFIG_DIR` | `TestEnvScrub_MatchesTmuxkitServerEnviron` is updated, and the `internal/testkit/tmuxkit` diff is in the range. | Covered. |
| New `accounts` and `account_pins` keys classified `CategoryMachine` | `PATTERN-config-layers.md:45` says the open set is the template keys classified `CategoryMachine`. `configreg` and its test are in the diff. | Covered. |
| `PATTERN-shuttle-provider-seam` declared change | `pattern/PATTERN-shuttle-provider-seam.md` lines 94–96 land the declared stop-verb relaunch-mode text. Line 124 also adds a reed-resume sentence, which the decision record's wording does not state. | Declared entry landed. The extra sentence is consistent with the reed doc and is not a finding. |
| `checkResume` returns a copied flag; `SessionHeldError.TranscriptCopied`; `Spec.TranscriptCopied` | Callers: `run.go` start, `relaunch.go` (`relaunchForReprompt`), `orchcli/start.go` (`resumeOrch`), `accountrelaunch.go`. | Covered. |
| `awaitStartup(eventsProof bool)` | Callers: `run.go` launch, `relaunch.go`, `accountrelaunch.go`. | Covered. |
| `orchcli` start branch refactored into `relaunchStrand` and `relaunchOrch` | The `startOrch` relaunch case now calls `relaunchStrand`. `orchcli/doc.go` is updated. | Covered. |

**Findings**

**F1. LOW, consistency.** `internal/reedcli/resume.go:25-27, 50` is untouched by the diff. The `lym reed resume` help says `held` names only the strands "left down" while a pause covers the pair. `ResumeResult.Held` now also names strands a caller ended on purpose (`Strand.Held`, `lifecycle.go:46-50`, `planResumeLaunches`). The help text is now wrong for that case, and `reedengine/doc.go` documents it correctly. Fix: update the help text to mention both causes.

I found nothing else the diff leaves wrong, and no commit message in the range carries a `Pattern-Departure:` or `Loosened-Check:` trailer to check. I did not audit whether every reed op other than `Resume` and reconcile (layout, add, remove) leaves a held strand's record alone. My context holds only the `reedengine` diff, and the doc comments claim reconcile does.

**Tool calls (3)**
1. `grep` for `runOutcomeRunning` and the removed or renamed symbols, plus `ReedOps` users. My context lacked the unchanged call sites.
2. `grep` for `ResumeResult`/`Held` users, `CategoryMachine` in `pattern/`, and the stop-verb lines in the pattern file. My context lacked the unchanged consumers of `ResumeResult`/`Held` and the landed pattern text.
3. A read of `internal/reedcli/resume.go` plus `git diff --stat`. I needed the help text, which the diff does not touch.
