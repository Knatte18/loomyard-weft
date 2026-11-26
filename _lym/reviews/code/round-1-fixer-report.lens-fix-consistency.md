```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the F1–F3 change set.

**LOW: commit `ee02e99f5` (F2), failure condition of `waitForFixtureProcessesToExit` in `internal/loomcli/smoke_helpers_test.go`**
The fix loosens an existing check but records no `Loosened-Check:` trailer.
- Before, any process still under the root after the second patience failed the test with `t.Errorf`.
- Now a process that was not yet signalled is killed and gets a further patience, and the test passes if it then dies.
- The fixer's instructions require `Loosened-Check: <test> — <what was loosened> — <measured cause>` when a commit widens a limit in a pre-existing test, and the commit has a subject line only.
- This depends on reading a failing cleanup helper as part of "a test". The rule allows adding the line in a later commit, without rewriting history.

I found no other inconsistency. The helper's doc comment, the `registerBootstrapTeardown` doc and inline comments, the `_lym/landing/summary.md` bullets and the removed explicit call in `TestSmokeWatchdog_StartSpawnsTheDaemon` agree with the code and with the call-site grep in context.

Tool calls: none.
