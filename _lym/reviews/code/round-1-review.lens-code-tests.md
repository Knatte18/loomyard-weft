```yaml
lens: code-tests
outcome: reported
```

**Per-behaviour list (lens: code-tests)**

- **Counter records a failed leaf with its first non-status line.** Pinned by `counters_test.go:137`, which has an error line after `=== RUN`.
  - Skipping the `--- ` line is pinned by `counters_test.go:104`, where `TestB` has an empty `FirstLine`.
  - A panic line is pinned by `counters_test.go:148`.
- **Parent of a failed subtest is left out.** Pinned by `counters_test.go:137`.
- **A passing test's output is not recorded.** Pinned by the `TestB` passing note in `counters_test.go:137`.
- **A parent and a sibling sharing a name prefix, such as `TestA` and `TestAB`, are both kept.** None. `Failures` filters on `Test+"/"`, and no row has a failing test whose name is a prefix of another failing test's name without a `/`.
- **`vstest` counter returns no failures.** Pinned by the shared `wantFailures` equality at `counters_test.go:223`, which is nil for the existing `vstest` rows.
- **Failures reach the run record (`tierrun.go:115`, `record.Failures = counter.Failures()`).** None. No test runs `runTier`, or reads a record of a failed `go-json` tier, and checks `Failures`. The grep over `gatecli` and `gateslot` tests found `Failures` only in `counters_test.go`.
- **A passing tier's record carries no `failures` key, and an older record without it still reads.** None. `runlog_test.go` has `RecordWithoutLaunchesReadsWithNone` (line 93) and a round-trip with `Launches` (line 32), but nothing for `Failures` or the `omitempty` key. This is D2's acceptance criterion.
- **Adopt notice for a failed tier lists failures as `<package> <test>: <first line>`, then the log path.** Pinned by `check_test.go`, "a failing tier ends the check failed…", with the exact notice string. An empty `FirstLine` (`TestB`) is covered there too.
- **Cap of ten with a closing `and <n> more`.** Pinned by `check_test.go`, "lists failed tests up to the cap and counts the rest".
- **A green tier's notice is unchanged.** Pinned by the existing `wantNotices` in `TestRunCheck`.
- **`check.json` holds the failures.** Pinned by the `ReadCheck` assertion with `slices.Equal` on `Tiers[1].Failures`.
- **A failed tier with no recorded failures (build failure, VSTest) reads as summary then `log <path>`.** None. Every failed-tier row has at least one failure.
- **Hook-wait restart subtest and EXTERNAL row.** These are the changed tests themselves, not production behaviour. `advanceUntilReturned` is exercised by both changed steps in `TestCreelCLI`.

**Findings**

1. LOW, scope, `internal/gatecli/tierrun.go:115` and `internal/gateslot/runlog.go:71`: nothing pins that the run record carries `failures` for a failed `go-json` tier, or that a passing tier omits the key. Evidence: `grep` over the `gatecli` and `gateslot` tests found only `counters_test.go` using `Failures`. Add a `runlog_test.go` round-trip row with `Failures`, plus an assertion that a record without them encodes no `failures` key.
2. LOW, scope, `internal/gatecli/counters_test.go` (`TestCounters` table): there is no row for two failing tests where one name is a prefix of the other without a `/`, such as `TestA` and `TestAB`. Add one such row to `TestCounters`.
3. NIT, scope, `internal/adoptcli/check_test.go` (`TestRunCheck`): there is no row for a failed tier with an empty `Failures` list. It should read as summary then `log <path>`. Add a `failedTier(1)` row with no failures.

Tool calls:
1. `grep` over `gatecli` and `gateslot` tests for `Failures`, `"failures"` and `Launches`, to check whether any test covers the run-record wiring or the `omitempty` key; the manifest held only the counter tests.
