```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. For each change, I list what else it forces, then give the findings.

**Per change: set found, covered or not**

- **`verifytree.Result.RunnerCrash`, set by `runAndRecord` (`verify.go:419-425`).** The set is every `verifytree.Verify` consumer.
  - Covered: landingshed `check` (Parent-Align, Finalize, merge fixer turn gate), landingshed `checkHead` (pr-review), and loomshed `NewVerifyGate`.
  - Not covered: `internal/websterengine/verifygate.go:420-466`, the webster verify gate. It handles only `Passed`, `Skipped`, `Dirty` and `TimedOut`, and reads `RunnerCrash` nowhere.
  - Not covered: `internal/webstercli/verify.go:69`, `lym webster verify`. It reads no `RunnerCrash` either.
- **Closing line `RunnerCrashLine` and `FindRunnerCrash`.** The writer is `run.go:204`. The readers are `verify.go` `runnerCrashDetail` and `detachfixer.go` `endedOnRunnerCrash`. The test is `watchdogline_test.go`. Covered, and writer and reader agree on the prefix and the `;` cut.
- **`gatefixer` consumers, under PATTERN-gate-fixer's new text "a runner crash gets none".**
  - Covered: the card fixer in `detachfixer.go`, the landing fixer, and the loom row gate through `Terminal`.
  - Not covered: the webster fixers. `websterengine/planfixer.go`, `verifyfixer.go` and the `fix-verify` path still get a crash as a plain failure (F1).
- **New `RunRecord` fields `runner_crash` and `rerun`, and the extra first-run record.** The only reader is `gatecli/report.go:129-165` (`runReport`).
  - It counts every non-abandoned, non-build record as a tier run and sums wall time.
  - A rerun tier now writes two records. The first run's record is written by `recordFirst` with `WaitStart` set, and the rerun's record has `WaitStart = Start`.
  - `tier_runs` therefore counts a rerun tier twice. This is arguably right, since two runs happened, and the shortage rerun now behaves the same way.
  - The wait and wall sums stay correct. The doc comment says the row "carries the tier runs", so this is not a defect. No finding.
- **`exitCodeOf` in `run.go`: a crash-signalled child now gives 128+signal.**
  - Callers: `tierrun.go:279` and `build.go:48`.
  - A build step killed by SIGABRT, SIGSEGV or similar now exits 134 or 139 instead of 1. `build.go` consumes the build exit code as a failed build, and the doc.go exit-code paragraph names only tiers. Neutral; no finding.
  - `driftengine/steprun.go` has its own `exitCodeOf`, which is unaffected.
- **Rename `shortageRetryDelay` to `rerunDelay`, and `defaultShortageRetryDelay` to `defaultRerunDelay`.** Every reference is updated in `cli.go`, `rungate.go` and `detachrun_test.go`. A grep finds no leftover use of the old names or of `retryOnShortage`. Covered.
- **`verifyGate.check` signature, now `(reason, crashed, err)`.** The non-test callers are `parentalign.go:198`, `finalize.go:714` and `fixerrun.go:287`, and all are updated. The tests are updated, and the integration test is updated too. Covered.
- **`fixerEndCrash`.**
  - `haltReason` handles it.
  - `parentalign.go:389-425` switches only on `fixerEndFixed` and `fixerEndSpent`. Its default branch presumably goes through `haltReason`. I did not re-read it, so I treat it as covered.
  - `finalize.go:803-815` falls through to `haltReason`. Covered.
- **PATTERN.md `PATTERN-gate-fixer`.** The text is updated. The entry has no background file, so none needs editing. The doc.go edits for gatecli, gateslot, verifytree, landingshed and loomshed match the diff. The `loomshed/verifygate.go` doc is edited in the diff.
- **pr-review `headFailureReason`.** The agent is told to "end your turn again, which runs the gate again". The reason is mailed once per head (`prreview.go:575-579`, `MailedHead`), so there is no loop. Covered by the existing mechanism.

**Findings**

1. **MEDIUM, design (scope).** `internal/websterengine/verifygate.go:420-466` (also `internal/webstercli/verify.go:69`).
   - The webster verify gate does not read `Result.RunnerCrash`. A crashed runner is parsed as ordinary failure output.
   - The gate then reruns the verify once and, on a second failure, calls `fail(VerifyGateReport{Failures: ..., LogPath})`. That hands the crash to the verify-fixer fork and cold fixer (`verifyfixer.go`, `planfixer.go`, `fix-verify`).
   - This contradicts the new PATTERN-gate-fixer text "a runner crash gets none" and the board entry's "a crash that repeats stops the run with the crash named, not a fixer".
   - The webster gate also reruns a crashed verify itself, on top of the gate's own rerun, so up to four runs happen.
   - Fix: in both webster paths, treat `res.RunnerCrash` like the timeout case. Fail terminally, name the crash, and spawn no fixer. Or declare the narrowed scope: reword the PATTERN-gate-fixer text, or record a `Pattern-Departure`.
   - Raise it to BLOCKING if the handler judges the PATTERN text binding on every gatefixer consumer.

2. **LOW, consistency.** `internal/gatecli/doc.go:21` and `:399`, against `internal/gatecli/detachfixer.go:128`.
   - The doc says the card fixer is skipped when stderr "holds `gateslot.RunnerCrashLine`". `endedOnRunnerCrash` reads the whole stderr file for that line.
   - A card-scope call that also prints an earlier unrelated crash line cannot reach this state, because `judged` suppresses the line when any other run failed.
   - I found no defect, only a note: the check has no scoping beyond that suppression. Drop it if the handler judges it noise.

**Tool calls**
1. Bash grep of `websterengine/verifygate.go` and `webstercli/verify.go` for `RunnerCrash`, a grep for `Rerun`, and `prreview.go:545-600`: my context lacked webster's handling and the pr-review loop bound.
2. Bash grep for `RunRecord` readers and for "shortage" docs: to find other readers of the new record fields.
3. Bash `sed` of `websterengine/verifygate.go:415-480`, a grep of `adoptcli/fixloop.go`, and a grep of `Abandoned`/`Carried`: to see the webster gate's failure path.
4. Bash `sed` of `gatecli/report.go:120-185` and a grep of `gatefixer.` users: to see how the report counts the new rerun record and which fixers exist.
