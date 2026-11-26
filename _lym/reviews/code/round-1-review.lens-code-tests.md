```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test that would fail if the behaviour broke.

**Per-behaviour list**

| # | Behaviour | Test that pins it |
|---|---|---|
| 1 | `CheckParent` allows a write to `notePath` at its absolute spelling | `TestCheckParent` row "write to Master's friction note is allowed" (`audit_test.go:386`) |
| 2 | `CheckParent` allows the relative `_lym/...` spelling | `TestCheckParent` row "relative write to Master's friction note is allowed" (`audit_test.go:393`) |
| 3 | `CheckParent` still flags a write to a different note (`-2`) | `TestCheckParent` row "write to another friction note is a hard error" (`audit_test.go:400`) |
| 4 | `CheckParent` allows the link-resolved spelling, a transcript path through the real records worktree (claimed in the `audit.go` doc and `summary.md`) | **none.** No `TestCheckParent` row puts a real `_lym` link under `workdir`, so the canonical branch of `isWriteOf` is never reached for the note. |
| 5 | An empty `notePath` allows no note | Indirect only: callers pass `""` (`acceptbatch_test.go:39`, `auditseverity_test.go:152`), but no row writes a note while `notePath == ""` |
| 6 | `Run` saves `State.MasterNotePath` at spawn (`runlevel.go:805`) | **none.** No test reads `MasterNotePath` (grep of `*_test.go` finds nothing). The mutation report lists `runlevel.go:805` "drop statement" as a survivor. |
| 7 | record-batch and the run-exit audit pass `State.MasterNotePath` into `CheckParent` (`recordbatch.go:413`, `runlevel.go:1085`) | **none.** No `TestRecordBatch_*` or `TestRun_*` sets `MasterNotePath` and records a note write in the transcript. |
| 8 | `reconcileAcceptHead` accepts fix commits up to `FixedHead` | `TestAcceptPendingAudit_AcceptsVerifyFixCommits`, first half (`suspect_test.go:385-394`) |
| 9 | `reconcileAcceptHead` refuses a non-merge commit above `FixedHead`, naming `reset --to last-batch-head` | `TestAcceptPendingAudit_AcceptsVerifyFixCommits`, second half (`suspect_test.go:396-403`) |
| 10 | `reconcileAcceptHead` ignores `FixedHead` when `lastHead` is not its ancestor, when it equals `lastHead`, or when the SHA is missing | **none.** No row sets a `FixedHead` that is stale, equal to `lastHead` or missing. |
| 11 | `pass()` records HEAD as the fixed head when HEAD moved above `preFix` | `TestVerifyGate_DirtyTreeRecordsPreFixHead` (`verifygate_test.go:640, 650`) |
| 12 | `pass()` records no fixed head when HEAD did not move, or when there was no failure (`preFix == ""`) | **none.** `fixedHeads` is asserted only at `verifygate_test.go:650`. Dropping the `head != preFix` guard passes every test. |
| 13 | `recordGatePass` clears `PreFixHead`, records `FixedHead`, keeps it when given `""`, and leaves a missing `state.json` absent | `TestGateStateWrites` (`verifygate_test.go:1008-1047`) |
| 14 | `recordGatePass` saves when only `FixedHead` changes (`changed = true`, `verifygate.go:305`) | **none.** Every `TestGateStateWrites` step that passes a fixed head starts with `PreFixHead` set, so `changed` is already true. This is a mutation survivor. |
| 15 | `setPreFixHead` refactored onto `mutateGateState` | `TestGateStateWrites` steps "FailureRecordsPreFix" and "SecondFailure"; existing `TestVerifyGate_*` |
| 16 | `lym webster reset` clears `FixedHead` (`webstercli/reset.go:152`) | **none.** `reset_test.go:393` asserts only `PreFixHead == ""`. No webstercli test mentions `FixedHead`. |
| 17 | `recordsSideStatus` / `splitRecordsSidePaths`: absent and committed-unchanged paths clear; edited and untracked paths refuse toward a restore; webster run state takes the reset route | `TestAcceptPendingAudit_RecordsSidePath` rows `AbsentNote`, `CommittedNote`, `EditedNote`, `UncommittedNote`, `RunState` (`suspect_test.go:436-440`) |
| 18 | `pendingPathsWayForward` gives records-side paths the records route (cleared note, restore step, no reset to start) | Same test, through `pendingAuditFindingsError` (`suspect_test.go:465, 474, 482`) |
| 19 | Records-side paths under the reports or plan directories are excluded | **none** directly. `RunState` covers only the webster dir; no row puts a path under `ReportsDir`. Plan paths are split off before this check, so that part is moot. |
| 20 | An ignored records-side path counts as changed | **none.** No row writes a path the records repo ignores. |

The mutation survivors at `findingtext.go:162-181`, `auditledger.go:170` and `runlevel.go:1305` would be killed by rows of `TestAcceptPendingAudit_RecordsSidePath`. For example, negating `:181` turns `AbsentNote` into the reset route, and dropping `rest = tracked` at `:1305` breaks `AbsentNote`'s "1) lym webster accept-audit" way forward. The coverage map simply does not hold that tier-1 test: the reach report lists only `TestPendingFindingsText` and `TestRun_*` reaching `recordsSideStatus`. So those survivors are no finding. The remaining survivors (`:162`, `:166`, `:170`, `:178`) are error returns and a pathless guard, which no test can reasonably kill.

**Findings**

1. **MEDIUM, design. `internal/websterengine/runlevel.go:805` (also `recordbatch.go:413`, `runlevel.go:1085`).**
   Nothing tests the end-to-end fix for Idea 1. `Run` saving `State.MasterNotePath` and the two audit sites reading it are unpinned, so dropping the save would bring the reported bug back with every test green.
   Evidence: the mutation report's survivor at `runlevel.go:805`; grep finds no `MasterNotePath` in any `_test.go`.
   Fix: add an assertion to an existing `TestRun_*` (for example `TestRun_MasterSpawn`) that the saved state carries the note path from the prompt. Add a row to `TestRecordBatch_AuditOutcomes` or `TestRun_DoneOutcome` where the transcript writes that path and no `parent-write` finding results.

2. **LOW, design. `internal/webstercli/reset.go:152`.**
   Nothing tests that reset clears `FixedHead`, although the summary claims it does.
   Evidence: `reset_test.go:393` asserts only `PreFixHead`.
   Fix: in that test, seed `st.FixedHead` next to `PreFixHead` and assert both are cleared.

3. **LOW, design. `internal/websterengine/auditledger.go:203-209`.**
   The ancestry guard in `reconcileAcceptHead` has no refusing row: a `FixedHead` that `lastHead` does not precede, one equal to `lastHead`, or a missing SHA.
   Evidence: `suspect_test.go:385-403` covers only an ancestor `FixedHead`.
   Fix: add a row to `TestAcceptPendingAudit_AcceptsVerifyFixCommits` with a `FixedHead` off `lastHead`'s line, reached from HEAD only through a merge, and expect `ErrAuditNotAcceptable`.

4. **LOW, design. `internal/websterengine/verifygate.go:395-402` and `:305`.**
   Two gate branches are unpinned:
   - the "no fixes, no fixed head" branch of `pass()`: a pass with HEAD unmoved, or with no prior failure, should record `""`;
   - `recordGatePass` saving when only `FixedHead` changes.

   Evidence: `fixedHeads` is asserted only at `verifygate_test.go:650`; every `TestGateStateWrites` step with a fixed head has `PreFixHead` already set. The mutation survivor sits at `verifygate.go:305`.
   Fix: assert `f.fixedHeads == [""]` in a gate test that passes after a failure at an unchanged head. Add a `TestGateStateWrites` step that passes a new fixed head while `PreFixHead` is already empty.

5. **LOW, design. `internal/websterengine/audit.go:96`, via `isWriteOf`.**
   The link-resolved note spelling claimed in `CheckParent`'s doc and `summary.md` has no row.
   Evidence: `TestCheckParent` uses only the fake paths `/hub/...` (`audit_test.go:327-447`), and `audit_test.go` contains no link.
   Fix: add a `TestCheckParent` row with a real `_lym` dir link under a temporary `workdir`, writing the note through the link target's path.

6. **NIT, design. `internal/websterengine/findingtext.go:166-170` (the exclusion loop).**
   No row checks that an ignored records-side path counts as changed, or that a path under `ReportsDir` is excluded.
   Evidence: the rows at `suspect_test.go:436-440`.
   Fix: add `Ignored` and `ReportsDir` rows to `TestAcceptPendingAudit_RecordsSidePath`.

Outside this lens, for the correctness lens: a tracked records-side file that has been deleted reads as "absent", and therefore clean (`findingtext.go:185`). No row covers it either.

Tool calls:
1. Grepped `_test.go` files for `MasterNotePath`, `FixedHead` and the records-side notes, to find which tests assert the new state fields; my context held only the diff's own tests.
2. Grepped `verifygate_test.go` and `webstercli/*_test.go` for the `cleared`/`fixedHeads` assertions and the reset tests' `PreFixHead`/`FixedHead` checks; the reset tests were not in my context.
3. Grepped `audit_test.go` for link setups and the `runlevel`/`recordbatch` tests for the note path, to check link-spelling and `Run` coverage the manifest lacked.
