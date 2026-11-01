```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens over the dobby idle-notice diff. Case-by-case results first, then findings.

**Cases run**

- `ReadNewestRunEnd`, root absent: returns zero time and nil. Correct.
- `ReadNewestRunEnd`, root unreadable (non-ENOENT): returns the error. `noticeCondition` handles it, so correct.
- `ReadNewestRunEnd`, only a running record, or no record, or an undecodable record: skipped. Correct, and the undecodable case is logged once per episode.
- `ReadNewestRunEnd`, records for done, failed (`died`) and stopped (cancelled): all count, because the filter is anything but `running`. Correct.
- `ReadNewestRunEnd`, a legacy record whose outcome is empty: it counts as ended, since `"" != "running"`. That is wrong only if such a record is still live, and `legacyAskingOutcome` is handled elsewhere. I judge it harmless, because it can only make the clock fresher.
- `ReadNewestRunEnd`, the stat fails after the sweep removes the directory: skipped. Correct.
- `ReadNewestRunEnd`, the end time is the `run.json` modification time. This holds for the normal path, where finalize is the last write, so it is correct there. For a stopped run it also holds, because `stopRecorded` rewrites the record when it records the stop (`stop.go:77`).
- `ReadNewestRunEnd`, an old ended run rewritten later: the modification time moves forward, which can only make the clock fresher. A later rewrite of an ended run is only possible through `stopRecorded`, and that skips terminal records (`stop.go:73`). `attach.go:336` rewrites only a record that reads running. No wrong result.
- `readNewestAgentEnd`: `LoadConfig` and `ReadNewestRunEnd` both take the task worktree's `AnchorPath()`, the same as `readAgentActivity` (`wire.go:628`). Their arguments are consistent. An error from `LoadConfig` propagates. Correct.
- `NewestAgentEnd` closure, locator error: returns a zero time and the error, which the test pins with the slug in the message. Correct.
- `noticeCondition`, `NewestAgentEnd` nil or `runs` non-empty: `newestEnd` stays zero and the old path runs unchanged. Correct.
- `noticeCondition`, seam error: it calls `Fail` and returns an empty finding and stamp. The empty return matches the `Agents` error path, so no notice goes. An error from the new seam therefore suppresses the quiet notice for as long as it persists, which is the same as for `Agents`. Correct.
- `noticeCondition`, `Recover` is only reached on success, and its key is distinct from the others. Correct.
- `judgeAgents`, `newestEnd` zero: `zero.After(since)` is false, so the result is unchanged. Correct.
- `judgeAgents`, `newestEnd` after `since` and `NewestHistory`: it wins. A probe seconds after the end reads idle near zero and sends nothing.
- `judgeAgents`, the newest end is old: idle is measured from it and the text is right ("no agent activity readable for 45m10s", as in the test).
- `judgeAgents`, `newestEnd` earlier than the history entry: the history entry wins. Correct.
- `judgeAgents`, a `since` newer than all three: it wins. Correct.
- `judgeAgents`, runs still `running`: the `len(runs) == 0` branch is skipped and the path below is untouched. Correct.
- `judgeAgents`, the stamp now carries `newestEnd`. A new end changes the episode key, so an old quiet episode resets. That is correct, because the activity is fresh.

**Findings**

None.

**Tool calls**

1. Read `stop.go` lines 55–90 and `attach.go` lines 320–340, because the manifest lacked the other writers of `run.json`. I needed them to check that the end time is the last write and that no later rewrite moves it wrongly.
