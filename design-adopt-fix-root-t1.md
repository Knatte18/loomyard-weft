# Fix the adopt check of root on osl-1046

`adopt-fix-root-t1` · Task · bug, gate · running · Worktree-Create

The adopt check of root failed on this machine: tier2.

Failed tests:
- github.com/Knatte18/loomyard/internal/reedengine TestRunButton_RealTmux/AnswerWithEveryMetacharacterReachesTheFileIntact: buttonclick_integration_test.go:206: the ask entry wrote "it's \"q\" \\ \\$HOME ; ~ #{session_name} a  b", want "it's \"q\" \\ $HOME ; ~ #{session_name} a  b"

Excerpts of the check's suite.log:

```
=== RUN   TestRunButton_RealTmux
=== RUN   TestRunButton_RealTmux/AnswerWithEveryMetacharacterReachesTheFileIntact
    buttonclick_integration_test.go:206: the ask entry wrote "it's \"q\" \\ \\$HOME ; ~ #{session_name} a  b", want "it's \"q\" \\ $HOME ; ~ #{session_name} a  b"
--- FAIL: TestRunButton_RealTmux/AnswerWithEveryMetacharacterReachesTheFileIntact (0.21s)
=== RUN   TestRunButton_RealTmux/EscapeRunsNothing
--- PASS: TestRun_TeardownScenario/EndsTheSessionAndRemovesThePair (1.14s)
=== RUN   TestRun_TeardownScenario/WaitsForRetiringDriverThenArchivesTheCommittedReport
--- PASS: TestSmokeDebugLog/RepeatedCrashBootsBoundServerClientAndOutLogs (2.46s)
--- PASS: TestSmokeDebugLog (3.96s)
=== RUN   TestSmokeLifecycle
=== PAUSE TestSmokeLifecycle
=== RUN   TestSmokeUpRefusesAWorktreeNameTmuxWouldRewrite
```

Diagnose the failure first. A code failure you fix and land as usual. A machine failure, such as a missing tool, an old version or timing like a security agent holding new binaries, has nothing to commit.

Stop condition: the failure is a machine failure. Write a stop report naming the exact steps the operator must take on this machine and, as its way forward, a re-run of lym adopt root once the machine is settled, never lym loom resume.
