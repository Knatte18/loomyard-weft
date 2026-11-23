```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness lens on the fixer's change set, covering `describe.go`, `loop.go`, `beamcli/describe.go`, `dobbydeps.go`, `beamdescribe.go` and the added tests.

I walked each changed function with one concrete input and a failure of each call it makes. I found no wrong results, no broken callers and no test that passes with its defect in place.

Findings:

1. **LOW** — `concludeDescribe` in `internal/dobbyengine/describe.go`: a failed state write leaves the draft recorded twice.
   - If the `UpdateState` that stores `OwedLine` fails after `Record` has already run, it returns `(true, err)` with no `OwedLine` stored.
   - The next tick finds `Describe` still set and a draft that passes, and calls `Record` again. That is a second commit and a second pull-request update.
   - The `endDescribe` failure on the success path (`line == ""`) re-records in the same way.
   - It needs the state file to be unwritable right after a successful write, so it is an edge case.

2. **LOW** — `ReplaceBeamDescription` in `internal/landingshed/beamdescribe.go`: `summary.md` changes file mode.
   - `writeBytesAtomic` goes through `os.CreateTemp`, which creates the file at mode 0600. The old `os.WriteFile(..., 0o644)` made it 0644.
   - The restore path does the same. `DraftBeamDescription` still writes 0644, so the same file now gets different modes depending on who wrote it.
   - Git only tracks the executable bit, so this is a cosmetic inconsistency.
   - The error text also nests prefixes: `landingshed: replace the beam description: write the description: landingshed: write description: …`.

Traced and found correct:
- **Owed-line retry.** With the post failing, the first `describeStep` returns the wrapped error and `OwedLine` stays set with one `Record` call. After the post recovers, `settleDescribe` takes the `OwedLine` branch, posts once and ends. The F8 subtest asserts exactly this and fails if the persist-before-post or the skip-re-record is removed.
- **Unreadable draft.** `readErr` while `now.Before(Deadline)` returns the error and keeps the agent. Past the deadline it stops the strand and posts "draft unreadable: …", and the next landing spawns a second agent. The F3 subtest pins both phases.
- **Spawn failure.** `endFailedSpawn` drops the landings and posts, and returns nil. If `endDescribe` also fails (state path replaced by a directory), it returns the joined error, and the F13 subtest sees `strand-1` stopped. Without `stopDescribeStrand` that subtest fails.
- **Tick wiring.** `TestRun_DescribeFailureLeavesTheTidyRunning` would fail under the old `return err`: tick 3's tidy report notice would be missing and "tick failing: check the describer agent's draft…" would be logged. The `NoPassWhileAMoveStands` extension fails if `describeStep` runs under the hold, or leaves `scheduling`, or is removed.
- **Busy flag.** The F11 assertion fails under `d.busy = waiting`.
- **Record seam.** `errors.As(err, &afterRecordError{})` with a value target matches the value-typed wrapper, and is false for a nil error. `Unwrap` and `Error` preserve the verb's messages, so the hand path's callers and tests are unaffected. A `RecordedButPullRequestNotUpdated` row fails if `recorded` is ignored.
- **Callers.** The only callers of the changed `Record` signature are the harness and `beamDescribeDeps`. `readKeptAgents` and `rememberKeptAgents` are used only in `loop.go`. The prime's early return in `readKeptAgents` still returns the remembered values.
- **Worktree-gone stops.** Stop order is tidy, then describer. A test whose state record is undecodable gets nil for both.

Tool calls made: none. My context already held every changed file and test.
