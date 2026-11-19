```yaml
lens: fix-affected-set
outcome: reported
```

Task: as the fix-affected-set lens, check that the fix covers every member of the sets its findings rest on.

No findings.

- **F1 (`BootTime` / `StartedAt` / `started_at` sites):** I grepped the whole repo for `BootTime`, `proc.BootTime`, `Loop.BootTime` and `btime`, excluding `_lym`, `.lym` and `.git`. It returned nothing, so no stale reference survives in Go, markdown or docs. Every `LoopPIDRecord{` literal outside `shedverbs` (`pairteardown`, `loomcli/landingdeps`) sets only `LoopID`, `Loop` or `Child`, so none relied on `StartedAt`. Every other `started_at` hit is an unrelated seed, status or cancel record.
- **F2 (consumers of `loopLogLinesOf`, `loopExitEvidence`, `unmarkedExitCause`):** The three are called only inside `shedverbs`: `deadLoopStop`, `loopExitedEarlyStop` and the two integration-test call sites. All now pass a boot id or `""`.
- **F5 and F6 (doc claims):** The "start time" and "early end" claims appear only in `internal/shedverbs/doc.go`, at lines 155 and 225. Both are updated. The other "start time" hits are about the process start-time token in `pairteardown` and `dobbyengine`, which is correct as written.

Not checked: F3 and F4 are not set-based. Neither fix adds `LockGrace` to `internal/shedverbs/doc.go`, which does not enumerate `LoopSpec` fields.

Tool calls (3):
1. Repo-wide grep for `BootTime` and `btime`: my context held only the files I had read, not a full-repo search.
2. Grep for `LoopPIDRecord{`, `started_at` and the changed `shedverbs` functions across `internal`, `cmd` and `docs`: to find constructors and callers outside the files I read.
3. Grep for the doc phrases about early ends, start time and boot id: to find other statements that the doc changes could contradict.
