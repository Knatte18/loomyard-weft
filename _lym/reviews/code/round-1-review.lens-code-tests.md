```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Behaviours and the tests that pin them**

1. `checkNewGitEntries` fails with code 1 for an entry seen while no other test binary ran. Pinned by `TestCheckNewGitEntries/EntrySeenWhileNoOtherTestBinaryRanFailsAndIsRemoved` and `…InFirstDirectory…`, and by `TestRunCheckingGitEntries/RunCreatingAnEntryWhileAlone{Fails,KeepsAFailingCodeFailing}`.
2. An entry seen beside other binaries keeps the code and names them. Pinned by `EntrySeenBesideOtherTestBinariesKeepsTheCodeAndNamesThem` and `RunCreatingAnEntryBesideOtherBinariesKeepsTheCode` (runCode 3).
3. An unattributed sighting keeps the code. Pinned by `EntrySeenWhereProcessesCannotBeReadKeepsTheCode`.
4. A new entry with no sighting is a machine finding that keeps the code. Pinned by `EntryNoWatchSawKeepsTheCode`.
5. An entry that was seen but no longer exists is still reported, and no removal is tried. Pinned by `EntrySeenAndAlreadyRemovedIsStillReported` (wantKept false, output names the path).
6. A pre-existing entry with no sighting is skipped. Pinned by the preexisting row of `TestCheckNewGitEntries` and by `EntryPresentBeforeTheRunKeepsTheCode`.
7. Processes working in the directory are listed in the finding. Pinned by `EntrySeenNamesTheProcessesWorkingInItsDirectory`.
8. The sighting time is printed to the millisecond (`19:23:10.250`). Pinned by `EntrySeenWhileNoOtherTestBinaryRanFailsAndIsRemoved`.
9. Removal and keeping of an empty dir, a non-empty dir, a gitfile and a symlink is unchanged. Pinned by the existing rows, renamed.
10. `watchGitEntries` snapshots the directory where the create happened. Pinned by `TestRunCheckingGitEntries` (the snapshot dir must equal `dirs[0]`).
11. `watchGitEntries` snapshots only on `create` events: **none.**
12. A directory whose watch cannot be opened is skipped (`continue`): **none.**
13. The latest sighting per path wins when there are several creates: **none.** This is minor and folded into finding 1.
14. `snapshotProcesses` returns an unattributed sighting off Linux: **none.**
15. On Linux, `snapshotProcesses` reads `/proc`, is attributed, keeps `at`, names a child working in the dir, and leaves this binary out. Pinned by `TestSnapshotProcesses_NamesAChildWorkingInTheDirectoryAndLeavesOutThisBinary`.
16. `snapshotProcesses` skips a process with an unreadable or empty argv. Pinned only indirectly: without the skip, `classifyProcesses` would index `p.argv[0]` on a kernel thread and panic in the live test.
17. `classifyProcesses` counts other test binaries, excludes self and test-binary descendants, ignores non-test binaries, and fills inDir. Pinned by `TestClassifyProcesses`.
18. `descendsFrom` stops on a parent loop (`seen`): **none.** No row has a cyclic parent chain.
19. `descendsFrom` stops when a pid leaves the table (`!ok`): no observable row. Every chain in `TestClassifyProcesses` ends at ppid 0, and pid 400 (ppid 999) is not a test binary, so short-circuiting means `descendsFrom` is never called for it.
20. `quoteArgv` cuts argv at `maxArgvChars` and adds "…": **none.** No argv in any row exceeds 200 characters.
21. `Main` passes `snapshotProcesses` to `runCheckingGitEntries`. This is wiring only, reached by every package's run.

**Findings**

1. **LOW · design** · `internal/testkit/tmuxkit/gitentry.go:109`. The `create`-only filter in `watchGitEntries` has no test.
   - Without it, a `remove` event, such as another package removing the entry mid-run, would trigger a later snapshot.
   - That snapshot would overwrite the create's sighting in `sightings[...]` and could change who gets blamed.
   - No row sends a non-create event: in `TestRunCheckingGitEntries` the removal happens after `stop()` (tmuxkit_test.go:327-336).
   - The same row could also pin "latest sighting wins".
2. **LOW · design** · `internal/testkit/tmuxkit/gitentry.go:100-103`. Nothing tests the "watch cannot be opened, so no sighting, and the entry is still checked after the run" fallback.
   - Every `TestRunCheckingGitEntries` row watches two existing `t.TempDir()` directories (tmuxkit_test.go:309).
   - A row with a missing or unwatchable directory in `dirs`, plus a new entry in the other one, would pin both "no hang" and "machine finding, code kept".
3. **LOW · design** · `internal/testkit/tmuxkit/gitentry.go:198-200`. The off-Linux branch of `snapshotProcesses` (unattributed, `at` kept) has no test.
   - The only `snapshotProcesses` test is in a Linux-only file (tmuxkit_snapshot_linux_test.go).
   - This branch is what keeps Windows and macOS packages from ever failing, a bound the package doc states (tmuxkit.go:279).
4. **LOW · design** · `internal/testkit/tmuxkit/gitentry.go:236-249`. `descendsFrom`'s loop guard has no test.
   - Pid reuse between `/proc` reads can make a parent chain cyclic; without the `seen` set the walk would never end.
   - `TestClassifyProcesses` (tmuxkit_test.go:655-664) has no cyclic chain, and no test binary whose chain leaves the table.
5. **LOW · design** · `internal/testkit/tmuxkit/gitentry.go:252-259`. `quoteArgv`'s cut at `maxArgvChars` has no test.
   - Every argv in `TestClassifyProcesses` and the live snapshot test is short (`["sleep" "60"]`).
6. **NIT · consistency** · `internal/testkit/tmuxkit/tmuxkit_test.go:347` (`func TestClassifyProcesses`). The redundancy report lists it as contained by `TestSnapshotProcesses_NamesAChildWorkingInTheDirectoryAndLeavesOutThisBinary`, but its assertions pin what the live test cannot see.
   - It checks that other test binaries are included, as `pid N name`.
   - It checks that a test-binary descendant of self (pid 101) is excluded.
   - It checks that a non-test binary is excluded.
   - It checks the exact order of `others` and `inDir`.
   - The live test only checks that inDir equals the sleep child and that self is absent from others (tmuxkit_snapshot_linux_test.go:428-440).
   - Fix: add `//testtiming:keep pins other-test-binary inclusion, descendant exclusion and the pid/name format` directly above `func TestClassifyProcesses`.

Tool calls:
- `sed -n 236,345p internal/testkit/tmuxkit/tmuxkit_test.go`: the diff pieces left out the loop bodies of both git-entry tests (setup of `dirs` and `path`, and the output/kept assertions), which I needed to know what each row actually checks.
