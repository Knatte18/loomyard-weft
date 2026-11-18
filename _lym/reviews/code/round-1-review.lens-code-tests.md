```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I named the test that would fail if it broke.

**Per behaviour, the test that pins it**

- **`runfsm.Order` removed.** The removed function was the only thing its subtest `OrderListsEveryStateHighestFirst` tested, so the subtest goes with it (`internal/runfsm/state_test.go`, removed hunk). `Derive` still walks the same unexported `order` table.
  - `TestDerive` pins that table's precedence through its "X over Y" rows: `LandedOverCancelled`, `CancellingOverPaused`, `PausedOverHalted`, `HaltedOverLanding`, `LandingOverRunning`, `RunningOverStarting`, `QueuedOverNoRun` and `CancelledOverPaused` (`internal/runfsm/state_test.go:56-63`).
  - Two adjacent pairs have no "over" row, and neither can overlap:
    - Cancelled/Cancelling depend on `Finish != nil` versus `Finish == nil` (`state.go` `order`).
    - Starting/Queued: Queued requires `!r.BattenFound` and Starting requires `r.BattenFound`.
  - So no reachable precedence lost its pin.
- **`Derive`'s doc comment no longer names `Order`.** This is prose only; there is no behaviour to pin.
- **The guard allowlist entry `internal/runfsm.Order` was removed** (`cmd/lym/testonlycode_linux_test.go:58`). The guard pins it: an entry that matches no unreachable function fails the guard, and keeping the entry would fail it now. Removing it without removing `Order` would fail the guard as an unreachable production function (`pattern/PATTERN-test-only-code.md`, Enforcement).
- **`fakeHandle.Stop` in `internal/burlerengine/engine_test.go:211-222` no longer appends to `stoppedGUIDs`.** This is a change to a test double, not to production behaviour. Nothing read that field except the removed `stopCalls`; a grep found no `stopCalls` caller in any file. Stop's remaining behaviour (end Wait as died, or fail with `stopErr`) is unchanged, so the tests that use the fake still pin it.
- **The other test helpers were deleted:** `resumeFrom`, `pairPermittedRoots`, `reconcilePairFor` (fabricengine), `testGateTiers` (loomshed), `sendTurn`, `answerSession`, `commitOffChain`, `stopLog`/`probedHandle`, `withRubric`, `skillStartFixture`, `failOutput`, and `fakeShuttle.stop`/`blocked`/`fakeHandle.settled`.
  - None of these is production behaviour, and nothing in any test still calls them. The greps found no remaining references in any file. The same-named symbols in `hubgeom`, `websterengine/template_test.go` and `loomcli` belong to other packages.
  - No surviving assertion was removed or widened. The only assertion the diff deletes is the `Order()` DeepEqual, whose subject is gone.
- **`skillStartFixtureWith`'s doc comment was rewritten** (`internal/shuttleengine/startup_test.go:1426-1431`). This is prose only. The body is unchanged and is still exercised by its callers at `startup_test.go:1632` and `:1923` and `run_test.go:741`.

**Findings**

None. Every behaviour the diff changes either still has a pinning test (`TestDerive`, the guard test) or lives only in removed test-only code. The diff adds no behaviour, and it loosens no assertion that existed at the review base.

**Tool calls**

None; the inherited context held everything this lens needed.
