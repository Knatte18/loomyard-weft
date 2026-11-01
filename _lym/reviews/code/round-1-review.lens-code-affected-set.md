```yaml
lens: code-affected-set
outcome: reported
```

Task: list, per change, the set of things the diff's changes force outside the lines it touches (lens code-affected-set).

**Per change: the set found, covered or not**

1. **New required seam `PullRequestDeps.RecordChanges`** (`internal/landingshed/deps.go`).
   - Every place that builds or lists `PullRequestDeps` seams is covered. The seam lists I found by grepping `PauseCovered`, a sibling seam, and I checked each one:
     - The `checkWiring` nil case in `prreview.go`.
     - The nil-wiring table in `prreview_test.go`, plus `prReviewTestDeps`.
     - The wait-test world in `prreview_wait_test.go`.
     - Production wiring in `loomcli/landingdeps.go`.
     - The `intentionallyNil` tables in `loomcli/wiring_seams_test.go` and `battencli/wire_seams_test.go`.
     - `envkit.LandingDeps`.
   - I found no other constructor of `PullRequestDeps`.
   - `prreview_session_test.go` and `prreview_wait_test.go` override seams on top of the shared builders, so they need no change.

2. **`wait` and `pause`: a record change ends the sleep early.**
   - The description lives in the `wait` doc comment and the `internal/landingshed/doc.go` wait paragraph. Both cover it.
   - `docs/overview.md:270` only names the row and states no timing, so it needs no edit.
   - I found no stencil or contract text describing the wait's timing.

3. **`pattern/PATTERN-daemon-wakeups.md` bullet added.**
   - The `PATTERN.md:62` one-liner is unchanged, as it should be.
   - `cmd/lym/daemonwakeups_test.go` has no reference to `fswatch`, `landingdeps`, `landingshed` or `loomcli`. The bullet is not enforced by that test, and the test does not list the new site.

4. **`watchFileChanges` extracted, and `openLandRequestWatch` now uses it.**
   - Sole callers: `landingdeps.go:366` and `pawlwiring.go:155`. `sharedbootstrap.go:369` still calls `openLandRequestWatch`.
   - No test or doc asserts the old Warn text. A grep for "could not be watched" finds only the new code.
   - The Warn text and fields changed: the message went from "the land request directory could not be watched, so Beam-Await polls" to a generic one, and gained a `waiter` field. This is a behaviour change to Pawl's log with nothing asserting it. See finding F1.

5. **Commit message.** It carries `Co-Authored-By`. That is the working commit, not a landing commit, so the landing credit-block rule (`f666ba87`) does not apply. No `Pattern-Departure:` trailer is needed, because no PATTERN entry is departed from.

**Findings**

- **F1, LOW, consistency**, `internal/loomcli/pawlwiring.go:146`.
  - The `openLandRequestWatch` comment says the watcher "lives until the process ends". The stop function is now discarded with `changed, _ :=`, which is still true but is easy to miss.
  - The generic Warn in `watchFileChanges` (`internal/loomcli/landingdeps.go:378`) lost Pawl's specific message, "the land request directory could not be watched". Operators grepping for the old text will no longer find it.
  - Fix: keep a call-site-specific message, or accept the change and say so in the commit.

No other member of the affected set is left wrong.

**Tool calls (4)**
1. Grep for `PauseCovered` and `Co-Authored-By`. My context did not list every seam site or the convention docs.
2. Grep of `PATTERN.md`, docs and contracts for the PR-Review wait wording, to find docs that describe the wait.
3. Grep of `_test.go` files for `daemon-wakeups` and `fswatch`, to find tests that enforce the pattern entry.
4. Grep of `cmd/lym/daemonwakeups_test.go`, to see whether it enumerates wait sites.
