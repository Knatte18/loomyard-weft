```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. I took each behaviour the diff adds or changes and named the test row that pins it.

**Behaviour → pinning test**

1. The production default `startStallTimer` (`time.AfterFunc(bound, check).Stop`, `gitexec.go:55`) has no test. Every row that arms the check now installs `SetStallCheckForTest`, so no row runs the real timer. The row that ran it before, the silent row, now replaces it. A wrong bound, a `check` that never runs, or a broken `Stop` would pass every row.
2. A silent remote gets a stalled `GitError` with `Timeout` equal to the stall bound and exit code -1, and the call is retried once. The silent row, `gitexec_unix_test.go:212-242`, pins this. It also pins the retry warning count, the run and kill counts, and leftover trace files.
3. The check's "trace still size 0" condition (`gitexec.go:302`) is pinned in both directions:
   - The silent row requires a stall once the check runs on an empty trace.
   - The answering row, `:279-299`, fires the check only when the trace is non-empty and requires no stall. Dropping the size condition would make that row fail, so it does not pass trivially.
4. The retry attempt's check, with readiness `count >= attempt`, is pinned by the silent row. It needs `runs == 2`, `armed() == [stall, stall]` and the second stall to end the call.
5. The stall check is armed with `stallBound` only when `stallBound < deadline`:
   - Armed once: the answering row's `armed()` assertion at `:297`. Without it the row would pass even if the check were never armed, because a remote that is never checked is never stalled.
   - Never armed when the bound equals the deadline: the unarmed row's `armed()` assertion at `:272`. The existing `Stalled`, `runs` and `kills` assertions in that row would already fail if the check fired, since `ready` is always true. So this assertion mostly duplicates them.
6. The kill at the deadline with no retry is pinned by the unarmed and answering rows. That behaviour is unchanged by the diff.

**Findings**

- **MEDIUM, design (coverage gap):** No test pins the production stall timer's wiring, the `time.AfterFunc(stallBound, check)` default.
  - Location: `internal/gitexec/gitexec.go:55-57`.
  - Evidence: `gitexec_unix_test.go:219, 261, 283`. All three stall rows replace the timer, and the `grep` for `startStallTimer` found no other user.
  - The diff removed the only row that ran the real timer. A row that runs the real timer, with a stall bound well under the deadline and no `SetStallCheckForTest`, would still carry the start-up race.
  - A seam-free alternative is to keep the real timer in one row and make only its bound large enough to absorb start-up. That conflicts with the board entry's wish to avoid raising the number.
  - The behaviour can still be covered without that: a unit assertion that the default, called with a short bound, runs `check` once, and that its returned stop prevents the run.
- **NIT, test economy:** The `armed()` assertion in the unarmed row (`gitexec_unix_test.go:272-274`) repeats what the row's `Stalled == false`, `runs == 1` and one-kill assertions already catch.
  - The focus asked whether each `armed()` assertion checks something new. This one adds nothing the row did not already cover.
  - The silent row's `armed()` assertion (`:237`) is partly covered by `runs == 2` and the retry warning, but it adds that the bound passed to the timer is `stall`. Nothing else pins that, so keep it.

**Tool calls:** none. My context already held the diff, the test file and the production code.
