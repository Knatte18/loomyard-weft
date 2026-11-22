```yaml
lens: fix-affected-set
outcome: reported
```

List: every finding checked; none left unchecked.

no findings

The F18 set is the sites where the hook is set and the receiver state the hook reads. Card 14 covers every site that builds a guard through `gotoGuardEndingFixers`: `arm.go:478` (`specFor`), `arbiterconsume.go:29` (`loomConsumeAttempt`) and `drivewiring.go:289` (`goDriverDepsFor`). No other card in `_lym/plan/` names `sessionLiveness`, `c.reed` or `c.runner`. The one other loomCLI state the hook reads is the commit seam. That seam is built by `pullRequestRecordCommitter(newPrePushCheck(c.location, …), c.location)`, as `prcontinue.go:83` builds it, and it needs only `c.location`, which `wireLightweight` sets. So nothing else the hook reads is nil on the `goto` path.

Tool calls:
1. Grepped the plan for `sessionLiveness`/`c.reed`/`c.runner`/`stepStopRunner`/`GotoAdmitted`, and the code for `gotoGuardEndingFixers`. This reran F18's search, whose site list my context lacked.
2. Read `pullRequestRecordCommitter` to check whether the commit seam also needs full wiring. My context did not show its body.
3. Grepped for the committer's callers and `prePushCheck` to confirm that its check is built from `c.location` alone. My context lacked its construction.
