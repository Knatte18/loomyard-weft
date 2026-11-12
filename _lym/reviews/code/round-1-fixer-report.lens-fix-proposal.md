```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals the fixer carried out as proposed (F1 to F5).

1. **MEDIUM** — `internal/beamcli/dobbydeps.go`, `startTestMap` (F1 fix). The output file lives under the `lymtemp` run directory, and nothing keeps that directory in place while the pass runs.
   - The run directory is deleted by `lymtemp.Remove` and by the sweep once the owner process is dead. The pass is designed to outlive the dobby, so the directory goes while the pass still holds the file.
   - On Windows, `os.RemoveAll` of a directory holding a file another process has open fails. `Remove` or the sweep would then error or retry for as long as the pass runs.
   - On Unix the unlinked inode stays writable, so no breakage there.
   - The fix's reason for choosing a file was that the pass must survive a dead dobby. That makes a dobby-scoped directory the wrong home on Windows.
   - Evidence: `lymtemp.Remove` "releases and deletes the process's run directory", and `lymtemp` has `removeRunDirectory` and `ownerDead` in its sweep.
   - I did not run this on Windows. It follows from the code and the doc comments.

2. **LOW** — same place. The only code that removes the output file is the goroutine after `cmd.Wait`. If the dobby dies or re-execs, that goroutine is gone and the file is never read or removed.
   - After a re-exec the pid is unchanged, so the sweep sees a live owner and leaves the old run directory until that pid exits.
   - The pass's output is unbounded on disk, where `tailBuffer` kept the last `passErrorTailBytes`. I could not see how much the `gate testmap --upkeep` pass prints.

3. **LOW** — the F1 test, `dobbytestmap_unix_integration_test.go`. The review asked for a test that a started pass still runs after the dobby closes its read end.
   - The added check only asserts that the child's stdout is not a pipe.
   - It proves the precondition, not that the pass survives a gone dobby. That is acceptable, but it would pass if the pass wrote to a socket or `/dev/null`.

4. **LOW** — F5, `configsync_test.go`. The review named "a hub base file or prime layer holding `test_map_every` 0".
   - The new row seeds only the single `baseDir` through `ReconcileAll(baseDir, "", ...)`, with no board dir.
   - It does not cover the hub-wide path (`ReconcileHubWideAt` over the board directory).
   - The discriminating power is real: if the prune ran first, `test_map_upkeep: false` would be absent.

F2, F3 and F4 had no case I could build against the proposals.

Tool calls made: none. My context already held everything these findings needed.
