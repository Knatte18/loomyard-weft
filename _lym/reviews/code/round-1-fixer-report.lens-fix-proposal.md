```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge, from the fix-proposal lens, the proposals the fixer carried out as written (F1–F8).

**Findings**

1. **LOW**: F7, the doc-only fix, `internal/gatecli/doc.go` Exit codes paragraph.
   - The proposal's doc option records the new build exit code (128 plus the signal) but misses its downstream effect.
   - `classifyHang` (`internal/websterengine/verifyhang.go:54`) treats any exit code `>= firstSignalExit` as a hang, so the webster verify gate never reruns it.
   - A `lym gate test` call whose build step is killed by SIGSEGV or SIGABRT used to exit 1, so the webster gate reran it as ordinary flakiness.
   - It now exits 139 or 134. It has no runner crash line, because a build is never a judged tier run, so `RunnerCrash` is false.
   - Webster then fails it as a hang, "killed or exited with exit code 139", with no rerun.
   - The other option the reviewer offered, keeping the old code for the build, would avoid this. The doc fix leaves it undocumented.

2. **LOW**: F1, the verb test `TestVerifyVerb_RunnerCrashIsNoFailureToFix` in `internal/webstercli/verify_integration_test.go`.
   - The test runs a real `lym gate test` with the production `defaultRerunDelay` of 10s and takes about 10.4s (the tier-1 log shows 10.44s).
   - This breaks the `testing` skill's rule that a production delay is injectable and the test shortens it instead of waiting it out.
   - The verb test has no way to shorten the delay. A faster row would be a `runVerify` test over a fake `verifytree` result, or putting the crash coverage in a shorter test.

3. **LOW**: F6, the `retryTier` change.
   - A shortage in the first run followed by a crash in the rerun now leaves `crash` empty.
   - `runLabel` and `end.crash` then count the run as `judged` (`internal/gatecli/run.go`: `else { judged = true }`).
   - The call closes with an ordinary failure at exit 134. The landing and loom sites, the webster gate and the card fixer then treat it as a judged failure and hand it to a fixer.
   - That fixer is spent on a failure that judged no code. The one-sided crash reading is what the reviewer asked for, but it moves this case from "no fixer" to "fixer" without a note.

I found no breakage in F2–F5 or F8 as proposed.
- F8's halt happens after the head-moved and dirty discard checks, and it does not set `MailedHead`. A resume therefore regates the head. The row's session is stopped on every Stuck verdict.
- F1's stuck reason takes the crash prefix branch before `gate.Reason`, so a stale verify-gate report on disk does not leak into it.
- F1's verb envelope omits the `failures` key on purpose. The only other readers of the `findings` envelope I found are in `webstercli/verify.go` and `validate.go`, so nothing breaks.

**Calls made**
1. grep for `findings`, `failures` and `exit_code` readers in webster and burler non-test sources: to check that the crash envelope's missing `failures` key breaks no reader.
2. grep of `classifyHang` for exit-code handling: to see how a build killed by a signal is now classified, since the exit code changed.
