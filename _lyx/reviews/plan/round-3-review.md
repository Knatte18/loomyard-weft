---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/19-orch-start-reconcile.md:22
    summary: The reconcile's mutation record (removed pairs, snapshot tags, moved branches) never reaches the mutations array of orch start, cleanup or wake, though card 17 says it does and PATTERN-mutation-record requires it
    fix: "Card 19: the `startCmd` envelope gains top-level `mutations` (the report's `Mutations.Entries()`, always an array) and `partial` (false) beside `reconcile` and `failed_pushes`; add a row to `TestStart_ReconcilesTheHubFirst` where the fake report carries one teardown mutation and the envelope's `mutations` holds it. Card 20: `Command`'s envelope gains the same two top-level keys in the dry run (an empty array) and with `--apply`; the force-case step of the verb test asserts the removed pair and the snapshot tag in `mutations`. Card 23: `WakeReconcile` gains `Mutations []string`, one rendered line per entry of the report's record, filled by `hibernatecli#wakeReconcile`; `reconcileAtWake` logs each at Info through `internal/logger`, since the daemon has no envelope and its notice stays one line; a `TestWatch` row asserts the fake world logged the lines. Cards 21 and 22 have no envelope, so `reconcileRun` and `reconcileVerdict` log the report's mutations at Info the same way. Card 17's sentence naming the `mutations` array of orch start, cleanup and wake is then true as written."
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/01-failed-push-record.md:34
    summary: For a records-side failed push the way-forward lines say to run `lyx fabric push` in the branch's worktree, which on a records record is the weft sibling, and `RequireWarpWorktree` refuses every fabric verb there
    fix: "In card 1, have `FailedPush` carry the pair's code worktree for a `records` record (derived from the weft path with `weftname`, or the prime's own worktree for the prime weft), and have `FailedPushWayForward`, `recordsConflictWayForward` and card 15's records-branch lines name `lyx fabric push` as run in that code worktree, while the `git reset --keep` and `git merge` steps stay in the weft worktree. The assertion rows of `TestFailedPushRecord` and card 6's and card 15's exact-line rows then pin the code worktree for the push verb and the weft worktree for the git steps. This changes card text only, no target list."
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/23-wake-reconcile.md:42
    summary: A stop's way-forward line is asserted at the judge but never at the two hops that print it, the wake notice (and its OriginReport mapping in hibernatecli) and the cleanup --apply refusal
    fix: "Card 23: split the report mapping out of `hibernatecli#wakeReconcile` into a pure `wakeReconcileFromReport(report fabricengine.OriginReport) hibernateengine.WakeReconcile` and add Create targets `plan:internal/hibernatecli#wakeReconcileFromReport` -> `func wakeReconcileFromReport(report fabricengine.OriginReport) hibernateengine.WakeReconcile` and `plan:internal/hibernatecli#TestWakeReconcileFromReport` -> `func TestWakeReconcileFromReport(t *testing.T)`, whose rows build an `OriginReport` by hand and assert that a stopped branch's `WayForward` reaches `WakeRun` exactly, that `PushLockHeld` becomes the parked-push flag, and that a D3 stop carries its line. Add to the `TestWatch` rows for a stopped pair, a D3 stop and a busy ended entry an assertion that the queued notice contains the fake `WakeRun`'s exact way-forward line followed by `lyx resume <slug>`, or by `lyx cleanup --apply` for the ended entry. Card 20: the two refusal steps of `TestCleanupVerb_RefusesOutsideThePrimeThenListsAndThenRemovesEverySeededLeftover` (tracked change on a Bolt replay path, both-sides change of the prime weft) assert the refusal text contains the stopped branch's exact `WayForward` line, the dirty line and `recordsConflictWayForward`'s line with `lyx board sync`. Checked with `lyx loom validate-plan --overlay` on card 23 with the two new Create targets: no new blocking finding against the baseline."
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/09-push-retry-and-record.md:72
    summary: No test reaches the Prime flag `pushClassified` sets, so the prime's own code-side failures (Finalize's parent push of the prime) are never shown to print `git push` in the prime, and Publish and Finalize get class rows only for rewound
    fix: "In card 9's `TestPushFailureClasses`, add rows for a code-side push run from the hub's prime worktree through `Fabric.PushBranch`: a rejection writes a `not_fast_forward` record with `Prime` set and returns an error whose text ends in the prime form of `codeDivergedWayForward` (`git push` in the prime, no `lyx fabric push`), and a 403 hook writes an `auth` record with `Prime` set whose line names `git push` in the prime. Add to `TestFinalize_MergeMarkPushOrderAndVerdict` rows for a `*PushFailedError` of class `not_fast_forward` with `Prime` set and of class `auth`, each asserting the reason ends in `FailedPushWayForward`'s exact line with no `push it by hand`. Add to Publish's `TestPublish_StuckBeforePullRequest` (add it to card 9's Edit list as `internal/landingshed#TestPublish_StuckBeforePullRequest`) a row for each of `gone`, `rewound` and `auth` asserting the Stuck reason is `push failed: ` plus the error text ending in the class's line. Also add the `auth` Prime row to `TestFailedPushRecord`'s assertion list in card 1."
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/09-push-retry-and-record.md:84
    summary: Card 9 says loomcli's pushFailureWayForward prints the Unwrap-ed push error so the line appears once, but PushPairAnchored's both-sides failure is an errors.Join, so one side's error text is dropped or one side's line is printed twice
    fix: "In card 9, drop the `Unwrap` printing rule: `commitRecordsVerb` prints `err.Error()` unchanged, since every `*PushFailedError` in it already ends with its own class's line, and appends `; way forward: ` plus `pushFailureWayForward`'s text only when `errors.As` finds no `*PushFailedError`; the same for `writeHubRules`. Add a row to `TestCommitRecordsVerb_ErrorEnvelopes` where both sides fail (an `errors.Join` of a `gone` and an `auth` `*PushFailedError`) asserting each side's line appears exactly once."
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/plan/15-origin-reconcile-judge.md:78
    summary: The dirty way-forward line offers `lyx fabric commit` for a weft path and `git commit` for a code path, but a dirty Bolt path, which card 16 asserts as the line exactly, is neither, and `lyx fabric commit` is refused in the board worktree
    fix: "In card 15, add a Bolt form to the dirty line, commit the named paths with `git commit` in the hub's `_board` worktree or restore them with `git checkout -- <path>` there, then re-run `lyx cleanup --apply`; card 16's Bolt dirty row then asserts that form. Card text only, no target list."
---

### [BLOCKING] The reconcile's mutation record does not reach orch start, cleanup or wake

**Location:** `_lyx/plan/19-orch-start-reconcile.md:22`, `_lyx/plan/20-cleanup-reconcile.md:19`, `_lyx/plan/23-wake-reconcile.md`, `_lyx/plan/17-origin-reconcile-ended.md:23`
**Issue:** Card 15 gives `OriginReport` an embedded `MutationRecord`, and card 17 extends it with each teardown's mutations "so a removed pair, its tags and its deleted branches appear in the `mutations` array of orch start, cleanup and wake".
No later card carries it there.
`PATTERN-mutation-record` requires the envelope of a mutating verb to carry `mutations` as an array and `partial` as a bool, and `fabriccli/envelope.go` sets both at the top level.
Card 19 puts the report under the key `reconcile`, so its record would sit at `reconcile.mutations` and the envelope has no top-level `mutations` or `partial`, and `startCmd` and cleanup's `Command` emit neither today.
Card 20 does the same.
Card 23's `WakeReconcile` is the engine's own view of the report with no record in it, and the daemon has no envelope, so a pair removed and a snapshot tag written at wake leave no trace but a one-line notice.
Cards 21 and 22 drop the record the same way.
After this task `lyx orch start` and `lyx cleanup --apply` remove pairs, tag branches and fast-forward worktrees, so both are mutating verbs under the pattern.

### [MEDIUM] Records-side `lyx fabric push` is named for the weft worktree

**Location:** `_lyx/plan/01-failed-push-record.md:34`
**Issue:** Card 1 names `lyx fabric push` "in the branch's worktree for a `code` or `records` record".
For a `records` record the repo path and branch are the weft's, and the weft sibling is not a warp worktree: `fabriccli.resolveWarpLocation` refuses every fabric verb from there through `RequireWarpWorktree` (`internal/fabricengine/fabric.go:134`).
`recordsConflictWayForward` has the same gap, since its closing step is the side's push verb after git steps in the weft worktree, and card 6 and card 15 reuse it with `lyx fabric push` for a weft.
The exact-line test rows would pin a command the operator cannot run from the directory the line sends them to.
The `Prime` code form (`git push` in the prime) is runnable, and `lyx fabric push` in the prime's code worktree does push the prime weft, so the fix is to name which worktree each command runs in.

### [MEDIUM] The way-forward line is asserted at the judge but not where it is printed

**Location:** `_lyx/plan/23-wake-reconcile.md:42`, `_lyx/plan/20-cleanup-reconcile.md:19`
**Issue:** Card 15 asserts each state's `WayForward` and `Summary` exactly, and cards 16 and 21 and 22 assert it for their own refusals through fakes.
The wake notice and the cleanup refusal are the two hops with no such assertion.
Card 23's `TestWatch` rows use a fake `Reconcile` and describe the notice's contents without saying the row asserts the line, and the mapping `hibernatecli#wakeReconcile` that carries `BranchOutcome.WayForward` and `PushLockHeld` into `WakeRun` is only covered by `TestHibernateVerbs_WireTheHub`, which asserts the seams are filled.
Card 20's refusal rows assert that the verb refuses and runs no segment, not that the message carries the line, though the Intent says it names it.
A mapping that composed its own line, which card 23 forbids, would pass every planned test.

### [MEDIUM] The Prime flag and the Finalize and Publish class rows are never reached

**Location:** `_lyx/plan/09-push-retry-and-record.md:72`, `_lyx/plan/09-push-retry-and-record.md:83`
**Issue:** The question was whether every command named for a failed push of the prime's code branch can push it.
`git push` in the prime worktree can, `lyx fabric push` is never named for a `Prime` record (card 1's rows assert it for `transport` and `not_fast_forward`), and `PushPairAnchored` in the prime pushes records only, so the lines are right.
What is missing is the path that sets the flag: `pushClassified` sets `FailedPush.Prime` from the `IsPrimeWorktree` predicate, and no row of `TestPushFailureClasses` pushes a rejection from the prime (its one prime row is a rewind that pushes and records nothing), so a flag that is never set would print `lyx fabric push` for the prime and pass.
Finalize's parent push of the prime is the production path, and `TestFinalize_MergeMarkPushOrderAndVerdict` gains only a `rewound` row for a non-prime parent.
Publish's `push failed: <error>` Stuck gets no row at all, so no test reaches `gone`, `rewound` or `auth` through it, and `auth` is reached nowhere through Finalize.
Card 1's `Prime` rows leave out `auth`, which is a reachable prime class.

### [LOW] A two-sided failure prints one line twice or loses one

**Location:** `_lyx/plan/09-push-retry-and-record.md:84`
**Issue:** Card 9 has `pushFailureWayForward` find a `*PushFailedError` with `errors.As` and the caller print its `Unwrap`.
`PushPairAnchored` returns `errors.Join` of the two sides when both fail, which `errors.As` resolves to the first `*PushFailedError` only.
Printing the joined error's `Error()` repeats the first side's line after the appended `way forward`, and printing the first side's `Unwrap` loses the second side's failure.
The "exactly once" assertion in `TestCommitRecordsVerb_ErrorEnvelopes` covers single failures only.

### [LOW] The dirty line has no form for Bolt

**Location:** `_lyx/plan/15-origin-reconcile-judge.md:78`
**Issue:** A tracked change on a path a Bolt replay touches stops every scoped entry, and card 16 asserts Bolt's `WayForward` as the dirty line exactly.
The line offers `lyx fabric commit` for a weft path and `git commit` for a code path, and a Bolt path is in the hub's `_board` worktree, where `lyx fabric commit` is refused.
The stop that halts every entry and refuses `lyx cleanup --apply` is the one with no runnable line.

## Rejected

- Evaluation point of `finalizeEntered` in hub scope: pinned.
  Card 16 calls it once per candidate slug right after `readEndedSet`, before Bolt's replay and any move, and passes the filtered set to every replay and to `reconcileEnded`; card 17's hub-scope row (Finalize as current producer, no history row, local parent behind origin, entry done on origin) is ended although the same reconcile fast-forwards the parent, which fails if the read moves after the fast-forward.
  Not a defect.
- The `PushPairAnchored` prime limitation (card 1): every command named for a `Prime` record is `git push` in the prime, which pushes the prime's code side, and none names `lyx fabric push`; the prime-weft lines use `lyx fabric push` in the prime, which does push the records side there.
  Only the untested flag (F4) remains.
- Card 17's `gone`, `busy` and ended-entry lines, card 18's seams and cards 13 and 14's removal and snapshot carry their record, tag and wake-hold rules through the tests listed, with the exception of F1.
