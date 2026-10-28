```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer proposals the fixer carried out as proposed (F1-F5).

Findings: no findings. A counter-case was tried for each of F1-F5 and none holds.

Notes, not findings: F5 fix said to assert nothing was announced as rerun, but retryOnShortage announces before the delay, so that premise is false; the fixer asserted the announcement and a single attempt instead. The calls == 2 guard on the log-prefix assertion is needed for the new row. No test pins Command() passing defaultShortageRetryDelay, outside the review findings.

Tool calls: none.
