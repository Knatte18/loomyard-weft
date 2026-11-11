```yaml
lens: code-tests
outcome: reported
```

**Behaviour → pinning test**

1. A green check renders the zero mark (`checkMarkOf`, `check.go:327`): `TestRunCheckMarks`, row `GreenShowsEachTierInFlightThenClearsTheMark`, final `{}`. Pinned.
2. A failed check with a `FailedTier` shows the one-item "failed at tier<N>" text (`check.go:331`): `TestRunCheckMarks`, row `FailedEndsWithItsFirstFailedTierAlone`. Pinned.
3. A failed check with no tier ended (`check.go:329`): `TestRunCheckMarks` at `check_test.go:582`, unchanged and still pinned.
4. A failed check with tiers but no `FailedTier`, giving "failed; lym beam status shows why" (`check.go:333`): none. A grep of `check_test.go` for that text finds nothing, and the only failed-state rows are the tier-failed one and the no-tier one. This is a new branch with no row.
5. `primeMark` leaves an empty own mark out of the join (`check.go:357`): `TestPrimeMark`, rows `ClearedOwnMarkShowsTheRunningOtherSubjectsAlone` and `ClearedOwnMarkWithNoOtherRunningSubjectClears`. The changed row `RunningOtherSubjectsAreJoinedAfterItSortedWithStateRunning` still pins the non-empty join. Pinned.
6. The adopt envelope's per-check wording (`adopt.go:258`): `TestAdopt`, both message assertions. Pinned.
7. `attach` and `up` are refused in the Bolt checkout and a records sibling, with exit 1, the verb, fabric's reason and the way forward, before any config read: `TestRunCLI_AttachAndUp_RefuseFabricCheckouts`, four rows. The checkouts hold no config, so the order is pinned too. Pinned.
8. A legitimate worktree or the prime is still accepted by `attach` and `up`: no unit test in the diff. The `up` and `attach` calls in `smoke_coldstart_test.go` and `smoke_debuglog_test.go` cover it, but they need real tmux and are not in the default tier. I did not read their build tags, so that is unverified. This is the weakest item.
9. Other verbs still run in the checkouts, so `down` still removes a session an older binary booted there: none. This is stated in `doc.go`, the commit message and `summary.md`, but no test runs a non-boot verb in a Bolt checkout and sees it pass the guard.

**Findings**

- LOW, `design`, `internal/adoptcli/check.go:333`: the failed-without-`FailedTier` branch ("failed; lym beam status shows why") has no test row. Add a `checkMarkOf` case or a `TestRunCheckMarks` row with a failed record that has tiers and a nil `FailedTier`. No evidence it is unreachable, so this stays a coverage gap.
- LOW, `design`, `internal/reedcli/cli.go:166`: nothing in the default test tier shows `attach` or `up` is still accepted in a drivable worktree, so a guard that over-refused would pass `TestRunCLI_AttachAndUp_RefuseFabricCheckouts`. The refusal test has only refusal rows. A row run in a legitimate worktree would pin this.
- LOW, `design`, `internal/reedcli/cli.go:166`: the guard is limited to `attach` and `up`, but no row asserts another verb (such as `down`) passes it in a Bolt checkout. The goal's accepted-path claim and `doc.go` rest on that. This is optional; a negative row for one non-boot verb would cover it.

**Test economy**

- `TestRunCLI_AttachAndUp_RefuseFabricCheckouts` duplicates no existing test. The only other `attach` or `up` calls in `internal/reedcli` are the tmux smoke tests, which drive a real session, not a refusal.
- The redundancy report lists the adoptcli tests as unique.
- The new test adds no assertion on static prose, and no test was added for a review fix.

**Tool calls (2)**

1. Grep of `internal/adoptcli/*_test.go` for the failed-mark texts and `checkMarkOf(`: my context did not show whether any test reaches the failed-without-`FailedTier` branch.
2. Grep of `internal/reedcli/*_test.go` for `"up"`, `"attach"` and `"down"`: my context did not show whether any test covers the accepted path or the other verbs.
