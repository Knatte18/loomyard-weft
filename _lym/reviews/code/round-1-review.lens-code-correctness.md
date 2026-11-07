```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, tracing each row the diff adds or changes through `loadSkillTurn` on the fake clock, with a 1 s poll interval from `fastConfig`.

**List (function or branch, case run, verdict)**

- **Shared upper bound `elapsed >= wantHeld+apiErrorRetryBackoff`, ceiling row (six API errors).** Waits are 30, 60, 120, 240, 300 and 300 s, plus 1 s of poll sleep after each retype. Elapsed is 1056 s against 1050 ≤ elapsed < 1080, so it is correct. A tripled back-off or no ceiling gives about 1890 s and fails. Only a ceiling within about 24 s of 300 s would slip past, which is acceptable.
- **Same bound, doubling row (three errors).** Elapsed is 213 s against 210 ≤ elapsed < 240, so it is correct. A tripled back-off gives 30+90+270 and fails.
- **Same bound, busy-session row.** The busy send sets `retypeAt` to `now + backoff` (60 s), so the retype is at 90 s and elapsed is 91 s against 90 ≤ elapsed < 120. It is correct. A non-doubled busy back-off gives 61 s and fails the lower bound.
- **Same bound, interactive and notifier rows (two errors).** Elapsed is 92 s against 90 ≤ elapsed < 120, so it is correct.
- **Same bound, retry-turn-errors row.** Elapsed is 31 s against 30 ≤ elapsed < 60, so it is correct.
- **Same bound, carried-turn row.** The carried START is appended at the first load-poll `Status` call and the carried STOP at the next. Elapsed is 31 s against 30 ≤ elapsed < 60, so it is correct.
- **Same bound, existing late-end API-error row.** The turn ends late at 8 s, then the back-off and retype add up to 39 s against 30 ≤ elapsed < 60. It is correct, with 9 s of poll overhead inside the 30 s slack.
- **Removed `wantHeldAtMost` on the ceiling row.** `wantHeld` is 30·15 + 2·300 = 1050 s, which equals the schedule `apiErrorRead` produces. The 6 s poll overhead sits inside the 30 s slack. The new bound is tighter than the old 25-minute one, so removing the field loses nothing.
- **New row "an open turn past its timeout that then ends loads its skills".**
  - `lateEnds {0: 8}` with `OpenTurns` writes START at once, so `open` is true.
  - The `!open` timeout branch is never reached. The deadline branch passes while 1 h remains.
  - The turn end is appended at the eighth load-poll `Status` call and read on the next iteration, so elapsed is 8 s against 0 ≤ elapsed < 30.
  - If the code returned `ended=false` at the 50 ms timeout, one skill would be skipped and `wantSkips` 0 would fail. Correct.
- **New row "an open turn after a hold fails the start at the deadline the hold moved".**
  - Turn 0 errors at T0, so `since` is T0. The retype goes out at T0+30, and the START of the hanging turn 1 is read at T0+31, which resets `began` to T0+31.
  - `heldDeadline` is therefore 1 h + 31 s, not 1 h + 30 s. The start fails at 3631 s, which satisfies `elapsed >= 1h + 30s`.
  - The row fails if the deadline were not moved at all, since it would fail at 3600 s.
  - The error text names `outcome "timeout"`, and the hold-over log is expected. Correct.

**Findings**

1. **LOW, design.** `startup_test.go:1932` (failure branch) checks only a lower bound on `elapsed` for rows with `wantFailed`.
   - The new row's claim, "at the deadline the hold moved", is therefore pinned from below only.
   - A `heldDeadline` or `endLoadHold` that moved the deadline too far, such as a hold counted twice or a doubled `began - since`, would still fail the row with `elapsed >= 1h+30s`.
   - Only a deadline moved too little, or not at all, is caught.
   - The diff added the upper bound for succeeding rows but not here.
   - The exact fail time is 1 h + 31 s, so an upper bound such as `wantFailedAfter + apiErrorRetryBackoff` would hold.
   - Evidence: `startup_test.go:1932-1934` and `1811-1821`, and `loadturn.go:64-71` and `163`.

No other correctness defects found. The virtual-time arithmetic above is not in the diff or the summary; it comes from `loadturn.go:117-171` and the `skillReed` fake at `fakes_test.go:662-837`.

**Tool calls:** none. The manifest already held `loadturn.go`, `startup_test.go`, `fakes_test.go` and the fake clock.
