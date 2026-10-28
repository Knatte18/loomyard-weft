```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. Per behaviour, the test that pins it, or none.

**Reader, `internal/proc/tasklimit_test.go` `TestReadTaskLimitAt`**
- Tightest level wins: row "the level with the fewest free tasks wins over a larger limit closer to the root". The user-slice row also covers it.
- `max` level skipped: rows 1 and 4.
- Level missing `pids.current`, or holding a malformed number, skipped: row 3.
- Not found for a v1-only file, an absent file, or no numeric limit: rows 4, 5 and 6.
- The `0::` line is picked out of a mixed file: row 3 has a `12:pids:` line first.
- The walk reaches the root and stops: every row walks up to `/`, so a loop that never stops would hang the test. No row has a `pids.max` at the mount root, but a real cgroup root has none.
- `ReadTaskLimit`'s own paths (`/proc/self/cgroup`, `/sys/fs/cgroup`): none, by design (no host read, per the Constraints).
- `Headroom()`: pinned through the tightest-level rows.

**Config, `internal/dobbyengine/config_test.go` `TestLoadConfig`**
- Default 500: row "the template defaults".
- Negative refused naming the key: the new `task_headroom: -1` row.
- 0 accepted: no `task_headroom: 0` row. The key list is checked in one shared loop and the `max_runs: 0` row covers its `< 0` test, so this is not a finding.

**Hold, `internal/dobbyengine/starts_test.go` `TestStartReady_TaskLimitHold`**
- No task started while the headroom is below the margin: subtest 1 (two ready tasks, two ticks, no spawns).
- No start record: subtest 1 (`stateRecord().Starts` empty).
- No backoff, and the task starts once the headroom returns: subtest 1 (both tasks spawn on the third tick).
- Hold line carries current, max, cgroup and margin, once per episode: subtest 1 (one line after two held ticks).
- Resume line fires once on the first start let through: subtest 1.
- Resume line with `no task limit applies` after `task_headroom` goes to 0: subtest 2.
- `task_headroom: 0`, a not-found reading and a nil seam start as before: the table rows `TaskHeadroomZeroHoldsNothing`, `AnUnreadLimitHoldsNothing` and `ANilSeamHoldsNothing`.
- Wiring: `Deps.TaskLimit` is filled in the `passHarness`. The `hubDobbyDeps` fill with `proc.ReadTaskLimit` (`dobbydeps.go:146`) has no test. `dobbydeps_integration_test.go` pins only the seams that differ between a beam's dobby and the prime's.

**Findings**
1. LOW, `scope`, `internal/dobbyengine/starts_test.go:308-387` (evidence `starts.go:99-103`): nothing pins that the reading happens right before each start, not once per tick. `fake.limit` is a constant, so a version that read the limit once per tick would pass every row. D4 says "right before dobby spawns a task's first pass". Fix: add a row where the seam returns a free reading for the first call and a near one for the second, and assert that only the first of two ready tasks spawns.
2. LOW, `consistency`, `internal/dobbyengine/starts_test.go:308-387` (evidence `starts.go:96-101`): "once per episode" is pinned for one episode only. No row holds, resumes and then holds again, so a regression where `taskLimitHeld` is not reset on resume would pass. Fix: extend subtest 1 with a second hold after the resume and assert the second hold line.
3. LOW, `scope`, `internal/beamcli/dobbydeps_integration_test.go:25-40` (evidence `dobbydeps.go:146`): the D6 acceptance criterion "`hubDobbyDeps` fills the seam" has no test. Dropping `TaskLimit: proc.ReadTaskLimit` would break no test and the hold would silently never fire in production. Fix: add a `beamDeps.TaskLimit == nil` check (the seam is filled) to the existing integration test. The test must not call the seam, since that reads the host cgroup.

No redundant tests found. The new rows and subtests each cover a distinct behaviour.

**Tool calls (1 of 6)**
- `grep -rn "hubDobbyDeps\|TaskLimit"` over `internal/beamcli`: to see whether any test covers the `hubDobbyDeps` wiring. The manifest held only the `dobbydeps.go` diff.
