```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I mapped each diff change to a decision, and each decision to the changes that build it.

**Per change → decision**
- `internal/shell` `UnsetEnv`: D4 (unset on the default directory).
- `claudeengine/accountconfig.go`, `config.go`, `template.yaml`, `configreg_test`: D1.
- `Worktree` type, `NewForWorktree`, `shuttleengines.AccountKeeper`, `hubgeom.EngineWorktree`, and the `NewForWorktree` wiring in `burlercli`, `gatecli`, `loomcli`, `webstercli`, `orchcli`, `battencli`: D4's told geometry, D2's names reader and D8's `Dirs` rule.
- `lymdirs` constants, `fabricengine.ClaudeAccountsDir`, `accountstore.go`: D2 and D5.
- `accountpick.go`, `accountkeeper.go`, `battencli/accountpick.go`, `adoptcli`, `battencli` create, cancel and drive: D3 and D9.
- `accountenv.go`, the `command.go` and `claudeengine.go` account export, `scrub.go`, `tmuxkit.ServerEnviron`, `shuttleengines` `EnvScrub` test: D4. The scrub edits are the `CLAUDE_CONFIG_DIR` scrub the scope allows.
- `accountverbs.go`, `claudecli`, `settings.go` `addStatusLine`, `OpenLaunch` status line, `cmd/lym` and `sandbox_coverage_test`: D5 and D7's verb, wired as `PATTERN-cli-cobra` requires.
- `transcriptread.go`, `prober.go`, `resume.go` `registryDirs`, `scope.go`, `manifest.go`, `audit.go`, `background.go`, `sessionname.go`, `usage.go`, `tools/tokencount`, `RecordReaders(dirs)` and `SessionNamers(dirs)`, `reedcli/watchdog.go`: D8.
- `CarryTranscript`, `TranscriptCarrier`, `SessionHeldError.TranscriptCopied`, `Spec.TranscriptCopied`, `RunState.TranscriptCopied` and `LaunchOffset`, `refusedResume`, `Runner.RefusedResume`, `loadRefusedResumeError`, `startFreshAfterRefusedResume`: D6.
- `reedengine/hold.go`, `Strand.Held`, `StrandStatus.Held`, and `planResumeLaunches` holding a strand: D7 steps 1 and 4, plus the Discussion-review ruling on the hold ops.
- `accountswitch.go`, `AccountSwitcher`, `switchAccountAtLimit`, `Runner.SwitchAccountOnLimit`: D7 and D5's `blocked_until`.
- `accountrelaunch.go`, `stopForRelaunch`, `updateRunState`, `updateRelaunchRecord`, the `Run.Stop` and `stopRecorded` lock changes, `awaitStartup(eventsProof)`, `relaunchBound`, `RunState.relaunching`, the `runIsLive` predicate and its readers, `sweepOrphans`, `strandRemovable`, `StopStepRuns`, `dispositionCandidate`, `settleStaleRelaunch`, `apiErrorWatch.handledOffset`: D7 and the arbiter's round-3 ruling. The "stop-ended relaunch returns `OutcomeDied`" entry and the "`switchAtLoadLimit`/`relaunchRepeat`" parent entry (F2) are built here.
- `orchengine/restart.go`, `orchcli` `relaunchOrch`, `relaunchStrand` and `soleOrchStrand`: D7's orch path and D6. The `soleOrchStrand` and `relaunchStrand` extraction is forced by the direct-call requirement.
- `shuttlecli/state.go` and `shuttleengine` `Relaunching` and `SessionCauseRelaunching`: D7's reader rule.
- `docs/reference/claude-accounts.md`: D10 wiring and manual check.
- `pattern/PATTERN-shuttle-provider-seam.md`: the declared PATTERN change. It also adds a line to the completion-signal section ("neither drops nor relaunches a strand held for a relaunch"), which follows from D7's reed hold.
- `shuttlefake`, `fakes_test` and other test fakes: forced by the new interfaces.
- `accountFile.Picked` (a timestamp per entry, beyond D2's `worktrees`, `counts`, `weights` and `listed`): forced by D2 and D3's concurrent-pick and told-names-reader rules, so not scope.
- `internal/testcover/mutantbuild_integration_test.go` (commit `c3f46285`): traces to no decision (see finding).

**Per decision → builders**
- D1 to D10, plus every `Added after Discussion` entry: each has at least one change above. No decision is unbuilt on what I read.
- Not checked: D9's "`lym adopt root` writes no entry for the prime". The prime guard is in `picksAccounts`, but I did not read the `adoptcli` diff.
- Not checked: the D3 adopt row, where `PickAccount(keepEntry)` replaces an existing entry for the name. I did not read the `adoptcli` or `battencli` call sites for that `keepEntry` value. The handler or `code-correctness` should confirm it.
- D10's landing-notes criterion: `docs/reference/claude-accounts.md` carries the wiring and the manual-check steps. `_lym/landing/summary.md` does not exist in the worktree, so the criterion is met only through the doc.

**Findings**
1. LOW, class `scope`. `internal/testcover/mutantbuild_integration_test.go:117-122` and commit `c3f46285` fix a flaky parallel-test env slice in `testcover`. It traces to no decision, no traced change forces it, and the range's commits carry no `Pattern-Departure:` trailer. Fix: move it out of this branch, or have the parent record why it rides here.

**Tool calls**
1. `cat` of the small diffs (docs, burlercli, gatecli, loomcli, testcover, tmuxkit, webstercli, reedcli, fabricengine, configreg, lymdirs): my context lacked them.
2. `grep` of the per-file headers of the other small diffs, `head` of the `battencli` diff, and `git show --stat c3f46285`: to confirm the files each package touches and the `testcover` commit's scope.
