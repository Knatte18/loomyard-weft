```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: check that the F1 and F2 test changes agree with the code and tests around them.

**Findings**

- **LOW** — `internal/loomcli/watchfilechanges_test.go`, `TestWatchFileChanges/AMissingDirectoryReturnsANilChannelAndAStopThatIsSafeToCall` runs in parallel.
  - `watchFileChanges` calls the process-global `logger.Warn` on a missing directory.
  - `looparm_test.go:25` and `landingdeps_test.go:236` say their serial tests stay serial because `logcapture` swaps that global logger.
  - Serial tests finish before parallel ones resume, so the new Warn cannot reach their capture. `looplock_test.go:149` also calls `CaptureVerbose`, and I could not confirm from the grep output that its test is non-parallel.
  - If it is parallel, a capture there could see the stray Warn, and the capturing test would need to tolerate or filter it.

Otherwise consistent:
- The `RecordChanges` row in `wiring_seams_test.go:50` ("assembled in run.go") is untouched and still accurate.
- The doc comments in `deps.go:363` and `doc.go:12` still match the tested behaviour.
- The floor-rest rows match `pause` in `prreview.go`. The `restsFloorRemainder` flag and the `d <= 0` guard keep the generic sleep-bounds check valid.
- The existing standing-clock row keeps its own comment and now goes through the shared `changeDuringFirstSleep(0)` helper.

**Tool calls**
1. Bash (`grep` for `logcapture` files, `git show --stat`, `grep` for `RecordChanges` docs): to find sibling logger-capturing tests and doc cross-references my context lacked.
2. Bash (`grep` for `logcapture` and `t.Parallel` lines): to see whether those siblings run in parallel.
