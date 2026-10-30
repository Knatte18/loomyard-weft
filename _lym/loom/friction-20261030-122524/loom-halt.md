loom halted blocked at PR-Review

```
the GitHub call to open the pull request failed: pullrequest: open a pull request from label-coverage into main: POST https://api.github.com/repos/Knatte18/loomyard/pulls: 422 Validation Failed [{Resource:PullRequest Field: Code:custom Message:No commits between main and label-coverage}]; way forward: make sure the gh user's token (GH_TOKEN, GITHUB_TOKEN, or gh auth login --hostname github.com) is valid and can reach the repository, then run lym loom resume
```

producer: PR-Review
state: blocked
anomaly: escalation-to-human
history_entries: 8
trace_file: /home/AD.NORCERESEARCH.NO/hanf/Code/Knatte18/loomyard-LYMHUB/label-coverage/.lym/logs/trace-20261030T102953Z-7fca76dbcdc0d424-3083558.log

history:
- Preflight / done / 2026-10-30T10:27:55Z
- Drift-Baseline / skipped / 2026-10-30T10:27:55Z
- Drift-Await / skipped / 2026-10-30T10:27:56Z
- Darn-Write / done / 2026-10-30T10:29:52Z
- Drift-Verify / skipped / 2026-10-30T10:29:52Z
- Code-Bouncer / done / 2026-10-30T10:29:53Z
- Describe / skipped / 2026-10-30T10:29:53Z
- PR-Review / stuck / 2026-10-30T10:30:03Z
