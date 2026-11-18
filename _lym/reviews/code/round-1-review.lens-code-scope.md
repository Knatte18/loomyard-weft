```yaml
lens: code-scope
outcome: reported
```

My task was to trace each change in the diff to the board entry's goal points. Everything traces, and the one finding is a wrong line count in the change description.

**Decisions.** The record has no plan and no `Added after Discussion` entries, so the decisions are the board entry's goal points:

- **G1:** run `deadcode` over every main, without and with `-test`.
- **G2:** delete each unreachable function and the tests that cover only it. Code that only tests reach goes the same way, unless it is a test helper.
- **G3:** check each hit before deleting it. A function reached through reflection, a registry, `go:linkname` or another platform's build tag stays, and the report names it with the reason.
- **G4:** no behaviour change, and nothing a user can call is removed.
- **G-report:** the report lists removed lines per package and every kept hit with its reason.

**Per change**

- **C1** `internal/runfsm/state.go`: removes `Order`. Traces to G2, because `Order` is production code that only tests reach and it is not a test helper; its only caller was its own subtest.
- **C2** `internal/runfsm/state.go`: rewrites the `Derive` doc so it no longer names `Order`. Forced by C1, because the old doc would be stale.
- **C3** `internal/runfsm/state_test.go`: removes the `OrderListsEveryStateHighestFirst` subtest and the `reflect` import. Traces to G2, which deletes the tests that cover only the removed function.
- **C4** `cmd/lym/testonlycode_linux_test.go`: removes the `internal/runfsm.Order` allowlist entry. Forced by C1, because the guard fails on an entry that matches no function.
- **C5** removes test helpers that no test calls. Traces to G2's first sentence: `deadcode -test` reaches these from nothing, so they are not "code only tests reach", and the test-helper exception does not apply. The helpers removed:
  - battenshed `resumeFrom`
  - burlerengine `stopCalls`, `settled`, `fakeShuttle.stop`, `blocked`
  - fabricengine `pairPermittedRoots`, `reconcilePairFor`
  - loomshed `testGateTiers`
  - orchcli `sendTurn`
  - reedengine `answerSession`
  - reportengine `commitOffChain`
  - shedadapters `stopLog`, `probedHandle`, `withRubric`
  - shuttleengine `skillStartFixture`
  - websterengine `failOutput`
- **C6** removes things that were only there for the C5 helpers. Forced by C5:
  - the `slices`, `fmt` and `sync` imports;
  - the burlerengine `stoppedGUIDs` field and the line that appended to it in `Stop`, since only `stopCalls` read it;
  - the `fakeHandle.Stop` doc rewrite that drops "records the call".
- **C7** `internal/shuttleengine/startup_test.go`: rewrites the `skillStartFixtureWith` doc so it stands alone. Forced by C5, because the old doc was written in terms of the removed `skillStartFixture`.

**Per decision**

- **G1:** no diff change, as expected. `_lym/landing/summary.md` reports both runs.
- **G2:** C1, C3 and C5.
- **G3:** no diff change, as expected. The summary's "Kept" table names each kept hit with its reason. No remaining reference to any removed symbol exists in any file, whatever its build tag (grep over the whole tree), so nothing from another platform was deleted.
- **G4:** C1 is the only production removal, and no main reaches `Order`. C2 to C7 are test-only.
- **G-report:** `_lym/landing/summary.md`. Its per-package figures match `git diff --numstat` except shuttleengine (see F1).

**Findings**

- **F1** — severity LOW, class `consistency`
  - **Location:** `_lym/landing/summary.md`, the shuttleengine row of the Removed table.
  - **Summary:** the report gives `internal/shuttleengine` 6 net lines removed. The diff removes 8 and adds 3, which is 5 net, so G-report's per-package count is wrong for that package.
  - **Evidence:** `git diff --numstat 9c5bd131..HEAD` shows `3 8 internal/shuttleengine/startup_test.go`. The summary's `| \`internal/shuttleengine\` | 6 |` row is in the summary text I read in step 1.

Tool calls:
1. `git diff --numstat 9c5bd131..HEAD`: per-file added and removed counts, to check the report's per-package line figures. My context had the diff hunks but no totals.
