```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1/F2 change set for agreement with its surroundings (fix-consistency lens).

No findings.

- The new `RowAt` re-application at `run.go:297` sits inside the `if gated` block, after the `found` check. Its comment sits there too and matches the Step 2 comment at `run.go:236`.
- `run.go:289` is the only post-Step-2 re-read of the status in `stepLocked`, so the line-240 resolution has no other twin to repeat.
- `entry.go:72` re-reads the status inside `enter`'s skip loop. A skip persists a named `OnDone` row, so that read never carries an empty `current_producer`.
- Only `progress.go:88` also calls `RowAt`, and its use is unchanged.
- `goto.go`, `failedstop.go` and `rename.go` read the status for their own verbs. The review's Rejected section already covers `goto` and resume.
- The F1 test and the F2 `TestRowAt` table were both added to `run_routing_test.go`, which has the `tier0` tag, and the shedengine gate run was green.

Tool calls (1 of 6):
1. `git diff` of `run.go` plus a grep of `RowAt` and `ReadJSON[Status]` across the package. My context lacked the final committed diff and the list of other status re-read sites.
